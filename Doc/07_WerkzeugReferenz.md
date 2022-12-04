# 7. Referenz hilfreicher Werkzeuge

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



Buildah, Connaiseur, Dive, Grype, Skopeo, eBPF
