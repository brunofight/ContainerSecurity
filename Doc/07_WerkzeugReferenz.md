# 7. Referenz hilfreicher Werkzeuge

Für die Erstellung, Überprüfung von Container Images, die Interaktion mit einer Registry, die Generierung von seccomp und AppArmor-Profilen und weiteren containerspezifische Aufgaben wurden viele OpenSource-Werkzeuge entwickelt. Innerhalb dieses Kapitels findet man eine Referenz zu u.a.:

- Tools des [Open Repository for Container Tools](https://github.com/containers) wie:
  - [buildah](https://github.com/containers/buildah) für rootless Container-Builds
  - [skopeo](https://github.com/containers/skopeo) - Interaktion mit Container Images und Registries
- [dive](https://github.com/wagoodman/dive) - Docker Image Layer Analyzer
- [bane](https://github.com/genuinetools/bane) - Erstellung von AppArmor Profilen
- Linux-Befehlen wie ``getcaps``, ``strace``
- Image-Scannern wie [trivy](https://github.com/aquasecurity/trivy), [grype](https://github.com/anchore/grype) oder in Teilen auch [kics.io](https://kics.io/)
- AdmissionController wie [Connaiseur](https://github.com/sse-secure-systems/connaisseur) oder [Open Policy Agent](https://www.openpolicyagent.org/)

, die als Nachschlagewerk für den Bedarfsfall gesehen werden kann. Das heißt im Mittelpunkt dieses Kapitels steht der zu erreichende Zweck und welche Tools dafür mit welchen Basisbefehlen (oder auch relevanten komplexeren Befehlen) sich als nützlich erweisen könnten.

## 7.1 Linux-Befehle

### 7.1.1 Capabilities

```bash
cat /proc/<PID>/status | grep Cap
```
```bash
getcaps <PID>
```

### 7.1.2 Syscalls


```
strace -c -f -S name <command/ app name> 2>&1 1>/dev/null | tail -n +3 | head -n -2 | awk '{print $(NF)}'
```

## 7.2 Tools

### 7.2.1 Docker Installation

1. Installation Container Runtime (hier Docker) [DInst]

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get install ca-certificates curl gnupg lsb-release

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo chmod a+r /etc/apt/keyrings/docker.gpg
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

### 7.2.2 Minikube

Donwload Minikube über [MK8Inst].

```bash
minikube start --driver=virtualbox
minikube config set driver virtualbox
```

