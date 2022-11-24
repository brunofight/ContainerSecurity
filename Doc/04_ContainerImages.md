# 4. Container Images

Container Images bilden die Verzeichnisstruktur ab, auf die eine containerisierte Anwendung zur Gewährleistung von dessen Funktionalität zurückgreift. Es ist dabei gängige Praxis die Abhängigkeiten der Anwendungen (bspw. Laufzeitumgebung) aus einem Basis Image zu beziehen und dieses gemeinsam mit der Anwendung in ein neues Container Image zu bündeln (**build**). 

Das fertige Container Image wird anschließend entweder direkt ausgerollt oder zunächst in einer Container Registry abgelegt. Container Images sind somit das zentrale Artefakt in der CI/CD-Pipeline und sind in jedem Schritt bis hin zum Deployment in ein Cluster Bedrohungen ausgesetzt (s. Abbildung). Die Kürzel CICD-SEC-x nehmen Bezug auf die **Top 10 OWASP CI/CD Security Risks**. Von besonderer Bedeutung ist das Risiko CICD-SEC-1, welches bei Nichtbeachtung Angreifern ermöglicht, von einem beliebigen System im Build-Prozess aus, schadhaften Quelltext unkontrolliert in die Produktion auszurollen. [OWASPCD], [Rice20]

Dementsprechend sind den Verifikationsmaßnahmen von Container-Images ein hohes Gewicht beizumessen. In den folgenden Unterkapiteln werden 4 Maßnahmen beschrieben, um sowohl das Deployment schadhafter Container zu verhindern als auch Schwachstellen und Fehlkonfigurationen zu vermeiden.

![Abbildung: Container Images in CI/CD](/Images/Container_CICD.png)


## 4.1 Image Signatur und Verifikation

(TUF, Notary)

## 4.2 Admission Control

(Connaiseur)

## 4.3 Image Scanning

aqua, trivy
- Container Registry

## 4.4 Sicherungsmaßnahmen im Build-Prozess

- kics, rootless builds, buildah



