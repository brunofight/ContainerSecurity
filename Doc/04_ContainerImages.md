# 4. Container Images

Container Images bilden die Verzeichnisstruktur ab, auf die eine containerisierte Anwendung zur Gewährleistung von dessen Funktionalität zurückgreift. Üblicherweise werden die Abhängigkeiten der Anwendungen (bspw. Laufzeitumgebung) aus einem Basis Image bezogen welches gemeinsam mit der Anwendung in ein neues Container Image gebündelt (**build**) wird. 

Das fertige Container Image wird anschließend entweder direkt ausgerollt oder zunächst in einer Container Registry abgelegt (s. Kapitel 4.2). Container Images sind somit das zentrale Artefakt in der CI/CD-Pipeline und in jedem Schritt bis hin zum Deployment in ein Cluster Bedrohungen ausgesetzt (s. Abbildung). Die Kürzel CICD-SEC-x nehmen Bezug auf die **Top 10 OWASP CI/CD Security Risks**. Von besonderer Bedeutung ist das Risiko CICD-SEC-1, welches bei Nichtbeachtung Angreifern ermöglicht, von einem beliebigen System im Build-Prozess aus, schadhaften Quelltext unkontrolliert in die Produktion auszurollen. [OWASPCD], [Rice20]

Dementsprechend sind den Verifikationsmaßnahmen von Container-Images ein hohes Gewicht beizumessen. In den folgenden Unterkapiteln werden 4 Maßnahmen beschrieben, um sowohl das Deployment schadhafter Container zu verhindern als auch Schwachstellen und Fehlkonfigurationen zu vermeiden.

![Abbildung: Container Images in CI/CD](Doc/Images/Container_CICD.png)


## 4.1 Image Signatur und Verifikation

Mit einer Image Signatur kann gewährleistet werden, dass ein aus einer Container Registry bezogenes Image auch dasselbe ist, welches zuvor im CI/CD-Prozess erzeugt wurde. Ohne Signatur könnte ein Angreifer mit Zugriff auf die Registry schadhafte Image-Imitate bereitstellen.

An dieser Stelle ist von der Image Digest abzugrenzen. Bei diesem handelt es sich um den Hash eines Image Manifests. Darin sind wiederum alle Layer eines Container Images mit ihrem jeweiligen Hash (*digest*) enthalten. Zusammengefasst ist es somit möglich ein spezifisches Container Image über dessen *Digest* aus einer Registry zu pullen und folglich auch dessen Integrität darüber sicherzustellen. Allerdings wäre in der Praxis ein solches Vorgehen wohl kaum anzutreffen. Ähnlich wie in der Beziehung von IP-Adresse zu FQDN ist es handlicher mit einem menschenlesbaren *Tag* zu arbeiten. Ein *Tag* referenziert einen spezifischen Image Digest; dieser Verweis ist variabel (insbesondere bei dem ``:latest`` Tag). Demzufolge kann über einen manipulierten *Tag*-Verweis unbemerkt ein schadhaftes Image gezogen werden. [Rice20], [Docker]

Das generelle Vorgehen zur Signatur von Container Images wird von *The Update Framework* (TUF) abgeleitet. Zumeist kommt *Notary* als Implementierung dieser Spezifiaktion zum Einsatz. Das Framework basiert im Kern auf einer rollenbasierten Hierarchie von asymmetrischen Schlüsselpaaren (wie in einer PKI), womit sich Metadaten signieren lassen. [TUF], [Notary]

Der *Notary*-Dienst besteht aus dem *Notary Server*, welcher signierte Metadaten in einer Datenbank abspeichert und Signatur-Requests an den *Notary Signer* delegiert (s. Abbildung). [NotArc]

![Abbildung: Notary Architektur](Doc/Images/Notary.png)

Während Docker mit den Parametern ``DOCKER_CONTENT_TRUST`` und ``DOCKER_CONTENT_TRUST_SERVER`` sich für die Einbindung eines Notary-Dienstes konfigurieren lässt, bietet Kubernetes selbst aktuell keine native Unterstützung hierfür. Stattdessen muss hier die Aufgabe der Signaturverifikation an einen **Admission Controller** (s. Kapitel 4.4) ausgelagert werden. Dabei ist es sinnvoll den *Notary*-Service (bestehend aus Notary-Signer und Notary-Server) selbst innerhalb des Kubernetes-Clusters bereitzustellen. [Siegert]

## 4.2 Container Registry

Entwickler containerisierter Anwendungen sind vermutlich mit der öffentlich einsehbaren Container Registry Dockerhub vertraut. Von dieser werden die benötigten Basis-Images bezogen. Dieser Ansatz ähnelt der ungeprüften Verwendung von Quelltextausschnitten, Bibliotheken oder sonstigen Abhängigkeiten aus einem offen verfügbaren Code-Repository. Ohne die Verwendung einer eigenen privaten Container Registry ist es folglich schwierig die Kontrolle darüber zu behalten welche (Basis-)Images im eigenen Cluster verwendet werden. Ferner reduziert eine private Container Registry Angriffsvektoren wie Typosquatting und DNS-Spoofing. [Rice20]

Zugleich ist die eigene Container Registry eng mit den Baustein-Anforderungen SYS.1.6.A6 Verwendung sicherer Images, SYS.1.6.A12 Verteilung sicherer Images und SYS.1.6.A14 Aktualisierung von Images verbunden. [BSI22]

Es gibt eine Vielzahl von Lösungen zur Realisierung einer privaten Container Registry. Aus diesem Grund sollten zunächst einige Qualitätsmerkmale für die Auswahl betrachtet werden:

1. Kompatibilität zu anderen Komponenten in der eingesetzten CI/CD-Pipeline (Gitlab-Runner, Jenkins, Github Actions, Admission Controler, etc.)
2. *On-Premise* Bereitstellung (insbesondere relevant für Kritische Infrastrukturen)
3. integriertes Image Scanning 
4. integriertes Image Signing

Einige Container Registries, die diese Bedingungen erfüllen wären:

- [Harbor Container Registry](https://goharbor.io/)
  - Opensource
  - hohe Kompatibilität zu anderen Repositories (Replication Adapters) und Image Scannern (Scanner Adapters)
- [Nexus Repository](https://de.sonatype.com/products/container?topnav=true)
  - vor allem interessant als Gesamtlösung im CI/CD-Prozess (Bereitstellung von language Packages, SBOM-Validierung, Nexus Container Security)
- [Red Hat Quay](https://www.redhat.com/en/technologies/cloud-computing/quay)
- [Docker private registry server](https://docs.docker.com/registry/deploying/)
  - erfordert hohes Maß an eigenständiger Konfiguration um 3 und 4 zu erfüllen

## 4.3 Helm Repository

Auch die Ansammlung von YAML-Konfigurationsdateien zur Definition von Deployments, Services, Nutzern, Volumes und weiteren Kubernetes-Komponenten sollte zentral abgespeichert werden, sodass einerseits deren Konfiguration auditiert und andererseits die Wiederverwendbarkeit von Kubernetes-Komponenten verbessert wird.

**Helm Charts** haben sich als Format für Kubernetes-YAML-Dateien etabliert. Genauso wie Dockerhub von der Allgemeinheit genutzt werden kann um Container Images zu teilen, erlaubt Artifacthub die Bereitstellung von Helm Charts. Somit müssen öffentliche Helm Charts gleichermaßen geprüft werden, bevor sie in ein lokales Repository gezogen werden. [Helm]

Sowohl Harbor, als auch Nexus können mit als Helm Repository verwendet werden.

## 4.4 Admission Control

Mithilfe eines *Admission Controllers* können feingranulare Policies für die Erstellung von Ressourcen in einem Cluster festgelegt werden. Insbesondere lässt sich hiermit prüfen, ob ein Container Image bestimmte Spezifikationen erfüllt, bevor dieses im Cluster bereitgestellt wird (bspw. Signaturprüfung s. 4.1). [Rice20]

Admission Controller werden in zwei Kategorien unterteilt: *Validating* und *Mutating*. Dabei eignen sich *Mutating Admission Controller* für Automationsaufgaben, wie die automatische Zuweisung einer ``DefaultStorageClass`` wenn in einem *Persistent Volume Claim* keine spezifiziert wurde. 

Kubernetes stellt eine geringe Menge von optionalen *Admission Controllers* zur Verfügung [K8S_AC]. Im Allgemeinen reichen diese jedoch nicht aus, um alle gewünschten Policies im Cluster abzubilden. Stattdessen kann ein externer *Admission Controller* als Webhook in der Kubernetes API eingebunden werden, der sämtliche Anfragen an das Cluster validiert und/oder abändert. Zwei bekannte Optionen hierfür sind:

- [Open Policy Agent Gatekeeper](https://open-policy-agent.github.io/gatekeeper/website/docs/) 
  - Erstellung von Regeln in Rego-Syntax
  - große vordefinierte Regelbibliothek
- [Connaisseur](https://sse-secure-systems.github.io/connaisseur/v2.7.0/) 
  - spezialisiert auf die Überprüfung von Image Signaturen

## 4.5 Image Scanning

aqua, trivy

```
trivy fs
trivy image
trivy repo
```

- Container Registry

## 4.6 Hinweise zum Build-Prozess und der Gestaltung von Images

In Kapitel 3 wurde bereits das Thema **Immutable Containers** und Möglichkeiten für die Durchsetzung dieses Prinzips zur Laufzeit besprochen. Die beschriebenen Maßnahmen können tiefergreifend verstärkt werden, indem ein Container Image auch nur die Laufzeitumgebung und Bibliotheken enthält, die die Anwendung benötigt. Selbst das minimalistische Basis-Image **Alpine** enthält eine große Menge von typischen Linux-Befehlen wie ``ls``, ``cat``, ``mount`` und ``sh``, welche einem Angreifer genügend Möglichkeiten bieten, um sich auf dem System umzusehen und ggf. seine Privilegien zu eskalieren. **Reverse Shells** greifen üblicherweise darauf zurück einen Shell-Prozess zu starten. Das heißt, würde man die Shell-Binärdatei garnicht erst im Container bereithalten, ist es auch wesentlich schwieriger die Anwendung als solche zu kompromittieren. 

**Distroless** Basis-Images greifen genau diese Problematik auf und reduzieren die Angriffsfläche dadurch, dass sie nur die notwendige Laufzeitumgebung beinhalten. Der Unterschied fällt besonders stark im Vergleich des ``node:18`` Basis-Images auf Dockerhub mit dem ``gcr.io/distroless/nodejs18-debian11`` Image von Distroless auf. [Distr], [Rice20]

``` bash
docker images -a
# REPOSITORY                            TAG       IMAGE ID       CREATED          SIZE
# ...
# node                                  18        e390ceb99781   13 days ago      991MB
# gcr.io/distroless/nodejs18-debian11   latest    34e1fabd14c3   52 years ago     160MB
```

![Abbildung 2: Layer Inhalt Node Basis-Image (``dive node:18``)](Doc/Images/dive_node.PNG)

![Abbildung 3: Layer Inhalt Distroless Node Basis-Image](Doc/Images/dive_distroless.PNG)

Bei der Bereitstellung einer Anwendung in einem Container-Image sind in der Regel zusätzliche Build-Schritte notwendig. Um bei dem Beispiel einer Node-Anwendung zu bleiben, müssten in diesem Fall zunächst alle Abhängigkeiten über ein ``npm install`` installiert werden. Der Node Package Manager ist jedoch zur Laufzeit nicht mehr notwendig und sollte deswegen auch nicht auf dem finalen Image enthalten sein. Gleiches gilt für Compiler und ähnliche Werkzeuge zur Fertigung einer Binary (z.B. in Go, C, etc.). Aus diesem Grund sollte auf **Multi-Stage Builds** zurückgegriffen werden, welche ein temporäres Image für den Buildprozess (Kompilierung, Dependency-Installation) erstellen und die fertige Anwendung anschließend in ein minimalistisches Basis-Image (wie distroless) einfügen:

```Dockerfile
# Beispiel Multi-Stage Dockerfile von distroless

FROM node:18 AS build-env
ADD . /app
WORKDIR /app
RUN npm install --omit=dev

FROM gcr.io/distroless/nodejs18-debian11
COPY --from=build-env /app /app
WORKDIR /app
EXPOSE 3000
CMD ["hello_express.js"]
```

Die übliche Verwendung von ``docker build`` zur Erstellung von Container Images ist mit Vorsicht zu betrachten. Schließlich übersetzt das CLI-Tool *docker* lediglich den Befehl für den *Docker Daemon*, welcher bekanntermaßen unter dem Root-User läuft. Für die Erzeugung von Container Images sind jedoch keine administrativen Rechte erforderlich (sondern nur für das Deployment). Dementsprechend wäre ein Tool für *rootless builds* (bzw. *daemonless builds*) in Erwägung zu ziehen (bspw. [buildah](https://github.com/containers/buildah) oder [podman](https://docs.podman.io/en/latest/)). [Rice20]


- kics





