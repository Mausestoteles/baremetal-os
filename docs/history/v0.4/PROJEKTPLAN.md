> Historischer Entwurf 0.4; nicht mehr normativ. Aktueller Stand: [Project OS](../../../README.md).

# Projektplan 0.4

Stand: 09.09.2026. Entwurf zur gemeinsamen Ausarbeitung.

## 1. Bestätigtes Ziel

Wir planen ein eigenes Betriebssystem mit selbst entwickeltem Kernel, das langfristig direkt auf echter Hardware läuft. Das Gesamtziel ist eine vollständige Betriebssystemplattform auf Windows-Niveau. Aktuell werden ausschließlich Plan und Spezifikationen erarbeitet.

Bestätigt sind inzwischen x86-64-Desktop als erste Plattform, Entwicklung zunächst in einer VM und danach auf ausgewählter echter Hardware. Bootloader, Standardbibliotheken, Grafikoberfläche und zentrale Anwendungen werden ebenfalls selbst entwickelt. Externe Entwicklungswerkzeuge sind erlaubt. Der Windows-Vergleich betrifft zunächst Komfort und Funktionsumfang mit nativen Apps; Windows-Apps werden später gesondert bewertet.

Das Projekt wird ausdrücklich als professionelle, ernstzunehmende Konkurrenz zu Windows geplant. Erstes nutzbares Produkt ist ein allgemeiner Desktop mit Fenstern, Dateimanager, Einstellungen, Terminal und Texteditor. Der [Produktentwurf](PRODUKT-DESKTOP-01.md) konkretisiert diesen Umfang; die [Qualitätsplanung](QUALITAET-UND-RELEASES.md) beschreibt die vorgesehenen Reifekriterien.

Weitere bestätigte Grenzen: Vorhandene Hardware und Firmware werden vorausgesetzt; unser Bootloader ist von Anfang an vorgesehen. Im eigenen Kernel dürfen Bibliotheken und Code übernommen werden. Sämtliche Systemdienste und Standardbibliotheken werden selbst entwickelt. Kompatibilität mit Windows-Treibern ist ein eigenes ausdrückliches Projektziel und beeinflusst die Architektur bereits in P0.

„Am Ende“ wird als langfristiges Projektziel verstanden. Ein Fertigstellungstermin wurde nicht vereinbart.

## 2. Das Zielbild messbar machen

Die folgenden Dimensionen sind ein Vorschlag zur Konkretisierung von „Windows-Level“. Sie legen weder Windows-Interna noch Windows-Kompatibilität fest.

| Dimension | Zu definierendes Ergebnis | Späterer Nachweis |
| --- | --- | --- |
| Systemkern | Speicher, Prozesse, Threads, Systemaufrufe, Interrupts, Mehrkernbetrieb | Isolations-, Last- und Fehlertests auf festgelegten Systemen |
| Hardware | Dokumentierte Unterstützung ausgewählter Geräte und Plattformen | Hardwarematrix mit getesteten Gerätekennungen und Firmwareständen |
| Desktop | Anmeldung, Fenster, Eingabe, Einstellungen, Dateiverwaltung, Zwischenablage | Wiederholbare Aufgaben mit mehreren Anwendungen |
| Anwendungen | Native Programme, Bibliotheken, SDK und Verteilung | Ein separat entwickeltes Programm lässt sich bauen, installieren und ausführen |
| Datenspeicherung | Dateien, Verzeichnisse, Rechte, dauerhafte Speicherung, Wiederherstellung | Neustart-, Fehler- und definierte Stromausfalltests |
| Kommunikation | Netzwerk, Namensauflösung und sichere Anwendungsprotokolle | Interoperabilitäts- und Fehlertests gegen festgelegte Gegenstellen |
| Sicherheit | Benutzer, Rechte, Isolation, geschützte Systemänderungen | Bedrohungsmodell und negative Zugriffstests |
| Wartung | Installation, Aktualisierung, Rückkehr zu einem funktionierenden Stand, Diagnose | Installation auf leerem Testmedium und Wiederherstellung nach fehlgeschlagenem Update |
| Alltagstauglichkeit | Audio, Energieverwaltung, Internationalisierung, Barrierefreiheit | Definierte Nutzungsszenarien und unterstützte Konfigurationen |
| Kompatibilität | Native Apps; weitere APIs, Dateiformate und Binärprogramme je nach Entscheidung | Benannte Programme und Versionen in einer Kompatibilitätsmatrix |

Jede Dimension braucht einen zugesagten Umfang. „Alle PCs“, „alle Programme“ oder „so schnell wie Windows“ sind ohne Referenzsysteme und Testfälle keine abnehmbaren Anforderungen.

## 3. Grenzen von „from scratch“

Die Grenzen sind nach der ersten Entscheidungsrunde wie folgt festgehalten:

| Ebene | Zu entscheiden |
| --- | --- |
| Hardware und Firmware | Vorhandene CPU, Geräte und Firmware werden vorausgesetzt. Bestätigt. |
| Bootloader | Von Beginn an selbst entwickeln; bereits der erste geplante Bootpfad nutzt unseren Loader. Bestätigt. |
| Kernel | Eigener Kernel; Bibliotheken und Code dürfen übernommen werden. Bestätigt. |
| Systemdienste und Standardbibliotheken | Sämtliche Systemdienste und Standardbibliotheken selbst entwickeln. Bestätigt; genauer Funktionsumfang wird je Spec definiert. |
| Treiber | Kompatibilität zu Windows-Treibern ist bestätigt. Weitere Treiberwelten, unterstützte Modelle, Versionen und Geräte bleiben zu konkretisieren. |
| Desktop und Anwendungen | Eigene Grafikoberfläche und zentrale Anwendungen bestätigt; genaue App-Liste und Browser-/Medienumfang offen. |
| Entwicklungswerkzeuge | Vorhandene Entwicklungswerkzeuge erlaubt; konkrete Toolchain noch auszuwählen. |
| Eigene Toolchain | Ein eigener Compiler und Bauen des OS unter sich selbst sind getrennte mögliche Langzeitziele. |

Vorschlag: Standards und dokumentierte Schnittstellen dürfen genutzt werden. Die Verwendung eines Standards und die Übernahme einer bestehenden Implementierung werden getrennt entschieden.

Übernommene Bibliotheken und Code im eigenen Kernel sind ausdrücklich erlaubt. Das Komponentenverzeichnis dokumentiert ihre Herkunft und Rolle. Die eigene Implementierung sämtlicher Systemdienste und Standardbibliotheken bleibt zugleich gesetzt; Compiler-Hilfsroutinen, Sprachkernbibliotheken, Schriftdateien und Gerätefirmware werden sachlich zugeordnet.

## 4. Vorläufige Entwicklungsbasis

**Bestätigt:** Desktop-System für x86-64, zuerst VM, danach ausgewählte echte Hardware. **Ergänzender Vorschlag:** UEFI als Firmware-Schnittstelle, QEMU als reproduzierbare Testumgebung und früher Starttest auf einem festgelegten echten Rechner.

Die VM dient als Entwicklungs- und Testumgebung. Die Fähigkeit, auf echter Hardware ohne darunterliegendes Host-Betriebssystem zu laufen, bleibt ein eigenes Abnahmekriterium.

Für x86-Systemdetails verwenden wir die [Intel-Systemprogrammierungsreferenz](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html). UEFI und ACPI werden gegen die [offiziellen Spezifikationen](https://uefi.org/specifications) geplant. Konkrete Spezifikationsversionen und die tatsächlich benötigten Funktionen werden erst mit dem Plattformprofil festgeschrieben.

Programmiersprache, Kerneltyp, Bootloader-Schnittstellen, Binärformat und Dateisystem bleiben offen. Sie werden gemeinsam anhand der Ziele und ihrer Konsequenzen ausgewählt. Der Bootloader selbst wird gemäß Entscheidung selbst entwickelt.

Die [Windows-Treiberplanung](WINDOWS-TREIBERKOMPATIBILITAET.md) ist jetzt eine Eingabe für diese Auswahl. Windows-Anwendungskompatibilität bleibt ein davon getrennter späterer Bewertungsbereich.

## 5. Etappen mit überprüfbarem Ergebnis

Die Reihenfolge ist ein Vorschlag und bildet technische Abhängigkeiten ab. Sie enthält noch keine Zeit- oder Aufwandszusage. Spätere Etappen werden vor ihrem Beginn detailliert geplant.

| Etappe | Ergebnis und Umfang | Vorgeschlagenes Abnahmekriterium |
| --- | --- | --- |
| P0 – Planungsbasis | Zielplattform, Eigenentwicklungsgrenzen, Windows-Treiberziel und erste Architektur; Ressourcenstatus dokumentiert | Grundsatzentscheidungen dokumentiert; Treiberanforderungen auf Kernelmechanismen abgebildet; Boot-, Diagnose- und Plattform-Spec für P1 ausreichend konkret |
| P1 – Eigenständiger Start | Eigener Bootloader lädt den eigenen Kernel, definierter Stack und Speicherzustand, frühe Diagnose, kontrollierter Halt | Start aus einem dokumentiert erzeugten Image in der festgelegten VM über den eigenen Loader; Kernel meldet Übernahme; absichtlicher Fehler liefert Diagnose |
| P2 – Kernelgrundlage und erste Hardware | Physischer und virtueller Speicher, Exceptions, Interrupts, Timer, initiale CPU-Erkennung und minimaler Hardwarezugriff | Reservierter Speicher bleibt geschützt; Allokationen bestehen Grenztests; eigener Kernel startet zusätzlich auf einem benannten echten Rechner |
| P3 – Geschützte Programme | Prozesse, Threads, Scheduler, User Mode, Systemaufrufe, IPC, Programmstart und erste Laufzeitbibliothek | Zwei getrennte User-Programme laufen; ungültiger Speicherzugriff eines Programms beendet dieses kontrolliert; andere Programme und Kernel bleiben funktionsfähig |
| P4 – Dauerhaft nutzbares Basissystem | Treiberschnittstellen, Blockzugriff, VFS, erstes persistentes Dateisystem, Konsole, Shell und native Werkzeuge | Eine Datei wird aus einem User-Programm geschrieben und nach Neustart identisch gelesen; Fehlerpfade sind geprüft; dokumentiertes Konsistenzmodell existiert |
| P5 – Grafischer Arbeitsplatz | Grafik- und Eingabepfad, Compositor/Fensterverwaltung, Toolkit, Terminal, Dateimanager, Einstellungen | Mehrere native GUI-Programme nutzbar; Tastaturfokus und Fensterverwaltung funktionieren; ein App-Absturz beendet die Sitzung nicht |
| P6 – Vernetztes System | Netzwerkpfad, Protokolle, DNS, sichere Verbindungen und passende Nutzerwerkzeuge | Definierte Gegenstellen erreichbar; Verbindungsabbrüche, fehlerhafte Pakete und negative Zertifikatstests behandelt |
| P7 – Installierbare Vorschau | Installer, Benutzerverwaltung, Softwarepakete, Updates, Rückkehr zum letzten funktionsfähigen Stand und Wiederherstellung | Installation auf leerem Testmedium; Update und provozierter Updatefehler mit anschließender Wiederherstellung erfolgreich getestet |
| P8 – Alltagssystem auf Referenzhardware | Mehrkernbetrieb ausgereift, gezielte Hardwareerweiterung, Audio, Energieverwaltung, Zugänglichkeit, Leistungs- und Stabilitätsarbeit | Abgenommene Alltagsszenarien, Hardwarematrix, Last- und Langzeittestziele erreicht; kritische offene Fehler bewertet |
| Langfristiger Ausbau | Breiteres Geräte- und App-Angebot, mögliche Windows-Kompatibilität und gegebenenfalls eigene Toolchain | Eigene Teilprogramme mit klarer Zielmatrix; keine pauschale Zusage vollständiger Windows-Gleichwertigkeit |

Abhängigkeiten: P1 → P2 → P3 → P4. P5 und P6 können auf P4 aufbauen. P7 integriert die bis dahin vorgesehenen Komponenten. Sicherheit, Diagnose und automatisierbare Tests begleiten alle Etappen. Erste Mehrkern-Unterstützung kann vor P8 entstehen; ihr genauer Zeitpunkt wird mit Scheduler und Speichermodell entschieden.

Zusätzlich entsteht ein durchgehender Windows-Treiberbereich: Zielmodell und Kernelverträge in P0, erste Ausführung gezielter Testtreiber nach den benötigten Kernelgrundlagen, anschließend ein benannter Hardwaretreiber und stufenweiser Ausbau. Die genaue Integration in P3/P4 hängt vom gewählten Testtreiber ab. Windows-Treiberkompatibilität wird damit früh entworfen; eine vollständige Unterstützung aller Treiber ist kein zugesagter Meilenstein.

Installation, Rechte, Updates und Geräteverwaltung brauchen frühe Schnittstellenentscheidungen, auch wenn die vollständigen Funktionen erst später geliefert werden. P3 benötigt bereits einen einfachen Programmlader und eine minimale Laufzeit; das umfassende SDK wächst ab dort mit.

## 6. Was zuerst spezifiziert wird

1. Das bestätigte erste Desktop-Produkt in Schnittstellen und abnehmbare Nutzungsszenarien übersetzen.
2. Referenzplattformen und ein erstes Windows-Treiber-Kompatibilitätsprofil einschließlich Geräte-/Framework-Abhängigkeiten.
3. Kernelarchitektur mit Verantwortlichkeiten, Schutzgrenzen, Kompatibilitätsverträgen und zulässigen Abhängigkeiten.
4. Bootvertrag: Übergabeformat, Speicherbesitz, CPU-Zustand, Firmwareübergang und Fehlerdiagnose.
5. Entwicklungsablauf: Host, Werkzeuge, reproduzierbare Builds, Testausführung und Debugging.
6. Grundlegendes Sicherheitsmodell, das Speicher-, Prozess- und Schnittstellendesign beeinflusst.

Spätere Funktionen bekommen jetzt Anforderungen und offene Fragen. Detaillierte interne Strukturen werden dann festgeschrieben, wenn ihre Voraussetzungen ausreichend geklärt sind.

## 7. Aufwand und Planungssicherheit

Ein kleiner startfähiger Kernel und eine ausgereifte Desktop-Plattform sind unterschiedlich große Ziele. Für die Gesamtvision rechnen wir vorläufig mit einem mehrjährigen Horizont. Das ist eine Planungseinschätzung, keine belastbare Aufwandsschätzung oder Fertigstellungszusage.

Der Projektinhaber hat den professionellen Produktanspruch bekräftigt und keine Angaben zu Zeitbudget oder Erfahrung gemacht. Wir setzen die fachliche Planung ohne weitere Abfrage dazu fort. Ressourcen und Termine bleiben unbeziffert. Wenn später ein Kalenderplan gewünscht ist, muss er auf einer Kapazitätsgrundlage und beobachtetem Fortschritt beruhen.

Die größten offenen Umfangstreiber sind Hardwarebreite, moderne Grafikbeschleunigung, Windows-Treiberkompatibilität, Browserumfang und die vollständige eigene Implementierung von Systemdiensten und Standardbibliotheken. Eine eigene Compilerentwicklung ist weiterhin nur ein mögliches separates Langzeitziel. Dafür werden separate Ziele und Entscheidungspunkte benötigt.

## 8. Definition einer abgeschlossenen Etappe

Vorschlag für alle späteren Implementierungsetappen:

- Die vereinbarten Anforderungen und Schnittstellen sind dokumentiert und umgesetzt.
- Ein frischer Build lässt sich mit den dokumentierten Werkzeugversionen erzeugen.
- Die zur Etappe gehörenden Funktions- und Fehlerprüfungen laufen auf den benannten Profilen.
- Bekannte Einschränkungen sind sichtbar und widersprechen keinem Abnahmekriterium.
- Änderungen an früheren Entscheidungen und Schnittstellen sind nachvollziehbar dokumentiert.

Ein erfolgreicher Start allein bestätigt weder die allgemeine Stabilität noch die Sicherheit des gesamten Systems.
