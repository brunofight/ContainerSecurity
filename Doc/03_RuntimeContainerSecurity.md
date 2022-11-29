# 3. Containersicherheit zur Laufzeit

In diesem Kapitel werden einige Mechanismen zur Gewährleistung der Containerisolation und Unveränderlichkeit von Containern zur Laufzeit betrachtet. Letzeres ist auch unter dem Begriff *Drift Prevention* bekannt. Die Analyse erfolgt anhand eines lokalen Minikube-Clusters (s. Kapitel 7.2 zur Referenz).

Seit einigen Jahren ist es möglich mit dem extended Berkeley Packet Filter (eBPF) beliebige Events im Kernel auszuwerten und Funktionalität basierend auf diesen hinzuzufügen. Während es komplex ist eigenhändig eBPF-Programme für den Kernel zu schreiben, gibt es bereits eine Vielzahl von Programmen, welche Detailwissen über den Kernel selbst über eine abstrahierte Schnittstelle verbergen. Das Open-Source-Projekt *Cilium* bietet ein breites Anwendungsspektrum für eBPF (s. Kapitel 3.3).

Dennoch lohnt es sich einige herkömmliche Ansätze zur Berechtigungsrestriktion, wie *seccomp*, *AppArmor* und *SELinux* anzusehen (Kapitel 3.2 und 3.3). 

## 3.1 Grundlegende Konfiguration


## 3.2 Secure Computing Mode (seccomp)

Der Linux-Kernel stellt mit Secure Computing Mode (seccomp) ein Feature zur Beschränkung der von einem Prozess ausführbaren syscalls bereit. 

- default docker seccomp Profile --> 44 syscalls blockiert
- 2022 ca. 400 syscalls
- laut aquasec benötigt Container zw. 40 und 70 syscalls --> default Profile unzureichend
- docker seccomp json dokument
  - SCMP_ACT_KILL, SCMP_ACT_TRAP, SCMP_ACT_ERRNO, trace, allow, log
- strace verwenden um syscalls eines containers zu profilen ``strace -qc time``, ``strace -c -f -S name time 2>1&1 1>/dev/null | tail -n +3 | head -n -2 | awk '{print $(NF)}'``


```bash
# cyberbit training
sudo docker run --security-opt seccomp=/home/cyberuser/profiles/violation.json --name cyberbit -dit busybox:latest

strace -c -f -S name <command line name> 2>&1 1>/dev/null | tail -n +3 | head -n -2 | awk '{print $(NF)}'
```

## 3.3 Linux Security Modules



## 3.4 Extended Berkeley Packet Filter mit Cilium

- Konfiguration
- Netzwerk
- Side-Car Container

- im Kontext eines Kubernetes-Clusters


