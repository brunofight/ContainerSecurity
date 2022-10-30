# Container Security

Arbeitsbereich für Masterprojekt Container Security.

## Motivation

Betrachtung von Angriffsvektoren und möglichen Mitigationsstrategien von Containern und Container-Orchestrierung im **technischen** und **prozessualen Kontext**.

### Technischer Kontext

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
      
### Prozessualer Kontext

Container als Bestandteil der Software Supply Chain

- Container Registry
- Admission Control


  
  
