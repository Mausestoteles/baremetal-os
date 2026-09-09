> Historischer Entwurf 0.4; nicht mehr normativ. Aktueller Stand: [Project OS](../../../README.md).

# SPEC-019: Windows-Treiberkompatibilität

Version: 0.1. Stand: 09.09.2026. Status: Architektur- und Anforderungsentwurf; noch keine umsetzungsreife vollständige DDI-Spezifikation.

## 1. Bestätigtes Ziel und Arbeitsannahme

Windows-Treiberkompatibilität ist ein ausdrücklich bestätigtes Projektziel. Der Kernel bleibt unsere eigene Entwicklung. Bibliotheken und Code dürfen darin übernommen werden. Sämtliche Systemdienste und Standardbibliotheken werden selbst entwickelt.

**Arbeitsannahme für diesen Entwurf:** Wir wollen vorhandene x64-Windows-Treiberbinärdateien unverändert auf unserem Betriebssystem ausführen können. Eine reine Quellcodeportierung wäre eine schwächere Form der Kompatibilität und wird getrennt ausgewiesen. Die genaue Kompatibilitätsstufe wurde noch nicht ausdrücklich gewählt.

Unterstützte Treibermodelle, Windows-/Framework-Versionen, Geräteklassen und konkrete Treiber sind noch festzulegen. „etc.“ in der Nutzervorgabe wird nicht automatisch als Zusage für weitere Betriebssysteme ausgelegt.

Windows-Anwendungskompatibilität bleibt ein separates späteres Thema. Benötigt ein ausgewählter Treiber zusätzliche Windows-User-Mode-Komponenten, werden deren Anforderungen ausdrücklich Teil seines Profils.

## 2. Technischer Umfang

WDM-Treiber verwenden Betriebssystemroutinen und Betriebssystemstrukturen. WDF ergänzt eigene Objekt-, Callback- und Warteschlangenverträge. Das Laden einer Datei und Auflösen von Funktionsnamen erfüllt deshalb noch nicht den gesamten Treibervertrag. Quelle: [Microsoft: Unterschiede zwischen WDM und WDF](https://learn.microsoft.com/en-us/windows-hardware/drivers/wdf/differences-between-wdm-and-kmdf).

Für ausgewählte x64-Treiber müssen mindestens deren verwendete Aufrufkonventionen und Datenlayouts berücksichtigt werden. Referenz für Aufrufe: [Microsoft x64 Calling Convention](https://learn.microsoft.com/en-us/cpp/build/x64-calling-convention?view=msvc-170).

KMDF/UMDF besitzen Framework-Versionen; die erforderliche Version und unterstützten Funktionen gehören zum Kompatibilitätsprofil. Quelle: [Framework Library Versioning](https://learn.microsoft.com/en-us/windows-hardware/drivers/wdf/framework-library-versioning).

Grafik ist ein eigener großer Bereich: WDDM umfasst zusammenwirkende Kernel- und User-Mode-Komponenten, darunter Grafik-Scheduling und Videospeicherverwaltung. Ein funktionierender einfacher Gerätetreiber belegt daher keine Kompatibilität zu modernen GPU-Treiberpaketen. Quelle: [Microsoft: WDDM-Architektur](https://learn.microsoft.com/en-us/windows-hardware/drivers/display/windows-vista-and-later-display-driver-model-architecture).

## 3. Vorgeschlagene Architektur

Der Entwurf sieht eine eigene native Plattform mit einer ausdrücklich definierten Windows-Treiberumgebung vor. Diese Umgebung darf intern auf unsere Kernelmechanismen abbilden, muss nach außen aber die für das Zielprofil zugesagte Semantik erfüllen.

| Bereich | Zu spezifizierender Vertrag |
| --- | --- |
| Binärladen | Unterstütztes Abbildformat, Relokationen, Imports, Datenlayout, Aufrufkonventionen und Initialisierung |
| Kernel-DDIs | Tatsächlich benötigte Routinen, Fehlerwerte, Nebenwirkungen und zulässige Aufrufkontexte |
| Objekte und Speicher | Lebensdauer, Referenzen, Gerätestrukturen, Speicherbereiche, Mapping und Ressourcenbesitz |
| Synchronisation | Locks, Wartezustände, Callbacks, Timer und zulässige Operationen je Ausführungskontext |
| Interrupts und DMA | Interruptzustellung, zurückgestellte Arbeit, Gerätezugriff, Puffer und Speicherzuordnung |
| I/O und Gerätestapel | Anfrageweitergabe, asynchroner Abschluss, Abbruch, Filter und darunterliegende Bustreiber |
| Geräte-Lebenszyklus | Erkennung, Start, Stop, Entfernen, Energiezustände und Fehler |
| Konfiguration und Pakete | Benötigte Konfigurationsdaten, Paketmetadaten, Gerätezuordnung und Integritätsprüfung |
| Frameworks | Ausgewählter KMDF-Umfang und später gegebenenfalls UMDF mit eigener Laufzeitumgebung |
| Geräteklassen | Jeweilige zusätzliche Klassen-/Port-/Miniportverträge und Abhängigkeiten |

Diese Tabelle ist eine eigene Planungszerlegung des Ziels. Die vollständige Liste ergibt sich aus dem gewählten Referenztreiber und seiner dokumentierten Laufzeitumgebung; eine Importliste allein reicht dafür nicht.

Ein im privilegierten Kernelraum laufender Fremdtreiber wird nicht allein durch eine API-Schicht isoliert. Eine Ausführung in einer getrennten Umgebung wäre eine zusätzliche Architektur mit eigenen Hardware-, Kommunikations- und Fehlerverträgen. Der Ausführungsort wird deshalb ausdrücklich entschieden und in das Sicherheitsmodell aufgenommen.

## 4. Kompatibilität nachweisen

| Stufe | Vorgeschlagenes Ergebnis | Was damit nachgewiesen ist |
| --- | --- | --- |
| T0 – Profil und Verträge | Ein x64-Treibermodell, Referenzsystem, DDI-Umfang und Testtreiber werden festgelegt | Prüfbarer Zielumfang und Auswirkungen auf die Kernelarchitektur |
| T1 – Binärgrundlage | Ausgewähltes Treiberabbild wird geladen; Initialisierung und benötigte Basisroutinen funktionieren | Nur der geprüfte Lade-/Initialisierungspfad |
| T2 – Verhaltenstest | Für Windows gebauter Testtreiber läuft unverändert mit I/O, Synchronisation, Abbruch und Fehlerfällen | Die tatsächlich getesteten DDI- und Lebenszyklusverträge |
| T3 – Reales Gerät | Ein benannter Windows-Hardwaretreiber steuert sein Gerät unter unserem OS | Funktion auf genau diesem Geräte-/Treiber-/Firmwareprofil |
| T4 – Framework und Klasse | Benannte KMDF-Version/Funktionsmenge und weitere passende Treiber funktionieren | Ausgewählter Framework- und Klassenumfang |
| T5 – Breiter Ausbau | Zusätzliche Modelle, Geräteklassen und komplexere Treiberstapel werden einzeln ergänzt | Dokumentierter Umfang der wachsenden Kompatibilitätsmatrix |

Ein selbst entwickelter Windows-Testtreiber hilft bei reproduzierbaren Vertragstests. T3 muss zusätzlich ein tatsächlich vorhandenes externes Hardwaretreiberpaket prüfen. Neu gegen unsere native API kompilierter Code zählt nicht als Nachweis unveränderter Windows-Binärkompatibilität.

## 5. Integration in den Gesamtplan

T0 gehört in die aktuelle Architekturplanung vor Festschreibung von I/O-, Speicher- und Synchronisationsverträgen. T1/T2 folgen, sobald die dafür benötigten Kernelmechanismen verfügbar sind; ein erster gezielter Nachweis soll damit vor einer breiten Treiberentwicklung liegen.

Ein späterer Zeitpunkt für umfassende Geräteabdeckung rechtfertigt keine Vertagung der Grundsatzarchitektur. Welche T-Stufe die erste Desktop-Version voraussetzt, bleibt ausdrücklich zu entscheiden.

Vorschlag: Das erste kompatible Hardwareprofil anhand überschaubarer Abhängigkeiten, verfügbarer Dokumentation und guter Testbarkeit auswählen. Moderne GPU-, WLAN-, Audio- und Storage-Pakete erhalten jeweils gesonderte Untersuchungen; ihre Kompatibilität wird nicht aus Erfolgen anderer Klassen abgeleitet.

## 6. Inhalt eines konkreten Kompatibilitätsprofils

- Treiberpaket, Hersteller, Version, Binärdateien und Hashes.
- CPU-Architektur, Treibermodell, Framework-Abhängigkeiten und Windows-Referenzversion.
- Gerät einschließlich IDs, Revision und Firmware.
- Weitere benötigte Treiber, Dienste, Bibliotheken und Konfigurationsdaten.
- Erforderliche Kernelroutinen und Datenstrukturen mit jeweils definierter Semantik.
- Getestete Operationen, Parallelität, Abbruch, Ressourcenmangel und Gerätefehler.
- Installations-/Ladeverfahren und ausdrücklich unterstützte Paketbestandteile.
- Leistung und Stabilität unter einem benannten Workload.
- Fehlende Funktionen und aktuelle Grenzen.

## 7. Nächste Architekturentscheidung

Ein erstes Profil aus T0 macht die Kernelwahl überprüfbar. Darauf aufbauend vergleichen wir einen modularen monolithischen Kern mit expliziter Windows-Treiberumgebung und eine stärker getrennte Ausführungsarchitektur. Entscheidungskriterien sind erreichbare Kompatibilität, Fehlerauswirkungen, Leistungsbedarf, Wartbarkeit und der Umfang der benötigten Mechanismen.

Erst danach werden konkrete Kernel-DDIs, Strukturdefinitionen und Zustandsautomaten als verbindliche Detail-Spec ausgearbeitet. Die aktuelle Planung behauptet weder vollständige Windows-Treiberkompatibilität noch bereits gelöste Umsetzung.
