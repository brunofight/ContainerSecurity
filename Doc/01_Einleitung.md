# 1. Einleitung

Containerisierte Anwendungen haben mit der Einführung von Docker (2013) einen großen Aufschwung erlebt. Schnelle Deployment-Zeiten, hohe Portabilität nach dem Prinzip "Build once, run anywhere" und Ressourceneffizienz machen die neue Technologie attraktiver als herkömmliche Virtualisierung. Insbesondere der geringe Speicherbedarf eines einzelnen Containers ermöglicht erst praktikable MicroService-Architekturen.

Während ein Großteil der Privatwirtschaft bereits seit einigen Jahren von den Vorzügen der Containerisierung profitiert, haben Betreiber Kritischer Infrastrukturen sich, mangels Sicherheitsvorgaben des BSI, von der Thematik fern gehalten. Wie sich in der Ausarbeitung herausstellen wird sind Container nur oberflächlich mit Virtuellen Maschinen vergleichbar.

Erst in der 2022 Version des IT-Grundschutzkompendiums wurden Sicherheitsvorgaben für Containerisierung in einem Baustein (SYS.1.6) erfasst. Umsetzungshinweise gibt es bisher nicht. 



- primäres Anliegen Grundlage für operative IT-Sicherheit von Container-Infrastruktur
- Containerisierung kann ein Sicherheitsgewinn gewinn sein in Bezug auf MicroServices und Observability

## Technischer Kontext

(migriert zu 2. ) 

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


