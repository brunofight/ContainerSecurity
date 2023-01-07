# 5. Angriffsszenarien

Dieses Kapitel zeigt mögliche Angriffspfade auf, die begünstigt durch Schwachstellen oder eine Kette dieser, zu einer Beeinflussung der drei primären Sicherheitsziele führen können. Es werden einerseits containerisierte Infrastrukturen in den Kontext der Phasen eines Angriffs gesetzt, wie bspw. im [MITRE ATT&CK Framework](https://attack.mitre.org/matrices/enterprise/containers/) oder der [Microsoft Threat Matrix für Kubernetes](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/).

Andererseits sollen konkrete Beispiele für einen erfolgreichen Angriff aufgeführt werden, welche wiederum Rückschlüsse auf IoCs für ein Sicherheitsmonitoring-System erlauben. Ferner folgt diesem Schritt die Ausarbeitung von Use-Cases und Runbooks. Wie soll man vorgehen, wenn ein verdächtiger Container im Cluster gestartet wird? Welche Schritte sind einzuleiten? Im Gegensatz zu den vorherigen Kapiteln wird davon ausgegangen, dass ein Angriff bereits stattgefunden bzw. gerade stattfindet.

Für die weitere Bearbeitung könnten folgende Ressourcen hilfreich sein:

- MITRE ATT&CK Framework
- Microsoft Threat Matrix für Kubernetes
- [Hacktrickz Docker Breakout/Privilege Escalation](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-breakout/docker-breakout-privilege-escalation#runc-exploit-cve-2019-5736)
- [Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)




