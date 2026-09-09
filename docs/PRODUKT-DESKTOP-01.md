# PROD-DESKTOP-01 – Desktop 0.1

Baseline 1.0. Status: festgelegte Produktabnahme, noch nicht implementiert.

## Umfang

Eigene grafische Sitzung, Fensterverwaltung, Dateimanager, Einstellungen, Terminal/Shell, Texteditor und Systemübersicht. Ein Bildschirm, Tastatur/Maus, persistente lokale Dateien, eigene libui/libos/libc. Native ELF64-Apps; WDM-TEST-1 als erster Treiberkompatibilitätsnachweis.

Browser, Spiele, beschleunigte 3D-Grafik, WLAN/Bluetooth, Audio, Druck, Mehrmonitorbetrieb und Windows-Apps gehören nicht zu dieser ersten Abnahme. Ihr langfristiger Ausbau ist im Projektplan eingeordnet.

## Referenzaufgabe

1. System auf VM-DESKTOP und später HW-01 über den eigenen Loader starten.
2. Dateimanager, Texteditor und Terminal öffnen und ihre Fenster bedienen.
3. Verzeichnis sowie UTF-8-Datei anlegen; Text einschließlich Umlauten bearbeiten.
4. Zwischen Anwendungen wechseln und Text kopieren/einfügen.
5. Datei speichern und im Terminal unverändert lesen.
6. Tastaturlayout und eine Sitzungseinstellung ändern.
7. Geordnet neu starten; Datei und Einstellung wiederfinden.
8. Eine App absichtlich abstürzen lassen und die übrige Sitzung weiter nutzen.
9. Eine Endlosschleife in einer App starten, andere App bedienen und erstere kontrolliert beenden.

## Funktionsabnahme

| Bereich | Nachweis |
| --- | --- |
| Fenster | Drei gleichzeitig nutzbare Fenster, Verschieben/Größe/Fokus/Minimieren/Maximieren/Schließen |
| Dateien | Erstellen, kopieren, verschieben, umbenennen, löschen; Konflikte, fehlende Rechte und volles Medium |
| Editor | 16-MiB-UTF-8-Datei, Suche, Undo/Redo, Speichern unter, Schutz ungespeicherter Änderungen |
| Terminal | Eigene Befehle und externes Programm; Returncode und Dateiumleitung |
| Einstellungen | Geprüfte Werte, dauerhafte Speicherung, Fehler bei unzulässigen Daten |
| Isolation | Fremder Speicherzugriff beendet den verursachenden Prozess, nicht die Sitzung |
| SDK | Separater Quellbaum erzeugt ein ohne Kerneländerung startbares Programm |
| Treiber | Unveränderte WDM-Testbinärdatei besteht das zugesagte Profil |
| SMP | Vier CPU-Kerne führen Last ohne Datenrennen in den geprüften Kernelverträgen aus |

## Fehlerverhalten

Voller Datenträger lässt den vorher gespeicherten Editorinhalt erhalten; ungespeicherter Text bleibt im Prozess. Ungültige Pfade beschädigen keine andere Datei. Queue-/Speichermangel liefert kontrollierte Fehler. App-Start/-Ende-Zyklen halten die Ressourcenbudgets ein. Ein ext2-Image nach ungeordnetem Abbruch wird nicht als konsistent vorausgesetzt; diese Version ist ein internes Entwicklungssystem.

## Budgets

Auf VM-DESKTOP: Kernel + Dienste ohne Apps höchstens 512 MiB resident; Gesamtbetrieb des Referenzszenarios höchstens 1 GiB, ausgenommen explizit begrenzter Dateicache. Eingabe-zu-Present p95 ≤50 ms unter dem definierten Desktop-Workload; auf TCG nur als Regression auf demselben Host vergleichen. Auf HW-01 ist dieses Latenzziel ein Releasekriterium. Testprotokoll enthält Hardware, Auflösung, Hostlast, Quellstand und Messverfahren.

Diese Zahlen sind Zielbudgets. Ihre Einhaltung wird erst durch die Implementierung gemessen; sie sind keine aktuellen Leistungsangaben.
