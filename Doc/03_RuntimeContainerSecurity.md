# 3. Containersicherheit zur Laufzeit

In diesem Kapitel werden einige Mechanismen zur Gewährleistung der Containerisolation und Unveränderlichkeit von Containern zur Laufzeit betrachtet. Letzeres ist auch unter dem Begriff *Drift Prevention* bekannt. Die Analyse erfolgt anhand eines lokalen Minikube-Clusters (s. Kapitel 7.2 zur Referenz).

Seit einigen Jahren ist es möglich mit dem extended Berkeley Packet Filter (eBPF) beliebige Events im Kernel auszuwerten und Funktionalität basierend auf diesen hinzuzufügen. Während es komplex ist eigenhändig eBPF-Programme für den Kernel zu schreiben, gibt es bereits eine Vielzahl von Programmen, welche Detailwissen über den Kernel selbst über eine abstrahierte Schnittstelle verbergen. Das Open-Source-Projekt *Cilium* bietet ein breites Anwendungsspektrum für eBPF (s. Kapitel 3.3).

Dennoch lohnt es sich einige herkömmliche Ansätze zur Berechtigungsrestriktion, wie *seccomp*, *AppArmor* und *SELinux* anzusehen (Kapitel 3.2 und 3.3). 

## 3.1 Grundlegende Konfiguration


## 3.2 Secure Computing Mode (seccomp)

Der Linux-Kernel stellt mit Secure Computing Mode (seccomp) ein Feature zur Beschränkung der von einem Prozess ausführbaren syscalls bereit. 

- default docker seccomp Profile --> 44 syscalls blockiert
- 2022 ca. 400 syscalls
- laut aquasec benötigt Container zw. 40 und 70 syscalls --> default Profile unzureichend
- docker seccomp json dokument
  - SCMP_ACT_KILL, SCMP_ACT_TRAP, SCMP_ACT_ERRNO, trace, allow, log
- strace verwenden um syscalls eines containers zu profilen ``strace -qc time``, ``strace -c -f -S name time 2>1&1 1>/dev/null | tail -n +3 | head -n -2 | awk '{print $(NF)}'``


```bash
# cyberbit training
sudo docker run --security-opt seccomp=/home/cyberuser/profiles/violation.json --name cyberbit -dit busybox:latest

strace -c -f -S name <command line name> 2>&1 1>/dev/null | tail -n +3 | head -n -2 | awk '{print $(NF)}'
```

## 3.3 Linux Security Modules

Sowohl AppArmor als auch SELinux sind Kernel-Module, die parallel zu dem per Default vorhandenen *Discretionary Access Control* (DAC), den Zugriff von Prozessen (und somit auch Containern) auf Systemressourcen (Capabilities, Dateizugriff,...) global beschränken. Dieses Konzept ist unter dem Namen *Mandatory Access Control* (MAC) bekannt. [Rice20]

### 3.3.1 AppArmor

Die Funktionsweise von AppArmor basiert auf Konfigurations-Profilen, die Prozessen zugeordnet werden können. AppArmor-Profile haben eine etwas komplizierte Syntax. Hierbei kann das Tool ``bane`` helfen. [Rice20]

Um ein AppArmor-Profil zu erstellen, welches lesenden/schreibenden Zugriff auf sämtliche Dateien unterbindet und nur die capabilities ``chown``, ``dac_override``, ``setuid``, ``setgid`` und ``net_bind_service`` erlaubt, erstellt man eine ``.toml``-Datei:

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


