# 3. Containersicherheit zur Laufzeit

In diesem Kapitel werden einige wichtige Mechanismen zur Gewährleistung der Containerisolation und Unveränderlichkeit von Containern zur Laufzeit betrachtet. Letzeres ist auch unter dem Begriff *Drift Prevention* bekannt. Die Analyse erfolgt anhand eines lokalen Minikube-Clusters (s. Kapitel 7.2 zur Referenz).

Seit einigen Jahren ist es möglich mit dem extended Berkeley Packet Filter (eBPF) beliebige Events im Kernel auszuwerten und Funktionalität basierend auf diesen hinzuzufügen. Während es komplex ist eigenhändig eBPF-Programme für den Kernel zu schreiben, gibt es bereits eine Vielzahl von Programmen, welche Detailwissen über den Kernel selbst über eine abstrahierte Schnittstelle verbergen. Das Open-Source-Projekt *Cilium* bietet ein breites Anwendungsspektrum für eBPF (s. Kapitel 3.3).

Es kann dennoch lohnenswert sein einige herkömmliche und etablierte Ansätze zur Berechtigungsrestriktion, wie *seccomp*, *AppArmor* und *SELinux* anzusehen, da mit diesen auch schon eine beachtenswerter Sicherheitsgewinn einhergeht (Kapitel 3.2 und 3.3). 

## 3.1 Grundlegende Konfiguration

Bevor ergänzende Sicherheitsmaßnahmen in Betracht gezogen werden, sollten Container genau so konfiguriert sein, dass sie nur die notwendigen Rechte für ihre Aufgabe besitzen. Dieser Ansatz leitet sich vom *Principle of least privilege* ab. 

Problematisch ist hierbei, dass die Standardwerte von gängigen *Container Runtimes* wie Docker nicht mit diesem Prinzip konform sind. Das fällt insbesondere bei dem *run-as-root default* auf. Ohne genauere Spezifikation eines Nutzers mit der ``--user``-Flag bzw. Angabe eines Nutzers mit ``USER`` im *Dockerfile* läuft der containerisierte Prozess unter dem System-Root-Nutzer. Sofern es einem Angreifer also gelingt aus dem Container auszubrechen (möglicherweise eine Konsequenz weiterer Fehlkonfigurationen in diesem Kapitel) besitzt dieser bereits die höchsten Systemrechte. Dabei ist herauszustreichen, dass Container im Allgemeinen nicht prozessübergreifende Aufgaben auf dem Host wahrnehmen müssen (eine Ausnahme bilden *Sidecar-Container*).

In einem Kubernetes-Cluster empfiehlt es sich Pods und Deployments mit dem ``securityContext``-Feld zu versehen. Für dieses Feld findet man in der API-Referenz zahlreiche untergeordnete Attribute, darunter auch die Spezifikation von *capability*-Restriktionen, eines *seccomp*-Profils, und eines *SELinux*-Kontext (s. 3.2 und 3.3). Im Sinne der laufenden Argumentation ist vor allem auf die Felder:

- ``allowPrivilegeEscalation``
- ``runAsGroup``
- ``runAsNonRoot``
- ``runAsUser``

hinzuweisen. Kubernetes legt somit global für die jeweilige *Container Runtime* fest, unter welchem Nutzer Container des spezifizierten *Pods* gestartet werden. Zwecks Auditierung wäre immer zu hinterfragen, warum die Felder-Wert-Paare ``allowPrivilegeEscalation: true`` oder ``runAsNonRoot: false`` gesetzt sein sollten.
[K8S_SC]

Kubernetes bietet mit *Pod Security Standards* eine leicht zu implementierende globale Policy für diese Werte. Die *Restricted*-Policy verlangt das Setzen der zuvor genannten Attribute im gesamten Cluster (nachträgliche Definition von Ausnahmen möglich).:

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: PodSecurity
  configuration:
    apiVersion: pod-security.admission.config.k8s.io/v1
    kind: PodSecurityConfiguration
    defaults:
      enforce: "baseline"
      enforce-version: "latest"
      audit: "restricted"
      audit-version: "latest"
      warn: "restricted"
      warn-version: "latest"
    exemptions:
      usernames: []
      runtimeClasses: []
      namespaces: [kube-system]
# s. https://kubernetes.io/docs/tutorials/security/cluster-level-pss/
```

In diesem Beispiel müssen mindestens *Baseline*-Standards eingehalten werden. Verstöße gegen *Restricted*-Standards werden protokolliert (``audit``).

[K8S_PSS], [K8S_PSA]

### 3.1.1 Clusterinterne Netzwerke

Wie in Kapitel 2.4 angesprochen sollten für die Erreichbarkeit eines Pods innerhalb eines Clusters, *Services* verwendet werden. Darüber wird sichergestellt, dass die Netzwerkschnittstelle (mit *kube-dns*) konsistent bleibt und somit andere Dienste auf diese zuverlässig zugreifen können. Zudem fungieren *Services* auch als *LoadBalancer* für das zugrundeliegende *ReplicaSet*, welches über den *Service* angesprochen wird (oder genau genommen alle Pods, die mit dem definierten Selektor ``app.kubernetes.io/name`` übereinstimmen).

Zusammen mit dem Kubernetes-*NetworkPolicy* Objekt ist es möglich innerhalb des Clusters eine weitreichende Netzwerk-Segregation umzusetzen. Ein abgewandeltes Beispiel der Kubernetes-Dokumentation:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: myproject
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
  ingress:
    - from:        
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379     
```

zeigt wie Pods mit der Rolle ``db`` im Namespace *myproject* nur eingehenden Verkehr von Pods im gleichen Namespace mit der Rolle ``frontend`` erlauben. Es ist auch möglich mit:

```yaml
ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
```

den Zugriff von cluster-externen IP-Adressen auf bestimmte Pods (und Deployments) einzuschränken. Hier würden ausschließlich IP-Adressen im Class-B-Netz ``172.17.0.0/16`` für *ingress traffic* zugelassen werden. [K8S_NET], [K8S_SVC]

### 3.1.2 Externer Netzwerkzugriff auf ein Cluster

Per Default sind Pods in einem Kubernetes-Cluster nicht extern erreichbar. Hierzu bedarf es einer der folgenden Komponenten:

- **NodePort**: Öffnung eines Ports auf einem Node (per default nur 30000-32767) und Zuordnung dessen zu einem *Service*. Für eine Produktivumgebung ist diese Einstellung nicht sinnvoll, da sämtlicher Traffic zwischen Cluster und externen Hosts dezentralisiert und dementsprechend schwieriger zu kontrollieren wird.
- **LoadBalancer**: Routinglösung bei externen Netzwerkverkehr, welcher nicht über HTTP/HTTPS laufen kann. Sofern das Cluster On Premise betrieben wird ist das Aufsetzen des zugehörigen technischen LoadBalancers mit erheblichen Aufwand verbunden.
- **Ingress mit IngressController**: für HTTP/HTTPS-Kommunikation präferierte Lösung. Da MicroServices letztlich über REST-Schnittstellen realisiert werden, stellt die Protokollbeschränkung keine große Hürde da. *Ingress* bedarf (mindestens) eines *IngressControllers*; eine Liste möglicher Kandidaten hierfür findet man in der [Kubernetes-Dokumentation](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/). Über *Ingress* werden *Services* innerhalb des Clusters zentral über Web-Pfade angesprochen.

![Abbildung: Ingress in Kubernetes](Doc/Images/Ingress.png)

### 3.1.3 Mounts

In dem Wissen, dass Container des gleichen Images in ihrem Verhalten identisch sein sollten  (*immutable Containers*) stellt sich die Frage der Persistenz von container-geschaffenen Daten über die Laufzeit eines Containers hinweg. Bei containerisierten Datenbanksystemen schließen sich Konsistenzanforderungen an. Daraus ergibt sich die Notwendigkeit einheitlicher lokaler oder externer Speichersysteme, die Containern zur Verfügung gestellt werden.

Auch hier gilt das *Principle of least privilege*. Das beginnt bei der genauen Definition von Dateirechten (read, write, execute) und Beschränkung auf die geringstnotwendige Dateimenge. Wie in [book.hacktricks.xyz](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-breakout/docker-breakout-privilege-escalation#privilege-escalation-with-2-shells-and-host-mount) beschrieben können willkürlich bzw. leichtfertig definierte Mounts für lokale *Privilege Escalation* oder Umgehung der Container Isolation ausgenutzt werden.

Für die feingranulare Festlegung von Dateirechten sollte eines der Linux Security Modules (AppArmor oder SELinux - s. 3.3) in Betracht gezogen werden.

## 3.2 Secure Computing Mode (seccomp)

Der Linux-Kernel stellt mit Secure Computing Mode (seccomp) ein Feature zur Beschränkung der von einem Prozess ausführbaren *System Calls* (syscalls) bereit.

*System Calls* sind die Schnittstelle für Prozesse in *User Space*, um auf privilegierte Funktionen in *Kernel Space* zuzugreifen. Die Anzahl der verfügbaren *syscalls* hängt von der Prozessorarchitektur ab. So gibt es Stand November 2022 für ARM-Prozessoren insgesamt 450 verschiedene *syscalls*. [Jusc22], [Rice20]

Daraus ergibt sich die Implikation, dass (containerisierte) Prozesse im Allgemeinen nicht alle *syscalls* für die Erfüllung ihrer vorgesehenen (!) Aufgabe benötigen. Man kann sogar so weit gehen, dass bestimmte Funktionen unter gar keinen Umständen von Containern übernommen werden sollten. So schließt das [Docker Default Seccomp Profil](https://docs.docker.com/engine/security/seccomp/#significant-syscalls-blocked-by-the-default-profile) *syscalls* wie ``create_module``, ``delete_module``, ``mount`` oder ``reboot`` aus. Es ist dabei ebenfalls anzumerken, dass Teilmengen von *syscalls* durch *capabilities* beschränkt werden (eine Aufgabe für Linux Security Modules - s. Kapitel 3.3).

Seccomp ist nicht nur dazu in der Lage Systemaufrufe zu blockieren, sondern kann auch Teilmengen auf Per-syscall-Basis auditieren (``SCMP_ACT_LOG``).

Die Idealvorstellung wäre es für jedes Container Image ein Seccomp-Profil zu erstellen, welches ausschließlich die für diesen Dienst benötigten *syscalls* zulässt. Mithilfe von ``strace`` lassen sich die von einer Anwendung genutzten *syscalls* extrahieren:

```bash
strace -c -f -S name <command line name> 2>&1 1>/dev/null | tail -n +3 | head -n -2 | awk '{print $(NF)}'
```

Dieses Vorgehen kann für vergleichsweise einfache Applikationen nützlich sein. Allerdings solle man beachten, dass ähnlich zu dem Wunsch einer vollständigen Testabdeckung, komplexe Anwendungen nicht mit Gewissheit auf eine konkrete Menge von *syscalls* festlegbar sind. Ein zu restriktives Seccomp-Profil könnte folglich negative Auswirkungen auf die Funktionalität der (containerisierten) Applikation haben. [Rice20]

Insgesamt ist das jedoch keine Argumentation gegen *seccomp*. Ein gut durchdachtes Standard-Seccomp-Profil reduziert die Möglichkeiten eines Angreifers aus einem kompromitierten Container heraus zu agieren, maßgeblich. Des Weiteren können verdächtige *syscalls* weiterhin geloggt und an ein Sicherheitsmonitoring-System weitergegeben werden.

In Kubernetes forciert der *Pod Security Standard* - *restricted* die Verwendung von wenigstens dem ``RuntimeDefault``-Profil (also dem Profil, welches von der gekapselten Container Runtime bereitgestellt wird) oder einem auf dem Host verfügbaren Profil (d.h. ``Unconfined`` ist nicht erlaubt). [K8S_PSS]

## 3.3 Linux Security Modules (LSM)

Sowohl AppArmor als auch SELinux sind Kernel-Module, die parallel zu dem per Default vorhandenen *Discretionary Access Control* (DAC), den Zugriff von Prozessen (und somit auch Containern) auf Systemressourcen (Capabilities, Dateizugriff,...) global beschränken. Dieses Konzept ist unter dem Namen *Mandatory Access Control* (MAC) bekannt. [Rice20]

In Kapitel 3.1 wurde darauf hingewiesen, dass es innerhalb des Kubernetes ``securityContext`` auch möglich wäre *Capability*-Einschränkungen zu treffen. Wenn man sich dafür entscheidet ein LSM zur Härtung von Containern zu verwenden, dann sollte man auch die Capability-Beschränkung von diesem übernehmen lassen, sodass es zu keiner redundanten Konfiguration und unerwünschten Konflikten kommt.

### 3.3.1 AppArmor

Die Funktionsweise von AppArmor basiert auf Konfigurations-Profilen, die Prozessen zugeordnet werden können. AppArmor-Profile haben eine etwas komplizierte Syntax. Hierbei kann das Tool ``bane`` helfen. [Rice20]

Um beispielsweise ein AppArmor-Profil zu erstellen, welches lesenden/schreibenden Zugriff auf sämtliche Dateien unterbindet und nur die capabilities ``chown``, ``dac_override``, ``setuid``, ``setgid`` und ``net_bind_service`` erlaubt, erstellt man eine ``.toml``-Datei:

```toml
Name = "myapp"

[FileSystem]
AllowExec = [
	"<nodejs install dir>"
]

[Capabilities]
Allow = [
	"chown",
	"dac_override",
	"setuid",
	"setgid",
	"net_bind_service"
]
```
  
,aus welcher ``bane`` wiederum ein AppArmor-Profil konstruiert, dieses automatisch unter ``/etc/apparmor.d/containers/`` ableget und ``apparmor_parser`` ausführt.

```
sudo bane myapp.toml
```

[Bane]

Das somit installierte AppArmor-Profil, kann Docker-Containern zugewiesen werden, indem es mit ``--security-opt`` angegeben wird. 

```bash
docker run --security-opt apparmor=<profile-name> <image>
```


In Kubernetes verwendet man die Annotation ``container.apparmor.security.beta.kubernetes.io/<container-name>``. Beispielsweise könnte ein AppArmor-gehärteter Pod wie folgt erstellt werden:
  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-apparmor
  annotations:
    container.apparmor.security.beta.kubernetes.io/hello: localhost/myapp.toml
spec:
  containers:
  - name: hello
    image: busybox:1.28
    command: [ "sh", "-c", "echo 'Hello AppArmor!' && sleep 1h" ]
```

[K8s_AA]
  
Zuvor muss allerdings sichergestellt werden, dass dieses AppArmor-Profil auch auf den Nodes verfügbar ist. Hierfür gibt es einige Ansätze. Am sinnvollsten scheint es ein *DaemonSet* zu erstellen. *DaemonSets* garantieren, dass darin definierte Pods auf allen Nodes laufen. Somit können Pods innerhalb eines *DaemonSets* die Aufgabe übernehmen periodisch aus einer ConfigMap neue Profile zu beziehen.
  
[K8s_AA]

### 3.3.2 SELinux

*Security Enhanced Linux* (SELinux) gewährt basierend auf zusätzlichen Datei-Labels Zugriff...
  

## 3.4 Extended Berkeley Packet Filter mit Cilium

- Konfiguration
- Netzwerk
- Side-Car Container

- im Kontext eines Kubernetes-Clusters


