# 2. Grundlegende Konzepte

Dieses Kapitel gibt einen kurzen Überblick zu den fundamentalen Prinzipien und der technischen Implementierung, die mit der Containerisierung von Anwendungen einhergeht. Die Wirkungsweise der linux-nativen *Capabilities*, *cgroups* und *namespaces* besitzt einen hohen Stellenwert für die Containersicherheit zur Laufzeit (s. Kapitel 3).

In seinem Kern ist ein Container ein Prozess, der auf einem Linux Kernel läuft. Gerade deshalb sind Container so effizient mit Deployment-Zeiten im Millisekundenbereich und in ihrer Ressourcennutzung. Das steht im harten Kontrast zu Virtuellen Maschinen, die jeweils mit einem eigenen Betriebssystem gestartet werden.

Aus Sicht der IT-Sicherheit ergibt sich mit Containern wiederum eine wesentlich höhere Komplexität zur Erreichung der vermutlich wichtigsten Grundbedingung für den Einsatz containerisierter Anwendungen - die Isolation. Ein Hypervisor kommt mit gerade einmal 50.000 Zeilen Quelltext aus und hat eine wesentlich einfachere Aufgabe bzw. ist es nicht einmal vorgesehen, dass VMs in irgendeiner Weise direkt miteinander interagieren (Netzwerkverbindung ausgenommen). [Rice20], [Xen19]

Je nach Version des verwendeten Linux Kernels besteht dieser aus 20 - 35 Millionen Zeilen Quelltext. Es ist durchaus möglich und unter Umständen auch gewollt Containern geteilte Ressourcen zur Verfügung zu stellen. Prozesse können i.A. auch andere Prozesse sehen. Zusammengefasst bedeutet das, dass mit Zunahme der Konfigurationsmöglichkeiten das Potenzial für eine Schwachstelle im Kernel Code oder eine Fehlkonfiguration steigt. [Rice20], [WiLK]

Aus diesem Grund werden zunächst die Mechanismen zur Erfüllung der Container Isolation in Kapitel 2.1 vorgestellt (weitergehende Härtungsmaßnahmen s. 3.). Kapitel 2.2 befasst sich mit der Unveränderlichkeit von Containern, einer weiteren wünschenswerten Eigenschaft von Containern und in 2.3 wird ein grober Einblick in die Terminologie von Kubernetes, der marktführenden Container-Orchestrierungslösung gegeben.

## 2.1 Container Isolation

Eine funktionierende Container Isolation setzt voraus, dass ein Container keine anderen auf dem gleichen Kernel laufenden Container oder sonstige Host-Prozesse negativ beeinflussen kann. Zusammengefasst lässt sich diese erreichen durch

| Linux (Befehl/Konzept) | Zweck | Beschreibung |
|-----|-----|-----|
| cgroups | Ressourcenbeschränkung | Limitierung des Speicher-, Netzwerk-, CPU-Verbrauchs oder auch Beschränkung der maximalen Anzahl an Kindprozessen. <br> *Cgroups* werden in ``/sys/fs/cgroup`` erstellt. Das Anlegen eines neuen Ordners in bspw. ``/sys/fs/cgroup/memory`` und Schreiben der PID im darin befindlichen ``cgroup.procs`` bindet einen Prozess an diese *cgroup*. |
| namespaces | Sichtbarkeitsbeschränkung | Mit dem Befehl ``unshare`` lassen sich Kindprozesse erstellen, die nicht den *namespace* des Elternprozesses übernehmen. Hierüber erhält ein Prozess (Container) u.a. ein vom Host unabhängiges Netzwerk-Interface, Prozess-Nummerierung und Mount-Points|
| chroot | Sichtbarkeitsbeschränkung | *Namespaces* alleine reichen nicht aus, um eine vollständige Sichtbarkeitsbeschränkung zu erreichen. Das liegt daran, dass Prozesse nach wie vor aus den Verzeichnissen ``/proc`` und ``/mnt`` lesen. Mit ``chroot`` wird das Wurzelverzeichnis eines Prozesses verlegt, sodass dieser nicht mehr auf die Verzeichnisse des Hosts zugreifen kann. In diesem Schritt ist jedoch zugleich der ``/bin``-Ordner unsichtbar geworden, wodurch innerhalb des Prozesses keine weiteren Befehle mehr ausgeführt werden können. Genau hierfür verwendet man *Container Images*, eine rudimentäre Verzeichnisstruktur, welche im Idealfall nur die notwendigen Befehle zur Ausführung der darin befindlichen Anwendung enthält. |
| capabilities | Fähigkeitsbeschränkung | *Capabilities* limitieren die *Syscalls*, die ein Prozess ausführen darf. Eine Liste gefährlicher Capabilities kann [HTCap] entnommen werden. Darunter bspw. ``CAP_SYS_ADMIN`` mit zahlreichen administrativen Berechtigungen, die trivial für Privilege Escalation genutzt werden können. |


## 2.2 Immutable Containers

Bei einer Gruppe von Containern handelt es sich genau dann um *Immutable Container*, wenn diese vom gleichen Image stammen und zur Laufzeit identisches Verhalten aufweisen. Somit sollten Container unter keinen Umständen:

- neue Code-Versionen oder Abhängigkeiten zur Laufzeit herunterladen
    - das ist sowohl aus Sicherheitsgründen, als auch aus Gründen der Wartbarkeit und Fehlerreproduzierbarkeit untragbar.
- Prozesse starten, die nicht für die Ausführung der Anwendung benötigt werden
    - das schließt insbesondere die schlechte Praxis zu Wartungszwecken eine Shell auf einem Container zu starten mit ein.

Unter der Voraussetzung eines *Immutable Container* genügt es, Schwachstellenscans (bzw. Image Scans) in der Container Registry auszuführen.

Viele Container-Infrastrukturen nehmen die Unveränderlichkeit von Containern als Prämisse an, ohne diese technisch garantieren zu können. Zu diesem Zweck sollten *Container Image Profiles* eingesetzt werden (s. Kapitel 3). Hierdurch offenbart sich ein maßgeblicher Sicherheitsgewinn bei der Entwicklung von Microservice-Architekturen gegenüber Monolithen. Es ist viel einfacher möglich festzustellen, was ein Microservice (Container) machen soll und darf und dementsprechend genau diese Aktivitäten in einer Whitelist zu erfassen. [Rice20]

## 2.3 Container Runtime 

Der Begriff *Container Runtime* ist überladen und wird oftmals synonym für high-level *Container Runtimes* (wie containerd, CRI-O) verwendet, welche wiederum eine low-level *Container Runtime* (in der Regel runC) einbeziehen. 

Die high-level *Container Runtime* ist für das Herunterladen von Container Images aus einer Registry, die Verwaltung von Mounts und Speichern, sowie das Ausführen von Containern über eine OCI-konforme *low-level Container Runtime* zuständig. Anschließend erstellt und führt die low-level *Container Runtime* die containerisierten Prozesse aus. [Dono21]

Für diesen Zweck muss eine *Container Runtime* zwingend mit Root-Rechten laufen. 

Hinzu kommt eine Schnittstelle zur Interaktion mit der *Container Runtime*, also das *Container Runtime Interface* (CRI). Das CRI kann in Form einer Kommandozeile (``docker``) oder über eine API gegeben sein. Es ergibt sich somit die Implikation, dass unprivilegierte Nutzer mit Zugriff auf das CRI faktisch privilegierte Nutzer sind (s. Kapitel 3). [Rice20]

## 2.4 Container-Orchestrierung

Sobald etwas komplexere Microservice-Architekturen oder einfacher ausgedrückt der Bedarf an containerisierten Anwendungen im Unternehmen zunimmt, gelangt man aus administrativer Sicht schnell an Grenzen. Das wird unter anderem deutlich bei der Aktualisierung einer laufenden Container-Umgebung, wo das Vorgehen schematisch wie folgt abläuft:

```bash
docker ps 
docker stop <containerName>
docker rm <containerName>
docker run <neuesImage> --name <containerName>
# Vorgehen für jeden Container im Deployment wiederholen
```

Mit Kubernetes ist lediglich eine Anpassung in der zugehörigen ``deployment.yaml`` vorzunehmen und diese anschließend mit ``kubectl apply -f deployment.yaml`` auszurollen. Die Orchestrierung garantiert, dass das zugrundeliegende redundant aufgesetzte Deployment während des Rollouts stets laufende Container enthält. Des Weiteren erkennt und behandelt Kubernetes automatisch abgestürzte Container und startet diese wieder.

Zur Realisierung solcher Aufgaben basiert Kubernetes auf einer komplexen Architektur aus **Nodes** und darauf laufenden Diensten (s. Abbildung). Auf den **Nodes** laufen letztendlich die **Pods**, eine Abstraktionsschicht für mehrere Container im gleichen *namespace*.

![Abbildung: Kubernetes Node-Architektur [K8S_Arc]](Doc/Images/components-of-kubernetes.png)

In der Control Plane bzw. auf dem sogenannten Master-Node laufen folgende Dienste:

- **API-Server**: Schnittstelle zur Interaktion mit dem Cluster
- **Controller Manager**: Monitoring auf Abweichungen vom Soll-Zustand des Clusters und Propagierung von Maßnahmen zur Wiederherstellung an die Worker-Nodes
- **Scheduler**: Verteilung neuer Pods auf Worker-Nodes basierend auf deren Auslastung
- **etcd**: Key-Value-Store, welcher den Zustand des Clusters abspeichert, sodass der Controller Manager Änderungen erkennen kann

Controller Manager und Scheduler interagieren mit dem **kubelet**-Dienst auf den Worker-Nodes. Dieser setzt die angefragten Änderungen in der vorliegenden **Container Runtime** (bspw. containerd oder CRI-O) um. Abschließend läuft auf den Worker-Nodes noch der **kube-proxy**, welcher die Netzwerkregeln zur Kommunikation der Pods untereinander (mittels *services*) oder mit der Außenwelt (mittels *ingress*) geltend macht. Dabei greift der kube-proxy auf den Paketfilter des Betriebssystems zurück, also bei Unix-Derivaten *iptables*. [K8S_Arc]

Auf Grundlage dieser Architektur können Deployments basierend auf einer Vielzahl von Konzepten wie *ReplicaSets*, *Services*, *Ingress*, *Volumes*, *PersistentVolumeClaims*, *Secrets*, *ConfigMaps*, *LimitRanges*, *ResourceQuotas*, *Namespaces* und *Policies* konfiguriert werden. *Namespaces* im Kontext von Kubernetes spielen eine ähnliche Rolle wie die in Kapitel 2.1 eingeführten *Linux Namespaces*. Während *Linux Namespaces* die Sichtbarkeit von Betriebssystemressourcen für einzelne Prozesse beschränken, isolieren *Kubernetes Namespaces* Ressourcen und Nutzerrechte im gesamten Cluster.

Von besonderer Relevanz für die Sicherheit des Clusters sind dabei *LimitRanges* und *ResourceQuotas*, welche die Ressourcenbeschränkungs-Konzept von *cgroups* auf Pod- bzw. Namespace-Ebene durchsetzen.

In der offiziellen Kubernetes-Dokumentation werden die einzelnen Komponenten wesentlich detaillierter charakterisiert. Die folgenden Kapitel beschreiben ausschließlich Möglichkeiten innerhalb des Clusters Sicherheitsmaßnahmen zur Gewährleistung der Container-Isolation und von *Immutable Containers* einzubringen.
