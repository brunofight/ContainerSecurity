# 2. Grundlegende Konzepte

Dieses Kapitel gibt einen kurzen Überblick zu den fundamentalen Prinzipien und der technischen Implementierung, die mit der Containerisierung von Anwendungen einhergeht. Leser, die mit Linux *Capabilities*, *cgroups* und *namespaces* vertraut sind können mit [3. Containersicherheit zur Laufzeit](Doc/03_RuntimeContainerSecurity.md) fortfahren. 

In seinem Kern ist ein Container nur ein Prozess, der auf einem Linux Kernel läuft. Gerade deshalb sind Container so effizient mit Deployment-Zeiten im Millisekundenbereich und in ihrer Ressourcennutzung. Das steht im harten Kontrast zu Virtuellen Maschinen, die jeweils mit einem eigenen Betriebssystem gestartet werden.

Aus dem Winkel der IT-Sicherheit betrachtet ergibt sich mit Containern wiederum eine wesentlich höhere Komplexität zur Erreichung der vermutlich wichtigsten Grundbedingung für den Einsatz containerisierter Anwendungen - die Isolation. Ein Hypervisor kommt mit gerade einmal 50.000 Zeilen Quelltext aus und hat eine wesentlich einfachere Aufgabe bzw. ist es nicht einmal vorgesehen, dass VMs in irgendeiner Weise direkt miteinander interagieren (Netzwerkverbindung ausgenommen). [Rice20], [Xen19]

Je nach Version des verwendeten Linux Kernels besteht dieser aus 20 - 35 Millionen Zeilen Quelltext. Es ist durchaus möglich und unter Umständen auch gewollt Containern geteilte Ressourcen zur Verfügung zu stellen. Prozesse können i.A. auch andere Prozesse sehen. Zusammengefasst bedeutet das, dass mit Zunahme der Konfigurationsmöglichkeiten das Potenzial für eine Schwachstelle im Kernel Code oder eine Fehlkonfiguration steigt. [Rice20], [WiLK]

Aus diesem Grund werden zunächst die Mechanismen zur Erfüllung der Container Isolation in Kapitel 2.1 vorgestellt (weitergehende Härtungsmaßnahmen s. 3.). Kapitel 2.2 befasst sich mit der Unveränderlichkeit von Containern, einer weiteren wünschenswerten Eigenschaft von Containern und in 2.3 wird ein grober Einblick in die Terminologie von Kubernetes, der vorherrschenden Container-Orchestrierungslösung gegeben.

## 2.1 Container Isolation



## 2.2 Immutable Containers



## 2.3 Container-Orchestrierung


