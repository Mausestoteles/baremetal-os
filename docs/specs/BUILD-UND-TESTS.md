# Build, Werkzeuge und Tests

Baseline 1.0. Dieses Dokument spezifiziert das in B-001 bis B-008 zu implementierende Buildsystem. Die nachstehenden OS-Befehle funktionieren erst nach diesen Aufträgen.

## Host und Werkzeuge

Primärhost Windows 11 x64 mit PowerShell, Git und Python 3.11.x. Cross-Build ausschließlich mit expliziten Targets; kein WSL als Voraussetzung. Sekundärhost Ubuntu 24.04 x64 kann nach identischen Artefaktprüfungen zugelassen werden.

| Werkzeug | Baseline | Zweck |
| --- | --- | --- |
| LLVM/Clang/LLD | 22.1.8 | Compiler, Assembler, ELF- und PE-Linker, Objektprüfung |
| CMake | 3.31.8 | Buildgraph und Presets |
| Ninja | 1.12.1 | Buildausführung |
| Python | 3.11.x | Host-Image-/Testwerkzeuge; Patchversion im Manifest |
| QEMU | 11.1.1 | CPU-/Geräteemulation |
| OVMF | edk2-stable202605, X64 ohne Secure Boot im Testprofil | VM-Firmware |
| WDK/SDK | 10.0.26100, nur im Windows-Treiberauftrag | Referenztreiber, kein Kernel-Buildbedarf in P1 |

Offizielle Quellen stehen in [QUELLEN](../QUELLEN.md). B-001 protokolliert tatsächlich installierte Versionen, Herkunft, Archiv-/Firmware-SHA-256 und Befehlszeilen in out/toolchain-manifest.json. Versionen sind gewählte Ausgangspunkte; ein erfolgreicher lokaler Build ist noch nicht nachgewiesen. Kein automatisches Herunterladen oder Ausführen ungeprüfter Abhängigkeiten durch einen normalen Build.

## Geplanter Quellbaum

~~~text
boot/uefi/                 Eigener Loader und minimale UEFI-Definitionen
kernel/arch/x86_64/         Einstieg, Paging, GDT/IDT, IRQ/SMP
kernel/core/               Speicher, Threads, Handles, IPC
kernel/drivers/            Native Treiber
kernel/compat/windows/     Windows-DDIs und Fassadenobjekte
include/os/                Native öffentliche Verträge
include/boot/              boot_info_v1
lib/libc/                  Eigene Standardbibliothek
lib/libos/                 Eigene System-API
lib/libui/                 Eigener GUI-Client und Toolkit
services/                  Eigene User-Dienste
apps/                      Native Basisapps
cmake/toolchains/          Cross-Target-Konfiguration
tools/                     Eigene Host-Image- und Prüfwerkzeuge
tests/host/                Hardwareunabhängige Tests
tests/boot/                Boot-/Fehlertests
tests/kernel/              Kernel- und Prozessprüfungen
tests/compat/              Treiberverträge und Referenzprofile
~~~

Verzeichnisse werden mit der jeweiligen Implementierungsaufgabe erstellt. Keine leeren Platzhalter sollen einen vorhandenen Funktionsumfang vortäuschen.

## Compiler- und Linkvertrag

C17, Warnungen für eigenen Code als Fehler, -ffreestanding, -fno-builtin, -nostdinc mit eigenen Zielheadern, keine implizite Host-CRT. Debug -O0/-g, Release -O2 mit Symbolartefakt. Kein LTO in P1, damit Diagnosen und ELF-Struktur einfach bleiben.

Kernel: --target=x86_64-unknown-none-elf, -mno-red-zone, -mcmodel=kernel, -fno-pic, -fno-pie, -fno-stack-protector in P1, keine SSE/MMX/FP-Codegenerierung. Die genaue akzeptierte Clang-Flagkombination wird in B-001 per Objektprüfung belegt. Eigene Stack-Canary-Unterstützung ab P2; das P1-Profil ist keine Release-Sicherheitsfreigabe.

ELF-Link über ld.lld und eigenes Linkerskript; getrennte RX/R/RW-PT_LOAD-Segmente, Entry kernel_entry, Basis 0xffffffff80000000, kein dynamischer Interpreter, keine ungelösten Symbole. BSS wird über p_memsz beschrieben.

UEFI: --target=x86_64-pc-windows-msvc, -ffreestanding, -fno-stack-protector, -fshort-wchar, keine impliziten Stack-Probe-/CRT-Abhängigkeiten; lld-link /subsystem:efi_application /entry:efi_main /nodefaultlib /machine:x64. Falls der Compiler Hilfsaufrufe erzeugt, werden sie entweder durch dokumentierte Flags vermieden oder durch eigene passende Routinen erfüllt. PE-Relokationen müssen erhalten bleiben.

Boot- und Kernelobjekte werden getrennt gebaut. EFIAPI verwendet Microsoft x64; Kernel-native Aufrufe SysV AMD64. Protokollstrukturen erhalten Compile-time-Größen-/Offsetprüfungen.

## Herzustellende Entwicklerbefehle

Nach B-001 bis B-008:

~~~text
cmake --preset debug
cmake --build --preset debug
python tools/mkimage.py --profile vm-boot --output out/os.img
python tools/run_qemu.py --profile vm-boot --image out/os.img
python tools/test_boot.py --suite p1
~~~

Presets liegen im Repository; lokale Toolpfade in ignorierter CMakeUserPresets.json oder Umgebungsvariablen. CMake-Ninja-Ausgabe unter build/debug, Lieferartefakte unter out/. Fehlende Abhängigkeiten führen zu einer verständlichen Fehlermeldung mit erwarteter Version, nie zu einem scheinbar erfolgreichen leeren Build.

mkimage.py erzeugt GPT/FAT32 in einer neuen Image-Datei und validiert Struktur und enthaltene Hashes. Es akzeptiert in P1 ausschließlich reguläre Dateien als Ausgabe, keine physische Disk oder Device-Pfade. Bestehende Images werden nur mit explizitem --overwrite ersetzt.

run_qemu.py löst Tool-/Firmwarepfade aus dem Manifest auf, baut eine Argumentliste ohne Shell-Stringinterpolation und startet QEMU. TCG ist für Referenztests festgelegt. Optionale beschleunigte lokale Läufe werden als anderes Profil protokolliert.

## Artefakte

BOOTX64.EFI, kernel.elf, kernel.map, os.img, build-manifest.json, toolchain-manifest.json, Testlogs und maschinenlesbarer Testbericht. Private Schlüssel, lokale Accounts, Tokens, absolute persönliche Pfade und unbereinigte Hostumgebungen gehören nicht in committed Manifeste.

Ein Buildmanifest enthält Quellcommit, Dirty-Status, Konfiguration, Target, Werkzeuge, Eingabe-/Ausgabe-Hashes. SOURCE_DATE_EPOCH wird für reproduzierbare Zeitfelder verwendet. Bitidentische Builds sind ein nachzuweisendes Ziel; nicht bloß aus einem festen Versionsnamen ableiten.

## Prüfungen

B-001: Formate/Architektur/Entrypoints/Imports, richtige Segmentrechte, kein Host-CRT. B-002: sizeof/offsetof sämtlicher Bootstrukturen. B-003/B-004: Hosttests für ELF-/Konfigurationsparser und Imageformat. B-005/B-006: normaler Boot, Bootinfo-Übernahme und Exceptions. B-007: kaputte Eingaben und ExitBootServices-Retry. B-008: vollständige P1-Suite und Manifest.

Testharness: 30-Sekunden-Frist pro Boot, bei Timeout Prozess stoppen und Log behalten. QEMU wird nicht durch einen festen Schlaf als „erfolgreich“ gewertet. Positivtests erwarten geordnete Marker und einen passenden Prüfabschluss; Negativtests erwarten konkrete Fehler und Abwesenheit von P1_READY.

Hosttests eigener Parser verwenden Clang-Sanitizer, soweit vom jeweiligen Host unterstützt; ergänzend Linux-Testjob für ASan/UBSan. Fehlende Sanitizerfähigkeit wird offen gemeldet und nicht als bestanden markiert. Fuzzing-Korpora enthalten gültige, gekürzte, überlaufende und überlappende Eingaben.

## Automatisierung

In B-001 einen dokumentations-/formatprüfenden Job und Artefaktprüfung vorbereiten; der vollständige Bootjob folgt in B-008. Neue CI-Dateien dürfen keine grünen Platzhalterjobs als Kerneltests bezeichnen. Standardmäßig keine kostenpflichtigen Runner oder Cloud-Ressourcen anlegen.

Auf echter Hardware werden ausschließlich die explizit für B-020 ausgewählten Medien verwendet. Die Erstellung eines VM-Images autorisiert keinen Schreibzugriff auf einen physischen Datenträger.
