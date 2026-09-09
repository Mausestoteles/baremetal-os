> Historischer Entwurf 0.4; nicht mehr normativ. Aktueller Stand: [Project OS](../../../README.md).

# Anforderungs- und Spezifikationsrahmen 0.4

Stand: 09.09.2026. Dieses Dokument ist ein Entwurf, noch keine vollständige implementierbare Betriebssystem-Spezifikation.

## 1. Status und Begriffe

**Bestätigte Anforderungen** stammen aus dem Erstauftrag und ausdrücklich beantworteten Planungsfragen. **Vorgeschlagene Anforderungen** dienen der gemeinsamen Diskussion und werden erst nach einer Entscheidung verbindlich.

In später angenommenen Specs bedeutet MUSS eine verpflichtende Eigenschaft, SOLL eine begründbar abweichbare Empfehlung und KANN eine optionale Eigenschaft. Bis dahin bleiben auch technisch präzise formulierte Anforderungen Vorschläge.

## 2. Bestätigte Anforderungen

| ID | Anforderung | Status | Nachweis |
| --- | --- | --- | --- |
| R-001 | Das Betriebssystem besitzt einen selbst entwickelten Kernel. | Bestätigt | Architektur, Quellcodeherkunft und Buildbestandteile später nachvollziehbar |
| R-002 | Das Betriebssystem läuft auf echter Hardware ohne darunterliegendes Host-Betriebssystem. | Bestätigt | Dokumentierter Start und definierte Funktionen auf benannter Referenzhardware |
| R-003 | Das langfristige Ziel umfasst eine vollständige Plattform auf Windows-Niveau. | Bestätigte Vision; noch nicht messbar | In konkrete Funktionen, Qualitätsziele und Kompatibilitätsszenarien zerlegen |
| R-004 | Die aktuelle Projektphase erzeugt ausschließlich Planung und Spezifikationen. | Bestätigt | Projektergebnis besteht derzeit aus Planungsunterlagen |
| R-005 | Die erste Zielplattform ist ein x86-64-Desktop; Entwicklung zunächst in einer VM, danach ausgewählte echte Hardware. | Bestätigt | Plattformprofil und spätere Testnachweise |
| R-006 | Bootloader, Standardbibliotheken, Grafikoberfläche und zentrale Anwendungen werden selbst entwickelt. | Bestätigt | Komponentenverzeichnis mit nachvollziehbarer Herkunft; genaue App-Liste noch festzulegen |
| R-007 | Externe Entwicklungswerkzeuge dürfen verwendet werden. | Bestätigte Rahmenbedingung | Toolchain-Dokumentation; Laufzeitabhängigkeiten separat entscheiden |
| R-008 | Die native Plattform verfolgt vergleichbaren Komfort und Funktionsumfang; Windows-Apps werden später gesondert bewertet. | Bestätigt | Native Anwendungsszenarien; spätere eigene Kompatibilitätsentscheidung |
| R-009 | Das erste nutzbare Produkt ist ein allgemeiner Desktop mit Fenstern, Dateimanager, Einstellungen, Terminal und Texteditor. | Bestätigt | Produktumfang in PRODUKT-DESKTOP-01; dortige Detailabnahme noch Vorschlag |
| R-010 | Die Gesamtplattform soll eine professionelle, ernstzunehmende Konkurrenz zu Windows werden. | Bestätigter Produktanspruch; einzelne Qualitätsziele zu beschließen | Releasekriterien, Alltagsszenarien und späterer Funktions-/Qualitätsvergleich |
| R-011 | Vorhandene Hardware und Firmware werden vorausgesetzt; der eigene Loader ist ab dem ersten Bootmeilenstein vorgesehen. | Bestätigt | Plattform- und Bootvertrag |
| R-012 | Bibliotheken und Code dürfen im eigenen Kernel übernommen werden. | Bestätigte Rahmenbedingung | Dokumentierte Herkunft und Integration |
| R-013 | Sämtliche Systemdienste und Standardbibliotheken werden selbst entwickelt. | Bestätigt | Komponentenplan und Umsetzung je Funktions-Spec |
| R-014 | Das System soll Windows-Treiber unterstützen. | Bestätigtes Kompatibilitätsziel; Modelle, Versionen und Abnahme noch zu beschließen | SPEC-019 und konkrete Treiber-/Gerätematrix |

## 3. Vorgeschlagene überprüfbare Anforderungen

Die Nachweise beschreiben spätere Abnahmetests. Es wurden noch keine solchen Tests implementiert oder ausgeführt.

| ID | Vorgeschlagene Anforderung | Nachweisidee | Etappe |
| --- | --- | --- | --- |
| R-101 | Der eigene Bootloader lädt den eigenen Kernel; dieser übernimmt über den definierten Bootvertrag und erzeugt eine frühe Diagnoseausgabe. | Kaltstart des Testimages über unseren Loader; festgelegte Startinformationen erscheinen | P1 |
| R-102 | Reservierte, belegte und verfügbare physische Speicherbereiche werden unterschieden; ungültige Bereiche werden nicht ausgegeben. | Synthetische Speicherkarten und Allokationsgrenzfälle | P2 |
| R-103 | Ein User-Prozess kann weder Kernel-Speicher noch privaten Speicher anderer Prozesse unberechtigt lesen oder verändern. | Absichtliche Zugriffsverletzungen; fremder Prozess bleibt intakt | P3 |
| R-104 | Ungültige Systemaufrufargumente führen zu definierten Fehlern und beschädigen den Kernel nicht. | Ungültige Handles, Adressen, Längen und Grenzwerte | P3 |
| R-105 | Beendete Prozesse geben ihre Ressourcen gemäß dokumentierten Besitzregeln frei. | Wiederholtes Starten/Beenden mit Prüfung der Ressourcenbilanz | P3 |
| R-106 | Mehrere rechenbereite Programme machen nach definierten Scheduler-Regeln Fortschritt. | Konkurrenztest einschließlich Endlosschleife eines User-Programms | P3 |
| R-107 | Erfolgreich dauerhaft gespeicherte Daten überstehen Neustarts gemäß dem definierten Dateisystemvertrag. | Schreiben, explizite Dauerhaftigkeitsoperation, Neustart, Inhaltsprüfung | P4 |
| R-108 | Unterbrochene Schreibvorgänge hinterlassen einen Zustand, der dem dokumentierten Konsistenz- und Wiederherstellungsmodell entspricht. | Fehler an definierten Schreibpunkten injizieren und Wiederherstellung prüfen | P4/P7 |
| R-109 | Native Anwendungen können außerhalb des OS-Quellbaums gegen ein dokumentiertes SDK gebaut werden. | Separates Beispielprojekt bauen und ausführen | Ab P3; ausbauen bis P7 |
| R-110 | Ein Fehler in einer GUI-Anwendung beendet nicht andere Anwendungen oder die gesamte grafische Sitzung. | Anwendung absichtlich abstürzen lassen; andere Anwendung weiter bedienen | P5 |
| R-111 | Zugriffe auf geschützte Dateien, Geräte und Verwaltungsaktionen werden gemäß dem festgelegten Berechtigungsmodell geprüft. | Erlaubte und verweigerte Zugriffe mit unterschiedlichen Identitäten | Entwurf vor P3; Ausbau bis P7 |
| R-112 | Ein fehlgeschlagenes Systemupdate lässt einen dokumentierten Wiederherstellungsweg offen. | Abbruch in definierten Updatephasen; Wiederherstellung nachweisen | P7 |
| R-113 | Das System kann auf einem leeren, ausdrücklich ausgewählten Testmedium installiert werden. | Installationsablauf im benannten Testprofil | P7 |
| R-114 | Unterstützte Hardware, Einschränkungen und Testresultate werden pro Version dokumentiert. | Hardwarematrix und zugehörige Prüfprotokolle | Ab P2 |
| R-115 | Jeder veröffentlichte Stand lässt sich aus dokumentierten Quellen und festgehaltenen Werkzeugversionen bauen. | Build in einer frisch eingerichteten Umgebung | Ab P1 |

## 4. Funktionale Systemgliederung

Die folgende Gliederung beschreibt Verantwortlichkeiten. Die konkrete Zuordnung zu Kernel, Bibliotheken oder eigenen User-Prozessen bleibt bis zur Architekturentscheidung offen.

| Bereich | Verantwortlichkeiten |
| --- | --- |
| Plattform und Start | Firmwarezugang, Kernel laden, Bootinformationen, CPU-Startzustand und frühe Diagnose |
| Kernelmechanismen | Speicherisolation, Threads, Scheduling, Interrupts, Synchronisation und kontrollierte Systemzugänge |
| Kommunikation und Ressourcen | IPC, Handles/Objekte, Besitz, Lebensdauer und Berechtigungsprüfung |
| Geräte und I/O | Erkennung, Ressourcen, Treiber, DMA, asynchrone Aufträge und Gerätefehler |
| Systemdienste | Prozesse starten, Dateien, Netzwerk, Benutzer, Konfiguration, Zeit und Dienste verwalten |
| Anwendungslaufzeit | Programmlader, Bibliotheken, native API/ABI, SDK und Paketmetadaten |
| Desktop | Grafische Ausgabe, Eingabe, Fenster, Toolkit, Sitzung, Zugänglichkeit und Basisanwendungen |
| Wartung | Installation, Updates, Wiederherstellung, Protokolle und Diagnosewerkzeuge |

Insbesondere sagt diese Tabelle noch nicht, dass Dateisysteme, Netzwerk oder Treiber im Kernel laufen. Schutzgrenzen werden in einer eigenen Architekturentscheidung mit Fehlerverhalten und Kommunikationswegen festgelegt.

## 5. Katalog der technischen Specs

Alle Einträge sind geplante Dokumente. Ihre Inhalte sind noch auszuarbeiten; die Bezeichner sind Vorschläge.

| Spec | Inhalt | Spätestens zu konkretisieren |
| --- | --- | --- |
| SPEC-001 Plattformprofil | CPU-Baseline, Firmware, VM-Version/-Geräte, reale Referenzhardware, Speicher und Bootmedium | Vor P1 |
| SPEC-002 Bootvertrag | Kernelabbild, Ladevorgang, Übergabestruktur, Speicherbesitz, CPU-Zustand, Firmwareübergang und Fehler | Vor P1 |
| SPEC-003 Diagnose | Frühe Ausgabe, Panics, Fehlerdaten, Debugzugang und später strukturierte Logs | Vor P1; laufend erweitern |
| SPEC-004 Build und Tests | Host, Toolchain-Versionen, Buildgraph, Images, Testprofile und reproduzierbare Ausführung | Vor P1 |
| SPEC-005 Speicher | Physischer Allokator, virtuelle Adressräume, Mappingrechte, Kernel-Heap und Grenzfälle | Vor P2 |
| SPEC-006 CPU und Interrupts | Exception-/Interruptpfade, Timer, Controller, Mehrkernstart und Synchronisation | Vor P2; SMP-Vertrag vor Mehrkernbetrieb |
| SPEC-007 Prozesse und Scheduler | Prozess-/Threadzustände, Kontextwechsel, Prioritäten, Warteoperationen und Ende | Vor P3 |
| SPEC-008 Systemaufrufe und IPC | Nummerierung/Versionierung, Argumente, Fehler, Handles, Kopierregeln, Nachrichten und Abbruch | Vor P3 |
| SPEC-009 Sicherheitsmodell | Vertrauensgrenzen, Identitäten, Rechte, Bedrohungsmodell und geschützte Operationen | Architektur vor P3; laufend vertiefen |
| SPEC-010 Programme und SDK | Binärformat, Lader, Stack/TLS, statisches/dynamisches Linken, Laufzeit und Kompatibilität | Minimalumfang vor P3 |
| SPEC-011 Treibermodell | Erkennung, Initialisierung, I/O, DMA, Interrupts, Lebensdauer, Fehler und Energiezustände | Grundvertrag vor erstem Gerätetreiber |
| SPEC-012 Speicherung | Blockmodell, VFS, Pfade, Metadaten, Rechte, Dateisystem und Dauerhaftigkeit | Vor P4 |
| SPEC-013 Grafik und Eingabe | Anzeigeprotokoll, Oberflächen, Compositor, Eingabe, Fokus, Schriften und Toolkit | Vor P5 |
| SPEC-014 Netzwerk | Protokolle, Sockets/Alternative, DNS, Konfiguration, sichere Verbindungen und Fehler | Vor P6 |
| SPEC-015 Pakete und Wartung | Paketformat, Installation, Updates, Vertrauen, Rückkehr zu früherem Stand und Recovery | Architektur früh; Detail vor P7 |
| SPEC-016 Desktop und Basisapps | Nutzungsszenarien, Interaktion, Einstellungen, Dateimanager, Terminal und Zugänglichkeit | Vor P5; Ausbau bis P8 |
| SPEC-017 Qualität und Releases | Leistung, Langzeittests, Hardwarematrix, Versionspolitik und Freigabekriterien | Erste Werte vor P1; je Etappe ergänzen |
| SPEC-018 Anwendungskompatibilität | Gewählte APIs/ABIs, Dateiformate, Zielprogramme und Spiele | Windows-Apps später gesondert bewerten gemäß O-003 |
| SPEC-019 Windows-Treiberkompatibilität | Binär-/Verhaltensverträge, Kernel-DDIs, Frameworks, Geräteklassen, Paketaufnahme und Referenztreiber | Architekturgrundlagen vor Kernelentscheidung; gestufte Detail-Spec |

## 6. Aufbau jeder technischen Spezifikation

1. Kennung, Status, Version und Verantwortliche.
2. Zweck, zugehörige Anforderungen und ausdrücklich definierter Umfang.
3. Abhängigkeiten und vorausgesetzte Architekturentscheidungen.
4. Begriffe und Verantwortlichkeiten.
5. Externe Schnittstellen einschließlich Datenformaten, Aufrufkonventionen und Versionierung.
6. Zustände, Übergänge und erlaubte Reihenfolgen.
7. Besitz und Lebensdauer von Speicher, Handles und anderen Ressourcen.
8. Fehlerfälle, Ressourcenmangel, Abbruch und Wiederherstellung.
9. Parallelität, Synchronisation und Schutzgrenzen, soweit relevant.
10. Konkrete Abnahmetests sowie bekannte Grenzen des Nachweises.
11. Offene Fragen, Alternativen und verknüpfte Entscheidungen.
12. Primärquellen mit verwendeter Version und relevanten Abschnitten.

Eine Spec ist umsetzungsbereit, wenn ihre Schnittstellen und wesentlichen Fehlerfälle hinreichend konkret sind und keine offene Frage das geplante Verhalten grundsätzlich verändert.

## 7. Qualitätsziele ohne erfundene Zahlen

| Eigenschaft | Vor verbindlicher Festlegung erforderlich |
| --- | --- |
| Bootzeit | Referenzhardware, Start- und Endereignis, Kalt-/Warmstart und Messverfahren |
| Arbeitsspeicherbedarf | Hardwareprofil, aktive Dienste, Sitzung und Messdefinition |
| Reaktionszeit | Nutzungsszenario, Hintergrundlast, Perzentile und Eingabe-/Ausgabemessung |
| Durchsatz | Datengröße, Cachezustand, Speicher-/Netzwerkgerät und Vergleichsbedingungen |
| Stabilität | Workload, Dauer, Ressourcenüberwachung und erlaubte Fehler |
| Wiederherstellung | Fehlerannahmen, tolerierter Datenverlust und Wiederanlaufziel |
| Hardwareumfang | Benannte Geräte/IDs, Firmware, Funktionen und Einschränkungen |
| Kompatibilität | Benannte Anwendungsversionen und zu bestehende Arbeitsabläufe |

Das erste Desktop-Produktziel ist beschlossen. Quantitative Zielwerte bleiben offen, bis Referenzplattform und Messbedingungen feststehen. Speichergrenzen für Boot und frühe Tests werden vor P1 ausdrücklich festgelegt.
