# 1. Einleitung

Containerisierte Anwendungen haben mit der Einführung von Docker (2013) einen großen Aufschwung erlebt. Schnelle Deployment-Zeiten, hohe Portabilität nach dem Prinzip "Build once, run anywhere" und Ressourceneffizienz machen die neue Technologie attraktiver als herkömmliche Virtualisierung. Insbesondere der geringe Speicherbedarf eines einzelnen Containers ermöglicht erst praktikable MicroService-Architekturen.

Während ein Großteil der Privatwirtschaft bereits seit einigen Jahren von den Vorzügen der Containerisierung profitiert, haben Betreiber Kritischer Infrastrukturen , mangels Sicherheitsvorgaben des BSI, sich von der Thematik fern gehalten. Wie sich in der Ausarbeitung herausstellen wird, sind Container nur oberflächlich mit Virtuellen Maschinen vergleichbar.

Erst in der 2022 Version des IT-Grundschutzkompendiums wurden Sicherheitsvorgaben für Containerisierung in einem Baustein (SYS.1.6) erfasst. Umsetzungshinweise gibt es bisher nicht. 

Das Ziel dieser Recherchearbeit liegt folglich einerseits in der Erarbeitung der technischen Besonderheiten einer Container-Infrastruktur (s. Kapitel 2) und deren Sicherheitsimplikationen, soll andererseits gleichzeitig die Anforderungen des neuen BSI-Bausteins berücksichtigen und erfüllen. Dennoch bleibt das primäre Anliegen eine operative IT-Sicherheit für eine Container-Infrastruktur zu erarbeiten (Kapitel 3-5). In Kapitel 6 werden die Baustein-Anforderungen, bezugnehmend auf die bisherigen Abschnitte, analysiert und abgebildet. 

Im Rahmen der Arbeit werden sowohl Aspekte der Sicherheit von Containern zur Laufzeit (Kapitel 3), als auch deren sichere Bereitstellung im Rahmen der Software-Supply-Chain (Kapitel 4) untersucht. Zusätzlich verdeutlicht Kapitel 5 in Referenzangriffsszenarien, wie böswillige Akteure undurchdachte Container-Infrastrukturen ausnutzen könnten. Abschließend werden in Kapitel 7 eine Reihe hilfreicher Tools mit deren Anwendungsmöglichkeiten aufgeführt.

# 1.1 Verwandte Arbeiten

Die Cloud Native Computing Foundation (CNCF) untersucht ähnlich zu dieser Ausarbeitung Sicherheitsaspekte containerisierter Infrastrukturen auf Basis ihrer Integrationsstufe innerhalb der CI/CD-Pipeline. Zusammengefasst findet man diese im [Cloud Native Security Whitepaper](https://github.com/cncf/tag-security/tree/main/security-whitepaper/v2). Im Gegensatz zur in Folge präsentierten Analyse bleibt das CNCF Whitepaper in Bezug auf eingesetzte Technologien generisch und zeigt keine Einsatzszenarien auf.

# 1.2 Information zur Dokumentation und weiteren Bearbeitung

Das Dokument Container Security entstand ursprünglich aus einem Masterprojekt an der HTWK, welches zugleich als Recherche und Grundlage zu interessanten Themen für eine Masterarbeit verwendet wurde. Als solches ist die Dokumentation (Stand Januar 2023) noch nicht vollständig. Insbesondere sind in den Kapiteln 5-7 noch weitere Ausarbeitungen erforderlich. Hinweise zur weiteren Bearbeitung des Inhalts wurden am Anfang dieser Kapitel ergänzt.

Die Dokumentation wurde im Auftrag einer öffentlichen Institution verfasst und steht somit auch der Öffentlichkeit ohne Einschränkung zur Verfügung steht.
