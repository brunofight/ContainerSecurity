# 1. Einleitung

- primäres Anliegen Grundlage für operative IT-Sicherheit von Container-Infrastruktur
- sekundär Erfüllung BSI SYS.1.6 für Betreiber von kritischen Infrastrukturen
  - BSI hat Stand 2022 noch keine Umsetzungshinweise für den Baustein SYS.1.6 Containerisierung [BSI19]
- Containerisierung kann ein Sicherheitsgewinn gewinn sein in Bezug auf MicroServices und Observability

## Technischer Kontext

- Container sind Prozesse mit eigener **cgroup** und eigenen **namespaces** auf einer Linux-Distribution
  - **cgroups** limitieren die Systemressourcen für eine Gruppe von Prozessen (Speicher, erlaubte Anzahl von Prozessen, Netzwerkbandbreite, ...)
  - **namespaces** beschränken die Sichtbarkeit von "file-like"-Ressourcen (``unshare``)
     - Nutzer und Gruppen
     - UTS (Unix Timeharing System)
     - Prozess IDs
     - cgroups
     - mounts
     - Interprozess-Kommunikation
     - Netzwerk (wird im Nachhinein mit einem virtuellen Ethernet-Paar an Host angebunden)
     
  - mit ``chroot`` wird das /-Verzeichnis auf einen anderen Ordner verlegt (mit eigenem /proc, /bin, usw.); dieses Verzeichnis mit rudimentärer Basisausstattung an Applikationen ist im Endeffekt ein **Container Image**
  
- des weiteren sind Linux Capabilities auf ein notwendiges Minimum zu beschränken
  
Diese Maßnahmen erreichen den gewünschten Effekt der **Container-Isolation**. Das heißt ein (kompromittierter) Container soll nicht in der Lage sein den Host oder andere Container zu beeinflussen. Offensichtlich bedeutet das nicht, dass ein kompromittierter Host die darauf befindlichen Container nicht beeinflussen kann, denn diese sind lediglich Prozesse aus seiner Sicht.

Somit sind traditionelle Angriffsvektoren auf den Host gleichzeitig Angriffsvektoren auf die darauf laufenden Container:

- bekannte Schwachstellen mit Exploits (für Host-Betriebssystem und Host-Applikationen)
- unsichere Host-Konfiguration (sehr breit gefasst, sowohl Betriebssystem als auch Applikationen)
- Credential Leaks, schwache Passwörter

Die gleichen Angriffsvektoren bestehen für alle auf dem Host laufenden Container und darin isolierten Applikationen. Mit Bezug auf die Erstellung und Verwaltung von Containern entstehen neue Einfallstore:

- die Container-Runtime (docker, containerd, runc, ...) muss unter dem Root-User laufen, damit die o.g. Befehle ausgeführt werden können, um neue Container zu starten.
   - dazu kommt das **CRI** (Container runtime Interface), womit Nutzer mit der Container-Runtime interagieren können
- kompromittiere Container-Images kompromittieren mindestens die darauf laufende Applikation
- unsichere Container-Konfiguration und Schwachstellen hebeln die Container-Isolation aus
   - im Deployment host-übergreifend (mehrere Nodes in Kubernetes) spielt insbesondere die Netzwerkkonfiguration eine Rolle
      - Network Policy (basiert auf netfilter)
      - Service Meshes in Sidecar-Containern (um bspw. TLS-verschlüsselte Kommunikation zu erzwingen)      
      
## Prozessualer Kontext

Container als Bestandteil der Software Supply Chain

- Entwicklung Quelltext für Anwendung in Container
- Dockerfiles 
- CI/CD
- Container Registry
- Admission Controller
- Container Orchestrierung

und damit verbunden sämtliche Punkte für einen Supply Chain Angriff:

- Schwachstellen im Quelltext oder Dependencies
- manipulierte Dockerfiles, CI/CD Anweisungen
- manipulierte Images in Container Registry
- Deployment manipulierter Images (ohne Admission Controller)


## Techniken zur Mitigation und Erkennung von Angriffen

Welche zusätzlichen Sicherheitsmaßnahmen werden betrachtet?

- Härtung der Container-Isolation (Container Runtime Security)
   - rootless Container
   - Einschränkung der Capabilities (AppArmor, seccomp, SELinux)
   - Drift Prevention (Whitelisting Ansatz für Prozesse und Dateien in einem Container und somit ein Sicherheitspluspunkt für Containern)
      - s. auch immutable Containers
   
- Schwachstellen
   - Container Image Scanning
   - Schwachstellenscans auf Nodes eines Clusters und Build-Server
   
- Erkennung
   - eBPF (extended Berkeley Packt Filter) Integration in ein SIEM
   
- sichere Bereitstellung von Secrets
   - temporäres Dateisystem
   - native Unterstützung in Kubernetes
   - secret storage, secret rotation (z.B. mit Hashicorp)
   
- Image signing, Admission Controller
   
Betrachtung typischer Konfigurationsfehler, z.B.:

- ``--privileged``-Container
- mounts, Container Firewall (= Network Policy in Kubernetes), service meshes
- schwache Zugriffsbeschränkung auf Nodes im Cluster (Node-Root kann auch ohne weiteres Secrets in den Containern auslesen)
- sensitive Informationen in Dockerfiles, jeder Entwickler darf Dockerfiles editieren (s. Docker RUN...)


