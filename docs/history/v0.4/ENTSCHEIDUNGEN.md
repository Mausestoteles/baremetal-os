> Historischer Entwurf 0.4; nicht mehr normativ. Aktueller Stand: [Project OS](../../../README.md).

# Entscheidungen und offene Fragen

Stand: 09.09.2026. Empfehlungen in diesem Dokument sind nicht als Zustimmung des Projektinhabers zu verstehen.

## Bestätigte Vorgaben

| ID | Vorgabe | Herkunft |
| --- | --- | --- |
| E-001 | Langfristiges Projekt eines eigenen Betriebssystems | Erstauftrag vom 09.09.2026 |
| E-002 | Eigener Kernel und direkter Betrieb auf echter Hardware | Erstauftrag vom 09.09.2026 |
| E-003 | Langfristig eine vollständige Plattform mit Anspruch auf Windows-Niveau | Erstauftrag vom 09.09.2026; genaue Vergleichskriterien offen |
| E-004 | Zunächst ausschließlich gemeinsam planen und spezifizieren | Erstauftrag vom 09.09.2026 |
| E-005 | Erste Plattform: x86-64-Desktop, zuerst VM, danach ausgewählte echte Hardware | Antwort auf O-001 vom 09.09.2026 |
| E-006 | Auch Bootloader, Standardbibliotheken, Grafikoberfläche und zentrale Anwendungen selbst entwickeln; externe Entwicklungswerkzeuge erlaubt | Antwort auf O-002 vom 09.09.2026 |
| E-007 | Windows-Niveau zunächst als vergleichbaren Komfort und Funktionsumfang mit nativen Apps verfolgen; Windows-Apps später gesondert bewerten | Antwort auf O-003 vom 09.09.2026 |
| E-008 | Erstes nutzbares Produkt: allgemeiner Desktop mit Fenstern, Dateimanager, Einstellungen, Terminal und Texteditor | Antwort auf O-004 vom 09.09.2026 |
| E-009 | Professioneller Produktanspruch mit ernstzunehmender Konkurrenz zu Windows als langfristigem Ziel | Bekräftigung vom 09.09.2026 |
| E-010 | Vorhandene Hardware und Firmware als Plattform voraussetzen | Antwort zur Umfangstabelle vom 09.09.2026, Punkt 1 |
| E-011 | Bootloader von Beginn an selbst entwickeln | Antwort zur Umfangstabelle vom 09.09.2026, Punkt 2; präzisiert E-006 |
| E-012 | Im eigenen Kernel dürfen Bibliotheken und Code übernommen werden | Antwort zur Umfangstabelle vom 09.09.2026, Punkt 3 |
| E-013 | Sämtliche Systemdienste und Standardbibliotheken selbst entwickeln | Antwort zur Umfangstabelle vom 09.09.2026, Punkt 4 |
| E-014 | Kompatibilität zu Windows-Treibern ist ein Projektziel; weitere Treiberwelten noch unbenannt | Antwort zur Umfangstabelle vom 09.09.2026, Punkt 5 |

## Runde 1 – Ziel und Eigenentwicklungsumfang

| ID | Frage | Bestätigte Entscheidung | Konsequenz |
| --- | --- | --- | --- |
| O-001 | Welche Plattform zuerst? | x86-64-Desktop, VM als Entwicklungsumgebung, anschließend ausgewählte echte Hardware | Macht den ersten Geräte- und Testumfang eingrenzbar; endgültige Hardware bleibt zu benennen |
| O-002 | Was muss selbst entwickelt werden? | Eigener Kernel mit erlaubter Code-/Bibliotheksübernahme; eigener Bootloader ab Beginn; sämtliche Systemdienste und Standardbibliotheken sowie eigene Oberfläche und zentrale Apps; externe Entwicklungswerkzeuge erlaubt | Loader und minimale eigene Laufzeit sind früh einzuplanen; eigene Systemdienste bleiben trotz erlaubter Kernel-Abhängigkeiten gesetzt |
| O-003 | Welche Windows-Kompatibilität wird verlangt? | Zuerst Komfort und Funktionen einer eigenen nativen Plattform; Windows-Apps später gesondert bewerten | Die ersten Produktziele benötigen eigene Apps; Windows-Binärkompatibilität ist kein zugesagtes Abnahmekriterium |

Alle drei Fragen wurden am 09.09.2026 beantwortet und sind für diesen Planungsstand entschieden.

## Runde 2 – Produkt und Ressourcen

| ID | Zu klären | Warum jetzt relevant |
| --- | --- | --- |
| O-004 – entschieden | Allgemeiner Desktop mit Fenstern, Dateimanager, Einstellungen, Terminal und Texteditor | Bestätigt als E-008; Details siehe Produktentwurf |
| O-005 – zurückgestellt | Ressourcen nicht beziffert; Projektinhaber betont professionellen Produktanspruch | Keine weitere Abfrage dazu für die fachliche Planung; keine Termin- oder Kapazitätsannahme |
| O-006 | Vorhandene Testhardware und Budget | Grundlage der realen Hardwarematrix |
| O-007 | Öffentliches oder privates Projekt; gewünschte Lizenz und Beitragsmodell | Beeinflusst später zulässige Abhängigkeiten und Zusammenarbeit |
| O-008 | Vorläufiger Projektname | Hilft bei Artefakten und Namensräumen; blockiert die technische Planung nicht |

## Runde 3 – Architektur

| ID | Entscheidung | Zu vergleichende Optionen oder Kriterien |
| --- | --- | --- |
| O-009 | Kernelarchitektur | Modularer monolithischer Kernel, Mikrokernel oder ausdrücklich definierte Mischform; Windows-Treiberverträge, Schutzgrenzen, IPC, Treiberfehler, Aufwand und Leistung vergleichen |
| O-010 | Implementierungssprachen | Rust, C, C++ und notwendiger Assembleranteil; Teamkenntnisse, Speicherfehler, Toolchain, ABI und Freestanding-Unterstützung bewerten |
| O-011 | Bootweg | Firmwareprofil, Aufbau des bestätigten eigenen Bootloaders, Kerneldateiformat, Übergabeinformationen |
| O-012 | Prozess- und Sicherheitsmodell | Identitäten, Rechte/Capabilities, Objektbesitz, IPC und Verhalten bei App- und Treiberfehlern |
| O-013 | Native Anwendungsschnittstelle | API, Binärschnittstelle, Systemaufrufe, Fehlerdarstellung, Versionierung, Bibliotheken und eventueller POSIX-Umfang |
| O-014 | Treibermodell | Eigene Treiberschnittstelle und Windows-Kompatibilitätsumgebung; Ausführungsort, Geräteerkennung, Ressourcen, DMA, Interrupts und Lebenszyklus |
| O-015 | Speicher und Parallelität | Virtueller Adressraum, Allokatoren, Synchronisation, Scheduler, Mehrkernstart und Reihenfolge der Einführung |
| O-016 | Dateien und Speicherung | VFS, bestehendes oder eigenes Dateisystem, Konsistenz, Unicode, Metadaten und Austauschformate |
| O-017 | Grafik und Desktop | Frühe Bildausgabe, Software-Rendering, spätere Beschleunigung, Eingabe, Fensterprotokoll und Toolkit |
| O-018 | Netzwerk und Kryptografie | Eigene und externe Implementierungen, Protokollumfang, Entropie, Schlüsselverwaltung und Aktualisierbarkeit |
| O-019 | Systemkonfiguration und Wartung | Dienste, Einstellungen, Pakete, Installation, Updateformat, Bootvertrauen und Wiederherstellung |
| O-020 | Entwicklungsumgebung | Host-Plattform, Buildsystem, Cross-Toolchain, Emulator, Debugger und Testautomatisierung |
| O-021 | Grenze externer Softwarebestandteile | Sprachkernbibliotheken, Compiler-Hilfsroutinen, Schriften, Gerätefirmware und spezielle Bibliotheken einzeln einordnen; eigene Standardbibliotheken bleiben gesetzt |
| O-022 | Windows-Treiberprofil | Arbeitsannahme: unveränderte x64-Treiberbinärdateien; konkrete Windows-/WDM-/WDF-Versionen, erste Geräteklasse, Referenztreiber und Paketinstallation noch festzulegen |
| O-023 | Weitere Treiberkompatibilität | „etc.“ benennt noch keine weiteren Treibermodelle; Linux-, macOS- oder andere Treiberunterstützung ist bislang nicht zugesagt |

Diese Themen werden in Abhängigkeitsreihenfolge bearbeitet. Die Tabelle ist keine Aufforderung, alle Entscheidungen ohne weitere Gespräche zu treffen.

## Windows-Kompatibilität präzisieren

Die folgenden Ziele sind getrennt zu entscheiden:

1. Ähnliche Bedienbarkeit und ähnliche Funktionen.
2. Austausch von Dateien und Netzwerkprotokollen.
3. Portierung von Quellcode auf die eigene Plattform.
4. Ausführung ausgewählter Windows-Programme ohne Neukompilierung.
5. Unterstützung bestimmter Spiele, Grafik-APIs und ihrer zusätzlichen Dienste.
6. Nutzung vorhandener Windows-Treiber: inzwischen ausdrücklich als Ziel bestätigt; genauer Umfang und technische Realisierung noch offen.

Ein Ja zu einem Punkt ist kein Ja zu den anderen. Für Binärkompatibilität brauchen wir konkrete Programme, Versionen, Architekturen und überprüfbare Arbeitsabläufe. Die Windows-Treiberarchitektur umfasst eigene Kernel-Schnittstellen und Subsysteme; ihre Übernahme ist deshalb eine separate Designfrage. Referenz: [Microsofts Architekturübersicht für Kernel-Treiber](https://learn.microsoft.com/en-us/windows-hardware/drivers/kernel/).

Die aktuelle Zielsetzung kombiniert native Anwendungen mit Windows-Treiberkompatibilität. Soweit ein Zieltreiber zusätzliche Windows-Dienste oder User-Mode-Komponenten benötigt, wird dieser Umfang ausdrücklich im Treiberprofil erfasst. Daraus folgt keine pauschale Zusage allgemeiner Windows-App-Kompatibilität.

## Vorlage für eine Architekturentscheidung

- **ID und Titel:** eindeutiger Bezeichner.
- **Status:** vorgeschlagen, angenommen, verworfen oder ersetzt.
- **Datum und Beteiligte:** wer die Entscheidung getroffen hat.
- **Problem:** welche Anforderung eine Entscheidung nötig macht.
- **Optionen:** ernsthaft betrachtete Alternativen.
- **Entscheidung und Begründung:** gewählte Option und ausschlaggebende Kriterien.
- **Folgen:** Vorteile, Kosten, Einschränkungen und betroffene Schnittstellen.
- **Nachweis:** Quellen, Untersuchung oder späterer Prototyp.
- **Erneut prüfen bei:** konkrete Änderung der Voraussetzungen.

## Änderungsverlauf

- 09.09.2026: Planungsstand 0.1 aus dem Erstauftrag angelegt; technische Grundsatzentscheidungen noch offen.
- 09.09.2026: Planungsstand 0.2 übernimmt Antworten auf O-001 bis O-003. Erste Produktversion und Ressourcenrahmen sind als nächste Fragen gestellt; Kernelarchitektur und Sprache bleiben offen.
- 09.09.2026: Planungsstand 0.3 übernimmt den ersten Desktop-Umfang und den ausdrücklich professionellen Produktanspruch. Ressourcenabfrage zurückgestellt; Produktabnahme und Qualitätsplanung ergänzt.
- 09.09.2026: Planungsstand 0.4 übernimmt die beantwortete Umfangstabelle: vorhandene Firmware/Hardware, eigener Loader ab Anfang, Wiederverwendung von Kernelcode erlaubt, sämtliche Systemdienste und Standardbibliotheken selbst, Windows-Treiberkompatibilität als Architekturziel. Externe Entwicklungswerkzeuge erneut bestätigt.
