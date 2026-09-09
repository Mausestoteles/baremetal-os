# Nach dem Klonen beginnen

## Was hier fertig ist

Die technischen Grundsatzentscheidungen und die erste Implementierungsetappe sind spezifiziert. Der Quellcode des Betriebssystems wird im nächsten Arbeitsschritt geschrieben. Dieses Repository ist kein fertiges Betriebssystem und besitzt noch kein Buildsystem.

## Lesereihenfolge

1. [Entscheidungen](docs/ENTSCHEIDUNGEN.md): Umfang und Architektur.
2. [Boot und Plattform](docs/specs/BOOT-UND-PLATTFORM.md): verbindlicher erster Zielzustand.
3. [Build und Tests](docs/specs/BUILD-UND-TESTS.md): Werkzeugversionen und herzustellende Befehle.
4. [Backlog](docs/BACKLOG.md): B-001 beginnen, danach B-002 bis B-008.
5. [Kernel und ABI](docs/specs/KERNEL-UND-ABI.md): Verträge, die beim ersten Code nicht verbaut werden dürfen.

## Erster Arbeitsauftrag

**B-001: Reproduzierbares Freestanding-Buildgerüst erstellen.**

Lege CMake-Konfiguration, Cross-Toolchain-Dateien und die in BUILD-UND-TESTS beschriebenen Verzeichnisse an. Erzeuge getrennte Targets für UEFI/PE32+ und Kernel/ELF64. Implementiere zunächst nur die zum Nachweis der Targets nötigen Eintrittsstubs. Prüfe Formate, Architektur, Einstiegspunkte und fehlende Host-Laufzeitabhängigkeiten.

Danach B-002 umsetzen: eigene Boot- und Firmware-Datentypen mit Größen-/Offsetprüfungen. Erst anschließend folgen Loader und Kernelstart. Die einzelnen Backlog-Aufträge enthalten Abhängigkeiten, erwartete Dateien und Abnahme.

## Entwicklungsregeln

- Hauptsprache C17; Compilerwarnungen werden für eigenen Code als Fehler behandelt.
- Toolchain und Firmware bleiben außerhalb des Quellbaums oder unter ignoriertem .tools/.
- Keine automatisch erfundenen Erfolgswerte, keine Erfolgsmeldungen für nicht implementierte Syscalls oder DDIs.
- Keine Host-Header/-Bibliotheken im Zielcode.
- Änderungen an boot_info_v1, Syscalls oder Treiberverträgen zuerst in der Spec dokumentieren.
- Neue Arbeitsbranches erhalten das Präfix codex/.
- Kein Schreiben auf echte Datenträger durch Standard-Testbefehle. Hardwaretests verwenden ausschließlich ein ausdrücklich ausgewähltes Testmedium.

## Vorlage für den nächsten Implementierungsauftrag

> Implementiere B-001 bis B-008 aus docs/BACKLOG.md gemäß Baseline 1.0. Beginne mit dem Buildgerüst und liefere anschließend den eigenen UEFI-Bootloader und minimalen Kernelstart in QEMU. Halte BOOT-UND-PLATTFORM und BUILD-UND-TESTS ein. Führe die dortigen positiven und negativen Tests aus. Melde tatsächliche Ergebnisse und konkrete verbleibende Fehler. Implementiere noch keine späteren Produktetappen.

Diese Vorlage kann nach dem Klonen direkt als Auftrag verwendet werden. Sie ist hier Dokumentation und löst selbst keine Implementierung aus.
