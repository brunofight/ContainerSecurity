# 1. Einleitung

Containerisierte Anwendungen haben mit der Einführung von Docker (2013) einen großen Aufschwung erlebt. Schnelle Deployment-Zeiten, hohe Portabilität nach dem Prinzip "Build once, run anywhere" und Ressourceneffizienz machen die neue Technologie attraktiver als herkömmliche Virtualisierung. Insbesondere der geringe Speicherbedarf eines einzelnen Containers ermöglicht erst praktikable MicroService-Architekturen.

Während ein Großteil der Privatwirtschaft bereits seit einigen Jahren von den Vorzügen der Containerisierung profitiert, haben Betreiber Kritischer Infrastrukturen , mangels Sicherheitsvorgaben des BSI, sich von der Thematik fern gehalten. Wie sich in der Ausarbeitung herausstellen wird, sind Container nur oberflächlich mit Virtuellen Maschinen vergleichbar.

Erst in der 2022 Version des IT-Grundschutzkompendiums wurden Sicherheitsvorgaben für Containerisierung in einem Baustein (SYS.1.6) erfasst. Umsetzungshinweise gibt es bisher nicht. 

Das Ziel dieser Recherchearbeit liegt folglich einerseits in der Erarbeitung der technischen Besonderheiten einer Container-Infrastruktur (s. Kapitel 2) und deren Sicherheitsimplikationen, soll andererseits gleichzeitig die Anforderungen des neuen BSI-Bausteins berücksichtigen und erfüllen. Dennoch bleibt das primäre Anliegen eine operative IT-Sicherheit für eine Container-Infrastruktur zu erarbeiten (Kapitel 3-5). In Kapitel 6 werden die Baustein-Anforderungen, bezugnehmend auf die bisherigen Abschnitte, analysiert und abgebildet. 

Im Rahmen der Arbeit werden sowohl Aspekte der Sicherheit von Containern zur Laufzeit (Kapitel 3), als auch deren sichere Bereitstellung im Rahmen der Software-Supply-Chain (Kapitel 4) untersucht. Zusätzlich verdeutlicht Kapitel 5 in Referenzangriffsszenarien, wie böswillige Akteure undurchdachte Container-Infrastrukturen ausnutzen könnten. Abschließend werden in Kapitel 7 eine Reihe hilfreicher Tools mit deren Anwendungsmöglichkeiten aufgeführt.
