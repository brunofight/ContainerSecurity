# 4. Container Images

Container Images bilden die Verzeichnisstruktur ab, auf die eine containerisierte Anwendung zur Gewährleistung von dessen Funktionalität zurückgreift. Es ist dabei gängige Praxis die Abhängigkeiten der Anwendungen (bspw. Laufzeitumgebung) aus einem Basis Image zu beziehen und dieses gemeinsam mit der Anwendung in ein neues Container Image zu bündeln (**build**). 

Das fertige Container Image wird anschließend entweder direkt ausgerollt oder zunächst in einer Container Registry abgelegt. Container Images sind somit das zentrale Artefakt in der CI/CD-Pipeline und sind in jedem Schritt bis hin zum Deployment in ein Cluster Bedrohungen ausgesetzt (s. Abbildung). Die Kürzel CICD-SEC-x nehmen Bezug auf die **Top 10 OWASP CI/CD Security Risks**. Von besonderer Bedeutung ist das Risiko CICD-SEC-1, welches bei Nichtbeachtung Angreifern ermöglicht, von einem beliebigen System im Build-Prozess aus, schadhaften Quelltext unkontrolliert in die Produktion auszurollen. [OWASPCD], [Rice20]

Dementsprechend sind den Verifikationsmaßnahmen von Container-Images ein hohes Gewicht beizumessen. In den folgenden Unterkapiteln werden 4 Maßnahmen beschrieben, um sowohl das Deployment schadhafter Container zu verhindern als auch Schwachstellen und Fehlkonfigurationen zu vermeiden.

![Abbildung: Container Images in CI/CD](Doc/Images/Container_CICD.png)


## 4.1 Image Signatur und Verifikation

(TUF, Notary)

## 4.2 Admission Control

(Connaiseur)

## 4.3 Image Scanning

aqua, trivy

```
trivy fs
trivy image
trivy repo
```

- Container Registry

## 4.4 Hinweise zum Build-Prozess und der Gestaltung von Images

In Kapitel 3 wurde bereits das Thema **Immutable Containers** und Möglichkeiten für die Durchsetzung dieses Prinzips zur Laufzeit besprochen. Die beschriebenen Maßnahmen können tiefergreifend verstärkt werden, indem ein Container Image auch nur die Laufzeitumgebung und Bibliotheken enthält, die die Anwendung benötigt. Selbst das minimalistische Basis-Image **Alpine** enthält eine große Menge von typischen Linux-Befehlen wie ``ls``, ``cat``, ``mount`` und ``sh``, welche einem Angreifer genügend Möglichkeiten bieten, um sich auf dem System umzusehen und ggf. seine Privilegien zu eskalieren. **Reverse Shells** greifen üblicherweise darauf zurück einen Shell-Prozess zu starten. Das heißt, würde man die Shell-Binary garnicht erst im Container bereithalten, ist es auch wesentlich schwieriger die Anwendung als solche zu kompromittieren.

**Distroless** Basis-Images greifen genau diese Problematik auf und reduzieren die Angriffsfläche dadurch, dass sie nur die notwendige Laufzeitumgebung beinhalten. Der Unterschied fällt besonders stark im Vergleich des ``node:18`` Basis-Images auf Dockerhub mit dem ``gcr.io/distroless/nodejs18-debian11`` Image von Distroless auf:

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
