# Project OS

Eigenständiges Desktop-Betriebssystem für x86-64 mit eigenem Bootloader, Kernel, Systemdiensten, Standardbibliotheken und nativen Anwendungen. Langfristiges Produktziel: eine ernstzunehmende Alternative zu Windows einschließlich gezielt nachgewiesener Windows-Treiberkompatibilität.

**Stand: Spezifikationsbaseline 1.0 vom 09.09.2026. Es gibt noch keinen implementierten oder bootfähigen Kernel.**

Der Projektinhaber hat die Festlegung der offenen Fragen delegiert. Die aktuelle Baseline enthält verbindliche Entwicklungsentscheidungen und eine umsetzungsreife erste Boot-Etappe. Spätere Systeme besitzen festgelegte Architektur, Lieferumfang und Abnahme; ihre vollständigen Implementierungen und Kompatibilitätsnachweise entstehen in den zugeordneten Etappen.

## Direkt anfangen

Lies [START_HERE.md](START_HERE.md). Der erste Implementierungsauftrag ist B-001 im [Backlog](docs/BACKLOG.md). Build-Befehle in den Specs beschreiben das herzustellende Werkzeugverhalten; sie sind in diesem reinen Planungsstand noch nicht ausführbar.

## Festgelegte Basis

| Bereich | Entscheidung |
| --- | --- |
| Plattform | x86-64, UEFI, ACPI; zuerst QEMU, danach Hardwareprofil HW-01 |
| Kernel | Modular monolithisch; geschützte User-Prozesse; explizites Windows-Treibersubsystem |
| Sprachen | C17 und x86-64-Assembler; eigene Laufzeit; kein C++/Rust im ersten Produkt |
| Werkzeuge | LLVM/Clang/LLD 22.1.8, CMake 3.31.8, Ninja, Python; Windows als erster Entwicklungs-Host |
| Boot | Eigener UEFI-Loader BOOTX64.EFI lädt eigenen ELF64-Kernel |
| Native Apps | Eigenes C-API, versionierte Syscall-ABI, ELF64, zunächst statisch gelinkt |
| Speicherung | Eigener ext2-Treiber für frühe Versionen; eigener ext4/JBD2-Pfad vor verteilbarer Vorschau |
| Desktop | Eigener Software-Compositor, Toolkit, Dateimanager, Einstellungen, Terminal und Texteditor |
| Windows-Treiber | Unveränderte x64-Binärdateien für explizite Profile; zuerst WDM-Vertragstests, danach KMDF und Gerätekategorien |
| Projekt | Privates Repository; Arbeitsname Project OS; keine Käufe, kein eigener Compiler im aktuellen Umfang |

## Dokumente

| Einstieg | Inhalt |
| --- | --- |
| [Entscheidungen](docs/ENTSCHEIDUNGEN.md) | Auflösung sämtlicher bisheriger Entscheidungsfragen |
| [Projektplan](docs/PROJEKTPLAN.md) | Lieferstufen und Abhängigkeiten |
| [Architektur](docs/ARCHITEKTUROPTIONEN.md) | Gewählte Architektur und verworfene Alternativen |
| [Spec-Index](docs/SPEZIFIKATIONEN.md) | Zuständigkeit der 19 Spezifikationsbereiche |
| [Boot und Plattform](docs/specs/BOOT-UND-PLATTFORM.md) | VM-Profil, ELF-Vertrag, Speicherübergabe und P1-Tests |
| [Kernel und ABI](docs/specs/KERNEL-UND-ABI.md) | Schutzbereiche, Speicher, Scheduler, Handles, IPC und Systemaufrufe |
| [System und Desktop](docs/specs/SYSTEM-UND-DESKTOP.md) | Dateien, Dienste, Apps, Grafik, Netzwerk und Wartung |
| [Build und Tests](docs/specs/BUILD-UND-TESTS.md) | Entwicklungsumgebung, Artefakte, Ziele und Prüfkriterien |
| [Windows-Treiber](docs/WINDOWS-TREIBERKOMPATIBILITAET.md) | Zielprofile, Verträge und messbare Kompatibilitätsstufen |
| [Desktop-Abnahme](docs/PRODUKT-DESKTOP-01.md) | Erstes durchgehendes Produktszenario |
| [Qualität und Releases](docs/QUALITAET-UND-RELEASES.md) | Freigabekriterien und Wartbarkeit |
| [Backlog](docs/BACKLOG.md) | Geordnete Implementierungsaufträge |
| [Quellen](docs/QUELLEN.md) | Technische Primärquellen |

Die alten Entwürfe liegen ausschließlich unter [docs/history/v0.4](docs/history/v0.4/README.md) und sind nicht mehr normativ. Änderungen an der Baseline erfordern einen dokumentierten Entscheidungsgrund und passende Änderungen an Specs und Abnahme.

Alle Ergebnisse dieses Commits sind Planung, Dokumentation und Repository-Konventionen. Es wurden keine OS-Builds, Bootversuche oder Treiberkompatibilitätstests behauptet oder ausgeführt.
