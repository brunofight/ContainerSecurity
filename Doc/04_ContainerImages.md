# 4. Container Images

Container Images bilden die Verzeichnisstruktur ab, auf die eine containerisierte Anwendung zur Gewährleistung von dessen Funktionalität zurückgreift. Es ist dabei gängige Praxis die Abhängigkeiten der Anwendungen (bspw. Laufzeitumgebung) aus einem Basis Image zu beziehen und dieses gemeinsam mit der Anwendung in ein neues Container Image zu bündeln (**build**). 

Das fertige Container Image wird anschließend entweder direkt ausgerollt oder zunächst in einer Container Registry abgelegt (s. Kapitel 4.2). Container Images sind somit das zentrale Artefakt in der CI/CD-Pipeline und sind in jedem Schritt bis hin zum Deployment in ein Cluster Bedrohungen ausgesetzt (s. Abbildung). Die Kürzel CICD-SEC-x nehmen Bezug auf die **Top 10 OWASP CI/CD Security Risks**. Von besonderer Bedeutung ist das Risiko CICD-SEC-1, welches bei Nichtbeachtung Angreifern ermöglicht, von einem beliebigen System im Build-Prozess aus, schadhaften Quelltext unkontrolliert in die Produktion auszurollen. [OWASPCD], [Rice20]

Dementsprechend sind den Verifikationsmaßnahmen von Container-Images ein hohes Gewicht beizumessen. In den folgenden Unterkapiteln werden 4 Maßnahmen beschrieben, um sowohl das Deployment schadhafter Container zu verhindern als auch Schwachstellen und Fehlkonfigurationen zu vermeiden.

![Abbildung: Container Images in CI/CD](Doc/Images/Container_CICD.png)


## 4.1 Image Signatur und Verifikation

Mit einer Image Signatur kann gewährleistet werden, dass ein aus einer Container Registry bezogenes Image auch dasselbe ist, welches zuvor im CI/CD-Prozess erzeugt wurde. Ohne Signatur könnte ein Angreifer mit Zugriff auf die Registry schadhafte Image-Imitate bereitstellen.

An dieser Stelle ist von der Image Digest abzugrenzen. Bei diesem handelt es sich um den Hash eines Image Manifests. Darin sind wiederum alle Layer eines Container Images mit ihrem jeweiligen Hash (*digest*) enthalten. Zusammengefasst ist es somit möglich ein spezifisches Container Image über dessen *Digest* aus einer Registry zu pullen und folglich auch dessen Integrität darüber sicherzustellen. Dennoch ist es keine gängige Praxis Images über deren Hash zu identifizieren

(TUF, Notary)

## 4.2 Container Registry

Bei der Entwicklung containerisierter Anwendungen stößt man als erstes vermutlich auf die öffentliche Container Registry Dockerhub, um von dort aus die benötigten Basis-Images zu beziehen. Dieser Ansatz ähnelt der ungeprüften Verwendung von Quelltextausschnitten, Bibliotheken oder sonstigen Abhängigkeiten aus einem offen verfügbaren Code-Repository. Ohne die Verwendung einer eigenen privaten Container Registry ist es folglich schwierig die Kontrolle darüber zu behalten welche (Basis-)Images im eigenen Cluster verwendet werden. Ferner reduziert eine private Container Registry Angriffsvektoren wie Typosquatting und DNS-Spoofing. [Rice20]

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

Auch das Sammelsurium an YAML-Konfigurationsdateien zur Definition von Deployments, Services, Nutzern, Volumes und weiteren Kubernetes-Komponenten sollte zentral abgespeichert werden, sodass einerseits deren Konfiguration auditiert und andererseits die Wiederverwendbarkeit von Kubernetes-Komponenten verbessert wird.

**Helm Charts** haben sich als Format für Kubernetes-YAML-Dateien etabliert. Genauso wie Dockerhub von der Allgemeinheit genutzt werden kann um Container Images zu teilen, erlaubt Artifacthub die Bereitstellung von Helm Charts. Somit müssen öffentliche Helm Charts gleichermaßen geprüft werden, bevor sie in ein lokales Repository gezogen werden. [Helm]

Sowohl Harbor, als auch Nexus können mit als Helm Repository verwendet werden.

## 4.4 Admission Control

(Connaiseur)

## 4.5 Image Scanning

aqua, trivy

```
trivy fs
trivy image
trivy repo
```

- Container Registry

## 4.6 Hinweise zum Build-Prozess und der Gestaltung von Images

In Kapitel 3 wurde bereits das Thema **Immutable Containers** und Möglichkeiten für die Durchsetzung dieses Prinzips zur Laufzeit besprochen. Die beschriebenen Maßnahmen können tiefergreifend verstärkt werden, indem ein Container Image auch nur die Laufzeitumgebung und Bibliotheken enthält, die die Anwendung benötigt. Selbst das minimalistische Basis-Image **Alpine** enthält eine große Menge von typischen Linux-Befehlen wie ``ls``, ``cat``, ``mount`` und ``sh``, welche einem Angreifer genügend Möglichkeiten bieten, um sich auf dem System umzusehen und ggf. seine Privilegien zu eskalieren. **Reverse Shells** greifen üblicherweise darauf zurück einen Shell-Prozess zu starten. Das heißt, würde man die Shell-Binary garnicht erst im Container bereithalten, ist es auch wesentlich schwieriger die Anwendung als solche zu kompromittieren. 

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

- kics, rootless builds, buildah





