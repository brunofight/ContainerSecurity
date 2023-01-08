# 6. Der Baustein Containerisierung

Der 2022 im IT-Grundschutz-Kompendium neu eingeführte Baustein SYS.1.6 - Containerisierung stellt spezifisch für Betreiber Kritischer Infrastrukturen eine Reihe von Anforderungen, die größtenteils mit Beachtung der operativen Sicherheitshinweise der vorhergehenden Kapitel erfüllt werden können. Dabei ist zu beachten, dass bisher kaum auf organisatorische Maßnahmen eingegangen wurde. 

Dieses Kapitel ist als Referenz, ähnlich zu den Umsetzungshinweisen für andere BSI-Bausteine, zu sehen. Dabei wird vor allem bei den technischen Maßnahmen auf die bereits dargebotenen Hilfsmitteln und Prinzipien hingewiesen, sodass bedarfsweise schnell an entsprechender Stelle nachgelesen werden kann.

**Notiz zur weiteren Bearbeitung des Dokuments**: oftmals sind Anforderungen aus mehrerern Teilanforderungen zusammengesetzt (jeder Satz mit SOLLTE oder MUSS). Deswegen sollten die Anforderungen nicht basierend auf dessen Titel sondern dessen Beschreibung in [SYS.1.6](https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Grundschutz/IT-GS-Kompendium_Einzel_PDFs_2022/07_SYS_IT_Systeme/SYS_1_6_Containerisierung_Edition_2022.pdf?__blob=publicationFile&v=3) untersucht werden.


## 6.1 Basisanforderungen

### SYS.1.6.A1 Planung des Container-Einsatzes

### SYS.1.6.A2 Planung der Verwaltung von Containern

### SYS.1.6.A3 Sicherer Einsatz containerisierter IT-Systeme

### SYS.1.6.A4 Planung der Bereitstellung und Verteilung von Images

### SYS.1.6.A5 Separierung der Administrations- und Zugangsnetze bei Containern

### SYS.1.6.A6 Verwendung sicherer Images

Diese Anforderung ist eng gekoppelt mit SYS.1.6.A12 und SYS.1.6.A14 und wird mit den Hinweisen zu diesen zwei Anforderungen erfüllt.

### SYS.1.6.A7 Persistenz von Protokollierungsdaten der Container

### SYS.1.6.A8 Sichere Speicherung von Zugangsdaten bei Containern

## 6.2 Standardanforderungen

### SYS.1.6.A9 Eignung für Container-Betrieb

### SYS.1.6.A10 Richtlinie für Images und Container-Betrieb 

### SYS.1.6.A11 Nur ein Dienst pro Container

Aufgrund des geringen Speicherbedarfs des Container-Images gibt es kaum einen Grund mehr als einen Dienst auf einem Container laufen zu lassen. Gerade deswegen bringen Container einen besonderen Anreiz für die Realisierung von Micro-Service-Architekturen mit sich.
Die Auflockerung dieser Anforderung hätte zur Folge, dass der Sicherheitsgewinn durch seccomp-Profile oder Capability-Beschränkungen abnimmt.

Mittels eines Admission Controllers lässt sich überprüfen, ob ein Container-Image nur einen Dienst startet. Insbesondere bedeutet das auch, dass nur einziger Port geöffnet wird.

### SYS.1.6.A12 Verteilung sicherer Images

Hier empfiehlt sich ein Auftragsprozess zur Beantragung neuer Basis-Images, die für den Betrieb benötigt werden. Somit ist der Prprozess ordentlich dokumentiert und es sind stets nur Basis-Images im Einsatz, die zuvor auch geprüft wurden. Zugleich sollte im Freigabeprozess überprüft werden, ob nicht ein bereits verifiziertes Basis-Image verwendet werden kann. An diesem Prozess sollte das Security Operations Center beteiligt werden.

Sämtliche auf diesem Weg hinzugefügte Basis-Images werden mit einer Notary-Signatur versehen (s. Kapitel 4.1)

### SYS.1.6.A13 Freigabe von Images

### SYS.1.6.A14 Aktualisierung von Images

Container Images werden in der privaten Container Registry auf Schwachstellen gescannt und falls erforderlich aktuallisierte Basis-Images geprüft und heruntergeladen. Unabhängig davon, ob das Basis-Image, eine Dependency oder der Quelltext selbst eine Schwachstelle enthält, muss die selbstenwickelte Container-Anwendung neu gebaut werden und deren Funktionalität erneut getestet werden. Eine vollständige Automatisierung von Updates, wie es beim Patchmanagement externer Anwendungen, Dienste und des Betriebssystems selbst üblich ist, kann für die CI/CD-Pipeline nicht sinnvoll umgesetzt werden. Schließlich sind die Entwickler (und kein externer Hersteller) in der Pflicht zu prüfen, ob mit einem Update der Abhängigkeiten die Schnittstelle des Containers semantisch gleich bleibt.

Schwachstellen und Updates im Basis-Image lassen sich mitunter komplett vermeiden, wenn in einem Multi-Stage-Build lediglich die Binary einer Anwendung im Container vorliegt. Ansonsten können diese zumindest maßgeblich durch die Verwendung minimaler Images (wie bspw. distroles) reduziert werden.

So verlockend es auch scheint in Kubernetes die ``imagePullPolicy`` auf ``Always`` zu setzen oder innerhalb eines Deplyoments (bzw. Pods) auf ein Image mit dem ``:latest``-Tag zu verweisen, ist ein solches Vorgehen nicht empfehlenswert. Dieser Herstellerhinweis ist begründet durch damit einhergehende Erschwerung des Rollback-Mechanismus und Prüfung der aktuell verwendeten Image-Version im Cluster. [K8_IMG]

Für genauere Informationen zu den beschriebenen Konzepten, kann in Kapitel 4 nachgesehe werden.

### SYS.1.6.A15 Limitierung der Ressourcen pro Container

Die Container-Orchestrierung Kubernetes stellt hierfür zwei Konzepte bereit **Resource Quotas** und **Limit Ranges**. Mit **Resource Quotas** lassen sich Obergrenzen für gesamte **namespaces** definieren. Innerhalb eines **namespace** könnte somit ein einzelner Pod die gesamten Ressourcen der zugewiesenen **Resource Quota** an sich reißen. Hier erlauben **Limit Ranges** eine Begrenzung der Ressourcen auf Granularität einzelner Pods. 

Ohne Container-Orchestrierungs-Werkzeug könnten Beschränkungen mit cgroups oder Slice-Files festgelegt werden. Allerdings wäre es fahrlässig in einer kritischen Infrastruktur auf die Automatisierungs- und Protokollierungsmöglichkeiten einer Container-Orchestrierung zu verzichten.

### SYS.1.6.A16 Administrativer Fernzugriff auf Container

Ein administrativer Fernzugriff auf Container darf unter keinen Umständen in einer Produktivumgebung erfolgen. Hierbei würde abermals das Prinzip **Immutable Containers** verletzt werden. Außerdem müssten nur zum Zweck der Administration übliche Linux-Befehle (``sh``, ``ls``, etc.) im Image vorbehalten werden.

In einer Entwicklungsumgebung könnte man zum Debugging einen Fernzugriff erlauben. Wenn damit einhergeht, dass ein anderes Basis-Image, als in der Produktivumgebung verwendet werden muss (bspw. node:18 und distroles/node) kann wiederum keine identische Semantik der Anwendungen garantiert werden, weswegen auch dieser Punkt hinfällig wird.

### SYS.1.6.A17 Ausführung von Containern ohne Privilegien

Per Default wird der *Pod Security Standard* ``Restricted`` in einem Kubernetes Cluster eingesetzt (s. Kapitel 3.1). Ausnahmen 

### SYS.1.6.A18 Accounts der Anwendungsdienste


### SYS.1.6.A19 Einbinden von Datenspeichern in Container


### SYS.1.6.A20 Absicherung von Konfigurationsdaten

## 6.3 Anforderungen bei erhöhtem Schutzbedarf

### SYS.1.6.A21 Erweiterte Sicherheitsrichtlinien

Für Mandatory Access Control stehen AppArmor oder SELinux zur Verfügung (s. Kapitel 3.3). Gemeinsam mit Policy-Definitionen aus SYS.1.6.A17 kann die Verwendung eines Profils erzwungen werden. Der Einsatz eines Security Admission Controllers für feingranularere Policies sollte in Erwägung gezogen werden:

- [KubeWarden](https://docs.kubewarden.io/)
- [Kyverno](https://kyverno.io/policies/pod-security/)
- [OPA Gatekeeper](https://open-policy-agent.github.io/gatekeeper/website/docs/howto/)

### SYS.1.6.A22 Vorsorge für Untersuchungen

### SYS.1.6.A23 Unveränderlichkeit der Container

Diese Anforderung wird mit der Bedingung **Immutable Containers** erfüllt.

### SYS.1.6.A24 Hostbasierte Angriffserkennung

Seccomp und Linux Security Module können in einem Auditiermodus gestartet werden. Dabei übernimmt Seccomp die Aufgabe Kernel-Anfragen (syscalls) und AppArmor/SELinux:

- Netzverbindungen
- erstellte Prozesse
- Dateisystem-Zugriffe

zu protokollieren. Meldungen werden anhand von SIEM-Regeln zu Angriffsmustern korreliert.

Ein vollumfängliches Cluster-Observability-Tool wie [AquaSec Kubernetes Security](https://www.aquasec.com/products/kubernetes-security/) könnte ebenfalls in Betracht gezogen werden.

### SYS.1.6.A25 Hochverfügbarkeit von containerisierten Anwendungen


### SYS.1.6.A26 Weitergehende Isolation und Kapselung von Containern 
