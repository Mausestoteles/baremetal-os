# Entscheidungen – Baseline 1.0

Stand: 09.09.2026. Der Projektinhaber hat ausdrücklich beauftragt, die noch offenen Punkte festzulegen und das Ergebnis in ein neues GitHub-Repository zu laden. Die folgenden technischen Entscheidungen wurden auf dieser Grundlage festgelegt. Sie sind keine Behauptung bereits implementierter Funktionen.

## Unveränderte Vorgaben des Projektinhabers

Eigener Kernel und Bare-Metal-Betrieb; professionelle langfristige Windows-Alternative; zuerst x86-64-Desktop in einer VM; vorhandene Hardware/Firmware; eigener Loader ab Anfang; Code-/Bibliotheksübernahme im Kernel erlaubt; sämtliche Systemdienste und Standardbibliotheken selbst; eigene Oberfläche und zentrale Apps; externe Entwicklungswerkzeuge erlaubt; Windows-Treiberkompatibilität; Windows-Anwendungen separat später bewerten.

## Auflösung der bisherigen Fragen

| Frühere ID | Festlegung |
| --- | --- |
| O-001 Plattform | x86-64 mit vierstufigen Seitentabellen; UEFI ab 2.7; ACPI ab 6.0; VM-BOOT, VM-SMP und VM-DESKTOP gemäß Boot-Spec |
| O-002 Eigenentwicklung | Eigener Loader, Kernel, Systemdienste, libc, libos, Toolkit und Basisapps; Kernelabhängigkeiten mit Herkunftsnachweis zulässig |
| O-003 Windows-Apps | Kein Releasekriterium für 0.1–0.3; spätere getrennte Kompatibilitätsetappe |
| O-004 Erstes Produkt | Eigener Desktop gemäß PROD-DESKTOP-01 |
| O-005 Ressourcen | Meilensteinplanung ohne Kalendertermine; keine unterstellte Teamgröße oder Verfügbarkeit |
| O-006 Hardware/Budget | Kein Kauf in dieser Phase. HW-01 ist das verbindliche Mindestprofil; ein vorhandener konformer Rechner wird vor dem ersten Hardwaretest mit IDs registriert |
| O-007 Sichtbarkeit/Lizenz | Neues privates Repository im authentifizierten persönlichen GitHub-Konto. Keine öffentliche Lizenzfreigabe für den eigenen Code in dieser Baseline; Fremdlizenzen bleiben erhalten |
| O-008 Name | Arbeitsname Project OS; Repository baremetal-os; interne Präfixe os_, OS_ und /System |
| O-009 Kerneltyp | Modular monolithisch; Speicher, Scheduling, VFS, Block-/Netzwerkmechanismen und zunächst Treiber im Kernel; Verwaltung und Desktop als User-Dienste |
| O-010 Sprachen | C17 und gezielter x86-64-Assembler für alle Zielkomponenten der ersten Produktlinie; Python nur für Hostwerkzeuge |
| O-011 Boot | Eigener UEFI-PE32+-Loader unter EFI/BOOT/BOOTX64.EFI; eigener ELF64-ET_EXEC-Kernel; boot_info_v1 |
| O-012 Sicherheit | User-/Kernel-Trennung, explizite Handle-Rechte, lokale Benutzer/Dateirechte, überprüfte Kopien über Schutzgrenzen; privilegierte Treiber sind Teil der Vertrauensbasis |
| O-013 Native ABI | ELF64, SysV AMD64 für native Funktionen; eigene versionierte SYSCALL-ABI; statische Programme bis 0.2 |
| O-014 Treiber | Native C-Treiberschnittstelle und getrennte Windows-x64-DDI-Umgebung im Kernel; Treiberladeberechtigung nur für Systemverwaltung |
| O-015 Parallelität | 4-KiB-Seiten, Buddy-Allokator und Slab-Klassen; präemptiver Prioritätsscheduler; IRQ/DPC/Thread-Kontexte; SMP vor 0.1-Desktop-Abnahme |
| O-016 Dateien | Eigene ext2-rev1-Implementierung für 0.1; ext4 mit JBD2 vor 0.2-Vorschau; GPT/FAT32 für Boot/Austausch; VFS und UTF-8-Pfade |
| O-017 Grafik | GOP-Framebuffer, CPU-Rendering und eigener User-Compositor; eigener Toolkit-/Eingabevertrag; ein Bildschirm bis 0.2 |
| O-018 Netzwerk | Eigener Ethernet-/ARP-/IPv4-/ICMP-/UDP-/TCP-Stack, DHCPv4 und DNS; TLS 1.3 vor öffentlichem Netzwerkprodukt; IPv6 in 0.3 |
| O-019 Wartung | Eigener init/service manager, Textkonfiguration, Paketmanager und Installer; A/B-Systemstände und getrennte Datenpartition ab 0.2 |
| O-020 Werkzeuge | Windows 11 x64 als primärer Host; LLVM 22.1.8, CMake 3.31.8, Ninja 1.12.1, Python 3.11; QEMU 11.1.1; siehe Build-Spec |
| O-021 Grenzen | Sprach-/Firmwaredefinitionen dürfen aus dokumentierten Standards abgeleitet werden; kein fremdes libc/GUI-Framework als eigene Komponente ausgeben. Compiler-Builtins werden inventarisiert; eigene Speicherprimitiven |
| O-022 Windows-Treiber | Unveränderte x64-Binärkompatibilität für WDM-TEST-1; Referenz Windows 11 24H2 / WDK 10.0.26100; danach KMDF-ECHO-1 und NETKVM-LAB-1 |
| O-023 Andere Treiber | Keine Linux-/macOS-Treiberbinärkompatibilität in 0.1–0.3; offene Treiberquellen können in die native Kernelarchitektur portiert werden |

Der reale Hardwarebesitz, Geheimschlüssel, Testergebnisse und künftige Verfügbarkeit sind Tatsachen, die erst bei ihrer Entstehung erfasst werden. Sie werden nicht als erfundene Architekturentscheidungen eingetragen. Ihre Erfassung ist jeweils einem Backlog-Auftrag zugeordnet.

## ADR-001 – Modularer monolithischer Kernel

Gewählt wegen direkter Abbildung der zuerst benötigten Windows-Kernelverträge, klarer interner Ressourcenkontrolle und nachvollziehbarer Entwicklungsfolge. Ein Mikrokernel würde zusätzlich eine kompatible Treiberausführungsumgebung über Prozessgrenzen benötigen. Die gewählte Modulstruktur isoliert privilegierte Treiberfehler nicht; diese Kosten sind Teil des Sicherheitsmodells.

## ADR-002 – C17 und Assembler

Gewählt für einen einheitlichen Freestanding-Unterbau, direkte Kontrolle von ABI/Datenlayout und Anschlussfähigkeit an Windows-DDIs und erlaubte Kernelcodeübernahme. Rust und C++ werden für die erste Produktlinie nicht eingesetzt. Das vermeidet zusätzliche Standardbibliotheks-/Laufzeitentscheidungen, verzichtet jedoch auf Rusts sprachseitige Speicherschutzmechanismen. Deshalb sind Review von Besitzregeln, Host-Sanitizer-Tests, Fuzzing und negative Kerneltests verbindlich.

## ADR-003 – Eigenes natives API neben Windows-DDIs

Native Apps verwenden ausschließlich libos/libc und unsere Syscall-ABI. Windows-Treiber verwenden eine separate Kompatibilitätsoberfläche. Interne Kernelobjekte werden nicht direkt als öffentliches Windows-Layout ausgegeben. Wo Treiber Felder direkt erwarten, besitzt die Umgebung eigene ABI-konforme Fassaden mit expliziten Synchronisations- und Lebensdauerregeln.

## ADR-004 – Bestehende Dateiformate, eigene Implementierung

Wir entwickeln Dateisystemcode selbst und verwenden dokumentierte Formate. ext2 ermöglicht einen begrenzten ersten Schreibpfad. Vor einer verteilbaren Vorschau folgt ext4/JBD2 mit geprüfter Wiederherstellung. Die frühe ext2-Version wird ausdrücklich nicht als stromausfallsicheres Produkt freigegeben.

## ADR-005 – Getrennte Reife- und Funktionsziele

0.1 ist ein internes Desktop-System; 0.2 eine installierbare Entwicklervorschau; 0.3 erweitert Geräte, Netzwerk und Alltagsszenarien. „Windows-Niveau“ bleibt das langfristige Dachziel und wird durch benannte Szenarien erweitert, nicht durch eine behauptete pauschale Gleichwertigkeit.

## Änderungen

Baselineänderungen erhalten Datum, betroffene Vertragsversion, Begründung und Abnahmeänderung. Redaktionelle Korrekturen dürfen ohne ABI-Wechsel erfolgen. Die Entwürfe 0.1–0.4 sind unter history/v0.4 archiviert.
