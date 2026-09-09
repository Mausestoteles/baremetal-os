> Historischer Entwurf 0.4; nicht mehr normativ. Aktueller Stand: [Project OS](../../../README.md).

# Architekturoptionen

Stand: 09.09.2026. Diskussionsgrundlage; noch keine bestätigte Wahl von Kernelarchitektur, Sprache oder Firmwareprofil.

## 1. Ausgangspunkt

Bestätigt sind ein eigener x86-64-Desktop, zunächst in der VM und danach auf echter Hardware, eigener Bootloader, Kernel, Standardbibliotheken, Oberfläche und zentrale Anwendungen. Externe Entwicklungswerkzeuge sind erlaubt. Windows-Apps werden später gesondert bewertet.

Ergänzt: vorhandene Hardware/Firmware ist gesetzt, eigener Loader ab Beginn, Kernelcode und Bibliotheken dürfen übernommen werden, sämtliche Systemdienste und Standardbibliotheken werden selbst entwickelt. Windows-Treiberkompatibilität ist inzwischen ein ausdrückliches Architekturziel.

Die folgende Empfehlung ist eine projektspezifische Einschätzung. Sie soll anhand des bestätigten Desktop-Produktziels sowie von Zuverlässigkeit, Sicherheit, Leistung und Wartbarkeit überprüft werden.

## 2. Kernelarchitektur vergleichen

| Option | Aufbau | Nutzen für dieses Projekt | Kosten und Konsequenzen |
| --- | --- | --- | --- |
| Modularer monolithischer Kernel | Speicher, Scheduling und zunächst wesentliche Treiber/Systemfunktionen teilen den privilegierten Kernelraum; Quellcode und Schnittstellen sind modular getrennt | Direkte interne Aufrufe und ein zusammenhängender Startpfad erleichtern den schrittweisen Aufbau | Ein schwerer Fehler in privilegiertem Treibercode kann das Gesamtsystem beschädigen; Module sind keine Schutzgrenzen |
| Mikrokernel | Kleiner privilegierter Kern; wesentliche Dienste und Treiber laufen in getrennten User-Prozessen und kommunizieren über definierte Mechanismen | Stärkere Trennung von Diensten und gezieltere Fehlerbehandlung möglich | Benötigt früh IPC, Ressourcenvergabe, Dienststart und Regeln für Dienstausfälle; Wiederanlauf muss ausdrücklich entworfen werden |
| Definierte Mischform | Bestimmte Funktionen bleiben im Kernel, andere werden getrennte Dienste | Trennung kann pro Subsystem anhand seiner Anforderungen entschieden werden | Der Begriff allein legt nichts fest; jede Platzierung, Schutzgrenze und Kommunikationsart muss beschrieben werden |

Die Fehlerwirkung privilegierter Treiber ergibt sich aus ihrem gemeinsamen Schutzbereich; siehe [Microsoft zu User Mode und Kernel Mode](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode). Als konkretes Mikrokernel-Beispiel beschreiben die [seL4-FAQ](https://sel4.systems/About/FAQ.html) Treiber im User Mode und die [IPC-Dokumentation](https://docs.sel4.systems/Tutorials/ipc) Kommunikation zwischen getrennten Diensten. Diese Quellen erklären Konzepte; sie bedeuten keine Übernahme eines fremden Kernels.

**Vorläufige Empfehlung:** Einen modularen monolithischen Kernel als Ausgangskandidaten prüfen. Für den schrittweisen Aufbau eines vollständig selbst entwickelten Desktops erscheint mir das ein überschaubarer erster Ansatz. Anwendungen und Desktop laufen in geschützten User-Prozessen. Ein Mikrokernel ist die ernsthafte Alternative, wenn die Isolation und Wiederherstellung von Treibern zu den wichtigsten Projektzielen gehören.

**Zusätzliche Entscheidungsbedingung:** Seit Windows-Treiberkompatibilität gesetzt ist, darf diese Wahl erst nach dem [Kompatibilitätsentwurf](WINDOWS-TREIBERKOMPATIBILITAET.md) festgeschrieben werden. Zu prüfen ist eine eigene native Kernelarchitektur mit expliziter Windows-Treiberumgebung. Sie muss für ausgewählte Treiber deren Datenlayouts, I/O-, Speicher-, Interrupt- und Synchronisationsverträge erfüllen. Eine allgemeine Modulgrenze allein leistet das nicht.

Eine spätere Verlagerung von Kernelfunktionen in User-Prozesse wäre ein eigener Umbau. Wir versprechen keine automatische Migration durch bloße Modularität.

## 3. Sprache separat entscheiden

Die Kernelarchitektur und die Implementierungssprache sind unterschiedliche Entscheidungen. Der Sprachvergleich richtet sich nach den technischen Produktanforderungen; Angaben zur persönlichen Erfahrung sind dafür keine Voraussetzung.

Zu bewerten sind:

- C, C++ oder Rust als Hauptsprache; Assembleranteile gezielt für notwendige Hardwareübergänge.
- Unterstützung einer Umgebung ohne vorhandenes Host-Betriebssystem und ohne dessen Standardbibliothek.
- Aufwand für eigene Laufzeit, Speicherverwaltung und Fehlerbehandlung.
- Kontrolle über Datenlayout, Aufrufkonventionen und Compiler-Hilfsroutinen.
- Prüfbare Regeln für Speicherzugriffe, Parallelität und hardwarenahe Operationen.
- Langfristige Wartbarkeit und Eignung für Treiber, Systemdienste und SDK.

Für Rust existiert beispielsweise ein dokumentiertes [x86_64-unknown-none-Ziel](https://doc.rust-lang.org/rustc/platform-support/x86_64-unknown-none.html) ohne Unterstützung von `std`. Damit wäre die Frage, welche Sprachkernbibliotheken und Compilerbestandteile unter unsere Eigenentwicklungsgrenze fallen, ausdrücklich zu klären. Ein vorhandenes Compilerziel ersetzt weder eigenen Kernel noch eigene Betriebssystemdienste.

## 4. Vorschlag für die Startkette

1. Vorhandene UEFI-Firmware startet unseren selbst entwickelten Bootloader.
2. Der Loader lädt und prüft unser Kernelabbild und sammelt vereinbarte Plattforminformationen.
3. Er übergibt Speicherkarte, Anzeigeinformationen und weitere festgelegte Daten über einen versionierten Bootvertrag.
4. Der Übergang von Firmware-Bootdiensten zum Kernel wird im Bootvertrag eindeutig beschrieben.
5. Der Kernel richtet seine eigenen Mechanismen ein und startet später den ersten User-Prozess.
6. Die eigenen Systemdienste starten Sitzung, Desktop und Anwendungen.

Die Nutzung vorhandener Firmware ist bestätigt; UEFI ist die vorgeschlagene konkrete Schnittstelle. Der Umfang benötigter UEFI- und ACPI-Funktionen wird anhand der [offiziellen Spezifikationen](https://uefi.org/specifications) festgelegt.

## 5. Vorschlag für das erste Testprofil

QEMU ist ein Kandidat: Seine Systememulation stellt CPU, Speicher und emulierte Geräte für ein Gast-Betriebssystem bereit. Quelle: [QEMU-Systememulation](https://www.qemu.org/docs/master/system/introduction.html).

Das verbindliche Profil muss konkrete Versionen, CPU-Funktionen, RAM, Firmware, Geräte, Bootmedium, Diagnosekanal und Timeouts festhalten. Ein zweites Profil beschreibt den echten Testrechner. Ein VM-Test gilt nur für die dort modellierte Hardware.

Für P1 planen wir nur die Hardware ein, die zum Laden und Diagnostizieren nötig ist. Tastatur, persistentes Systemlaufwerk, Netzwerk und beschleunigte Grafik erhalten anschließend jeweils eigene Treiber- und Testentscheidungen.

## 6. Vorschlag für den ersten nativen Desktop

Bestätigtes erstes Produktpaket: allgemeiner Desktop mit Fenstern, Dateimanager, Einstellungen, Terminal und Texteditor. Folgende Ausgestaltung ist vorgeschlagen; maßgeblich für die Diskussion ist der ausführlichere [Produktentwurf](PRODUKT-DESKTOP-01.md):

- Fensterverwaltung mit Maus und Tastatur, Fokus und grundlegender Zwischenablage.
- Terminal mit eigener Shell.
- Dateimanager mit Verzeichnissen und grundlegenden Dateioperationen.
- Texteditor zum Öffnen, Ändern und Speichern einfacher Textdateien.
- Einstellungen für zunächst unterstützte Anzeige-, Eingabe- und Sitzungseigenschaften.
- Kleine Systemübersicht für Prozesse und Speicherbelegung.

Ein möglicher durchgehender Abnahmetest: System starten, zwei Anwendungen öffnen, Textdatei erstellen, im Dateimanager finden, bearbeiten, speichern, neu starten und wieder öffnen. Zusätzlich wird ein App-Absturz ausgelöst und die verbleibende Sitzung weiter bedient.

Browser, Multimedia, Grafikbeschleunigung und breitere Hardwareunterstützung brauchen jeweils eine eigene Umfangsentscheidung. Vorgeschlagen ist, sie außerhalb der ersten Desktop-Abnahme zu behandeln; die langfristige Zielvision bleibt bestehen.
