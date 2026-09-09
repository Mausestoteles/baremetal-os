# Implementierungsbacklog – Baseline 1.0

Status aller Einträge: noch nicht begonnen. Reihenfolge und Abnahmen sind festgelegt. B-001 ist der nächste Arbeitsauftrag nach dem Klonen.

## P1: direkt implementierbare Aufgaben

### B-001 – Werkzeugmanifest und getrennte Buildtargets

Abhängigkeiten: keine. Lies BUILD-UND-TESTS und BOOT-UND-PLATTFORM.

Erzeuge CMakeLists.txt, CMakePresets.json, cmake/toolchains/uefi-x64.cmake und kernel-x64.cmake sowie minimale Entry-Stubs. Konfiguriere LLVM 22.1.8 und zwei echte Freestanding-Targets ohne Hostbibliothek. Erfasse tatsächlich installierte Werkzeuge. Beschaffe/baue OVMF gemäß dem festgelegten Quellstand und dokumentiere Herkunft/Hash; keine Firmwaredatei als vorhanden erfinden.

Abnahme: PE32+-UEFI-Target und ELF64-Kerneltarget enthalten richtige Architektur, Einstieg und Rechte; keine ungelösten oder impliziten Host-CRT-Symbole. Frische Konfiguration und Wiederholungsbuild funktionieren. Ein fehlendes Werkzeug erzeugt einen verständlichen Fehler. Ein Boot ist hier noch kein Erfolgskriterium.

### B-002 – Öffentliche Bootstrukturen und eigene Firmwaretypen

Abhängigkeit: B-001. Erzeuge include/boot/boot_info.h und die minimal benötigten UEFI-/ELF-/ACPI-Typdefinitionen. Implementiere keine bereits bestehende fremde Loaderlogik.

Abnahme: jede Strukturgröße und jeder Offset des Bootvertrags ist compile-time geprüft; eigene Integer-/Speicherprimitiven bestehen Grenz- und Overlaptests. EFI-/Kernel-Aufrufkonventionen werden separat geprüft.

### B-003 – Image-Erzeugung und Konfiguration

Abhängigkeit: B-002. Implementiere tools/mkimage.py für das vorgeschriebene GPT/FAT32-Image und die Dateipfade. Implementiere den kleinen boot.cfg-Parser als hosttestbare Logik.

Abnahme: Imagegröße, Partition, FAT-Verzeichnisse und Dateiinhalte unabhängig aus dem Image zurücklesen; Hashvergleich. Unbekannte Schlüssel, zu große Konfiguration und Device-Ausgabepfade abweisen. Bestehendes Image ohne --overwrite bleibt erhalten.

### B-004 – Eigener UEFI-Loader und ELF-Validierung

Abhängigkeiten: B-002/B-003. Erzeuge boot/uefi/entry.c, filesystem.c, elf.c und memory.c. Öffne ausschließlich das Startdateisystem, prüfe ELF und reserviere/lade alle Segmente.

Abnahme: normale sowie alle spezifizierten fehlerhaften ELF-Eingaben in Hosttests; Größen-/Überlauf-/Rechteprüfungen. Unter UEFI bis IMAGE_VALID gelangen. Noch kein behaupteter Kernelstart.

### B-005 – Seitentabellen und kontrollierte Übergabe

Abhängigkeit: B-004. Implementiere Bootreservierungen, normalisierte Speicherkarte, Stack/Guards, Mappings und ExitBootServices-Retry. Erzeuge eigenen Assemblerübergang.

Abnahme: stale MapKey über Testadapter; keine Allokation im finalen Übergangsfenster; korrekter CR3/Stack/ABI-Zustand. Alle noch lebenden Loaderdaten und MMIO-Cacheattribute bleiben korrekt.

### B-006 – Minimaler Kernel und Fehlerdiagnose

Abhängigkeit: B-005. Implementiere kernel_entry, minimale GDT/IDT/Fehlerstubs, Bootinfo-Prüfung und direkte COM1-Ausgabe. Liefere P1_READY erst nach wirklicher Übernahme und Prüfung.

Abnahme: normaler Eintritt, test=panic und test=pagefault; letzterer mit echtem CPU-Fehler, CR2/RIP und kontrolliertem Halt. Keine Firmware-Bootdienste nach Übergabe.

### B-007 – QEMU-Harness und negative Integrationstests

Abhängigkeit: B-006. Implementiere tools/run_qemu.py und tools/test_boot.py mit Argumentlisten, frischen VARS, Fristen und erhaltenen Logs. Erzeuge beschädigte Image-/ELF-Testfälle kontrolliert in out/.

Abnahme: erwartete Fehlercodes, kein falsches P1_READY, Timeout als Fehler, Prozessende kontrolliert. Parser-/MapKey-Tests dürfen gezielte Host-Testadapter verwenden, deren Verwendung im Bericht erkennbar bleibt.

### B-008 – P1-Reproduzierbarkeit und Abschluss

Abhängigkeit: B-007. Führe die vollständige P1-Suite aus, einschließlich 100 Boots und RAM-Varianten. Erzeuge Build-/Toolchain-/Testmanifeste und eine nutzbare Anleitung aus real getesteten Befehlen.

Abnahme: frischer Build besteht; Ergebnisse je Fall dokumentiert; keine ungeprüfte Hardware- oder Sicherheitszusage. Erst dann P1 als abgeschlossen markieren.

## Weitere festgelegte Aufgabenpakete

| ID | Abhängigkeit | Ergebnis | Abnahme |
| --- | --- | --- | --- |
| B-010 | B-008 | Physischer Buddy-Allocator und Bootreservierungen | OOM/Fragmentierung/Überlauf/Grenzen |
| B-011 | B-010 | Eigene Kernel-Seitentabellen, Heap, Guards | Schutzrechte, Mappingfehler und Ressourcenbilanz |
| B-012 | B-011 | APIC/IOAPIC, Timer, Kontextregeln | IRQ/DPC/THREAD-Regeln und Fehlertests |
| B-013 | B-012 | Threads/Scheduler/Sperren | Fairness, Wait/Timeout/Wakeup-Rennen |
| B-014 | B-013 | Vier CPUs und TLB-Shootdown | Parallele Allokation/Unmap und vollständige Ack-Regeln |
| B-020 | B-012 | HW-01 inventarisieren und erstmals booten | Geräte-/Firmwaredaten, ausgewähltes Medium, echter Bootlog |
| B-025 | B-013 | ABI-1-Detailstrukturen für Spawn/Stat/Verzeichnisse und eigener crt0 | Layouttests, Schnittstellenreview vor Implementierung |
| B-026 | B-025 | Ring 3, Syscall-Eintritt/Rückkehr, ELF-Apps | Fremder Speicherzugriff und ungültige Rückkehrzustände |
| B-027 | B-026 | Handles/IPC und libc/libos | Vererbung/Rechte, Queuevoll/Timeout/Abbruch, 10.000 Zyklen |
| B-030 | B-027 | WDM-TEST-1-Strukturlayouts/PE-Lader/DDIs | Referenz-Windows-Binär mit vollständigem Importabschluss |
| B-031 | B-030 | WDM-TEST-1-Abnahme | Unveränderte Testbinär, Referenzvergleich und Lebensdauer |
| B-035 | B-027 | AHCI/VFS/ext2-Formatter und eigener Dateitreiber | Dateien, sync, volle Disk, abgewiesene Features, dirty read-only |
| B-036 | B-035 | init/serviced/configd und eigene Shell | DAG/Restartlimits/Configfehler/Dateibefehle |
| B-040 | B-036 | Display-/Inputtreiber, Surface-/Event-Payload-Spec | Versionierte Felder, Größen/Rechte/Lebensdauer |
| B-041 | B-040 | displayd/inputd/libui | Doppelbufferbesitz, Fokus, Größenwechsel und Clientabsturz |
| B-042 | B-041 | Alle 0.1-Basisapps | PROD-DESKTOP-01 durchgängig |
| B-043 | B-014/B-020/B-031/B-042 | Desktop 0.1-Abnahme | Hardware/VM, Last, Budgets und WDM-Profil |
| B-050 | B-036 | Ethernet/IPv4/TCP/UDP/DNS/DHCP | Protokollzustände und Paket-/Timeouttests |
| B-051 | B-050 | Entropie, eigene TLS-Bibliothek, Zertifikate | Bekannte Vektoren und Negativtests; keine frühe Sicherheitsfreigabe |
| B-052 | B-035 | ext4/JBD2 und Migrationstool | Stromausfall-/Replay-/Flush-/Datenprüfung |
| B-053 | B-052 | Paketformat-Bytevertrag und Implementierung | Integrität, Traversal, Limits und Abbruch |
| B-054 | B-053 | Installer, A/B-Loader/Updater und Recovery | Leermedium, Updateabbrüche, Bootversuche/Rückkehr |
| B-055 | B-031 | KMDF-ECHO-1 | Unverändertes Echo-Binär und vollständige getestete Funktionsmenge |
| B-056 | B-051/B-054/B-055 | Vorschau 0.2 | Qualitätstabelle einschließlich 24-h-Test |
| B-060 | B-050/B-055 | NETKVM-LAB-1-Faktenmanifest und DDI-/NDIS-Verträge | Reales Paket/Hashes/Imports/Referenzverhalten |
| B-061 | B-060 | NetKVM-Gerätepfad | Echtes externes Binär auf virtuellem VirtIO-Net-Gerät |
| B-062 | B-056 | IPv6, HDA-Audio, ACPI-Shutdown/Reboot und Idle | Je Funktion versionierte Geräte-/Testprofile |
| B-063 | B-061/B-062 | Vorschau 0.3 | Mindestens ein externes Treiberprofil und 72-h-Test |

Spätere Detail-Specs in B-025/B-040/B-053/B-060 präzisieren bereits entschiedene Architektur. Sie sind Arbeitsergebnisse der jeweiligen Implementierungsetappe, keine ungelösten Voraussetzungen des ersten Schritts.

## Arbeitsablauf

Aufgabe auswählen, Abhängigkeiten prüfen, passende Specs lesen, Verhalten und Fehlerpfade implementieren, tatsächliche Tests ausführen, Ergebnisse dokumentieren. Status erst danach auf abgeschlossen setzen. Neue Probleme werden als konkrete Aufgaben mit Fehlernachweis erfasst. Keine Kalenderdaten oder geschätzten Fortschrittsprozente ohne Grundlage.
