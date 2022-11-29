# 6. Der Baustein Containerisierung

- spezifisch für Betreiber Kritischer Infrastrukturen
- was gilt es noch zur Erfüllung zu beachten?
  - was noch nicht in den vorherigen Kapiteln behandelt wurde
  - organisatorische Maßnahmen

## 6.1 Anforderungen

### SYS.1.6.A1 Planung des Container-Einsatzes

### SYS.1.6.A2 Planung der Verwaltung von Containern

### SYS.1.6.A3 Sicherer Einsatz containerisierter IT-Systeme

### SYS.1.6.A4 Planung der Bereitstellung und Verteilung von Images

### SYS.1.6.A5 Separierung der Administrations- und Zugangsnetze bei Containern

### SYS.1.6.A6 Verwendung sicherer Images

### SYS.1.6.A7 Persistenz von Protokollierungsdaten der Container

### SYS.1.6.A8 Sichere Speicherung von Zugangsdaten bei Containern


### SYS.1.6.A9 Eignung für Container-Betrieb

### SYS.1.6.A10 Richtlinie für Images und Container-Betrieb 

### SYS.1.6.A11 Nur ein Dienst pro Container

Aufgrund des geringen Speicherbedarfs des Container-Images gibt es kaum einen Grund mehr als einen Dienst auf einem Container laufen zu lassen. Gerade deswegen bringen Container einen besonderen Anreiz für die Realisierung von Micro-Service-Architekturen mit sich.
Die Auflockerung dieser Anforderung hätte zur Folge, dass der Sicherheitsgewinn durch seccomp-Profile oder Capability-Beschränkungen abnimmt.

Mittels eines Admission Controllers lässt sich überprüfen, ob ein Container-Image nur einen Dienst startet. Insbesondere bedeutet das auch, dass nur einziger Port geöffnet wird.

### SYS.1.6.A12 Verteilung sicherer Images


### SYS.1.6.A13 Freigabe von Images

### SYS.1.6.A14 Aktualisierung von Images

### SYS.1.6.A15 Limitierung der Ressourcen pro Container

Die Container-Orchestrierung Kubernetes stellt hierfür zwei Konzepte bereit **Resource Quotas** und **Limit Ranges**. Mit **Resource Quotas** lassen sich Obergrenzen für gesamte **namespaces** definieren. Innerhalb eines **namespace** könnte somit ein einzelner Pod die gesamten Ressourcen der zugewiesenen **Resource Quota** an sich reißen. Hier erlauben **Limit Ranges** eine Begrenzung der Ressourcen auf Granularität einzelner Pods. 

Ohne Container-Orchestrierungs-Werkzeug könnten Beschränkungen mit cgroups oder Slice-Files festgelegt werden. Allerdings wäre es fahrlässig in einer kritischen Infrastruktur auf die Automatisierungs- und Protokollierungsmöglichkeiten einer Container-Orchestrierung zu verzichten.

### SYS.1.6.A16 Administrativer Fernzugriff auf Container

Ein administrativer Fernzugriff auf Container darf unter keinen Umständen in einer Produktivumgebung erfolgen. Hierbei würde abermals das Prinzip **Immutable Containers** verletzt werden. Außerdem müssten nur zum Zweck der Administration übliche Linux-Befehle (``sh``, ``ls``, etc.) im Image vorbehalten werden.

In einer Entwicklungsumgebung könnte man zum Debugging einen Fernzugriff erlauben. Wenn damit einhergeht, dass ein anderes Basis-Image, als in der Produktivumgebung verwendet werden muss (bspw. node:18 und distroles/node) kann wiederum keine identische Semantik der Anwendungen garantiert werden, weswegen auch dieser Punkt hinfällig wird.

### SYS.1.6.A17 Ausführung von Containern ohne Privilegien


### SYS.1.6.A18 Accounts der Anwendungsdienste


### SYS.1.6.A19 Einbinden von Datenspeichern in Container


### SYS.1.6.A20 Absicherung von Konfigurationsdaten


### SYS.1.6.A21 Erweiterte Sicherheitsrichtlinien


### SYS.1.6.A22 Vorsorge für Untersuchungen

### SYS.1.6.A23 Unveränderlichkeit der Container

Diese Anforderung wird mit der Bedingung **Immutable Containers** erfüllt.

### SYS.1.6.A24 Hostbasierte Angriffserkennung


### SYS.1.6.A25 Hochverfügbarkeit von containerisierten Anwendungen


### SYS.1.6.A26 Weitergehende Isolation und Kapselung von Containern 
