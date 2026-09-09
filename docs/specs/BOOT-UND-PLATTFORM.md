# Boot und Plattform – SPEC-001/002/003

Baseline 1.0. Normativ für P1. Zahlen sind Designvorgaben, keine gemessenen Ergebnisse.

## Plattformprofile

| Profil | CPU/RAM/Geräte | Zweck |
| --- | --- | --- |
| VM-BOOT | QEMU 11.1.1; pc-q35-11.1; TCG; qemu64; 1 vCPU; 512 MiB; Standard-VGA/GOP; COM1; kein Netz | Reproduzierbarer erster Start |
| VM-SMP | Identisch, 4 vCPUs und 2 GiB | Speicher, Prozesse, Synchronisation |
| VM-DESKTOP | Identisch, 4 vCPUs und 4 GiB; 1280×720; QEMU-xHCI mit USB-Tastatur/-Maus; AHCI-Systemdisk; VirtIO-Net bei P6 | Produktabnahme |
| HW-01 | x86-64 mit NX, SSE2, Local APIC; UEFI ≥2.7, ACPI ≥6.0; ≥4 GiB RAM; GOP-Bildausgabe; AHCI-SATA-Testdisk; xHCI-USB-Eingabe | Erster echter Rechner |

CPU-Baseline: Little Endian, Long Mode, 4-KiB-Seiten und vierstufiges Paging; kein LA57, AVX oder x2APIC als Voraussetzung. P1 läuft nur auf dem Bootprozessor. UEFI-Runtime-Aufrufe nach der Übergabe sind zunächst deaktiviert; ihre Speicherbereiche bleiben reserviert.

VM-Firmware: OVMF-X64 aus edk2-stable202605, ohne Secure Boot für Entwicklungsimages. CODE ist schreibgeschützt, jeder Lauf erhält eine frische Kopie der VARS-Datei. Der erste Infrastrukturauftrag beschafft/erzeugt die Firmware und zeichnet SHA-256 sowie die tatsächliche Bauanleitung auf. Vorhandene Sicherheitskonfiguration realer Rechner wird durch Testwerkzeuge nicht automatisch verändert.

HW-01 ist ein Konformitätsprofil, keine Behauptung vorhandener Hardware. B-020 erfasst Hersteller, Modell, Geräte-IDs, Firmware und Testmedium vor dem ersten Hardwareversuch.

## Bootmedium und Loader

P1-Image: Raw-Disk, 512 MiB, GPT, eine FAT32-EFI-Systempartition ab LBA 2048; logische Sektorgröße 512 Byte. ESP-Partitions-GUID nach UEFI/GPT. Dateipfade:

- EFI/BOOT/BOOTX64.EFI: unser PE32+-UEFI-Loader.
- OS/kernel.elf: unser ELF64-Kernel.
- OS/boot.cfg: optionale UTF-8-Konfiguration, höchstens 4096 Bytes.

Fehlt boot.cfg, gelten die Defaults. Unterstützte Schlüssel: log=serial (Default), test=none|panic|pagefault. Unbekannte Schlüssel sind Fehler. Keine Includes, Variablenersetzung oder Skriptsprache. Ein späteres initrd wird erst ab P3 verpflichtend.

Der Loader öffnet das Dateisystem des Geräts, von dem sein eigenes Image geladen wurde. Er lädt keine Datei von einem zufällig gefundenen anderen Datenträger. Bootdienste dürfen zum Lesen verwendet werden; unser Loader selbst bleibt eigene Implementierung.

## Kernelabbild

ELF64, Little Endian, EM_X86_64, ET_EXEC. P1 lädt ausschließlich PT_LOAD-Segmente. PT_NULL, PT_PHDR, PT_NOTE und nicht ausführbares PT_GNU_STACK dürfen als geprüfte Metadaten vorkommen; andere Typen werden abgewiesen. Kein Interpreter, keine dynamischen Relokationen. Basisadresse 0xffffffff80000000; zulässiger Kernelbereich bis exklusiv 0xffffffffc0000000.

Prüfungen: ELF-Kennung/Klasse/Maschine/Version, Datei- und Headergrößen, höchstens 64 Program Headers, p_filesz ≤ p_memsz, vollständige Dateigrenzen, alle Additionen überlaufgeprüft, konsistente Seitenausrichtung, keine überlappenden virtuellen PT_LOAD-Bereiche und kein Segment mit W+X. Der Einstieg liegt in einem ausführbaren geladenen Segment. Kerneldatei und gesamter geladener Bereich sind jeweils auf 64 MiB begrenzt.

p_paddr wird nicht als Autorität verwendet. Der Loader reserviert einen zusammenhängenden physischen Bereich unter 4 GiB für die seitengerundete virtuelle Spannweite, kopiert p_filesz, nullt den Rest bis p_memsz und erstellt explizite virtuelle Mappings. Auch seitengerundete Segmentbereiche dürfen sich nicht überlappen. RX-Code, R-Konstanten und RW/NX-Daten werden getrennt gemappt. Kein Sprung auf ungeprüfte Adressen.

## Speicherübergabe

Alle Adressen in boot_info_v1 sind physische Adressen, sofern das Feld ausdrücklich _virt heißt. Der Kernel erhält in RDI den virtuellen Zeiger auf diese Struktur über den Direktmapping-Bereich. Native Datentypen: u32/u64 mit expliziter Breite. Strukturausrichtung 8, keine implizit unterschiedlich gepackten Compilerlayouts.

| Offset | Feld | Typ / Bedeutung |
| --- | --- | --- |
| 0 | magic | 8 Bytes ASCII OSBOOT01 |
| 8 | abi_major | u32 = 1 |
| 12 | abi_minor | u32 = 0 |
| 16 | struct_bytes | u32 = 256 |
| 20 | flags | u32; Bit 0 = Framebuffer gültig; übrige Bits 0 |
| 24 | self_phys | u64 |
| 32 | mmap_phys | u64; physische Adresse normalisierter Speichereinträge |
| 40 | mmap_count | u32 |
| 44 | mmap_stride | u32 = 32 |
| 48 | reservations_phys | u64 |
| 56 | reservation_count | u32 |
| 60 | reservation_stride | u32 = 24 |
| 64 | rsdp_phys | u64; geprüfte ACPI-RSDP-Adresse |
| 72 | framebuffer_phys | u64 |
| 80 | framebuffer_bytes | u64 |
| 88 | framebuffer_width | u32 |
| 92 | framebuffer_height | u32 |
| 96 | framebuffer_stride_pixels | u32 |
| 100 | framebuffer_format | u32; 1=BGRX8888, 2=RGBX8888 |
| 104 | kernel_phys_start | u64; Beginn des reservierten Kernelbereichs |
| 112 | kernel_phys_bytes | u64 |
| 120 | initial_cr3_phys | u64 |
| 128 | stack_top_virt | u64; oberes exklusives Ende |
| 136 | initrd_phys | u64; 0 in P1 |
| 144 | initrd_bytes | u64; 0 in P1 |
| 152 | cmdline_phys | u64; geprüfte Konfigurationskopie |
| 160 | cmdline_bytes | u32; ohne abschließendes NUL |
| 164 | boot_cpu_apic_id | u32 |
| 168 | boot_id | u64; Diagnosesequenz, 0 in P1, keine Entropie |
| 176 | reserved | zehn u64, alle 0 |

Größen und Offsets werden durch _Static_assert geprüft. Minor-Erweiterungen dürfen später zusätzliche Felder hinten anhängen; Major-Wechsel ist inkompatibel. P1 akzeptiert nur Major 1 und mindestens 256 Bytes; unbekannte gesetzte Flags werden zurückgewiesen.

Speichereintrag, 32 Bytes: base:u64, length:u64, kind:u32, reserved:u32=0, firmware_attributes:u64. kind: 1=verfügbar, 2=reserviert, 3=ACPI reclaim, 4=ACPI NVS, 5=MMIO, 6=Firmware Runtime, 7=defekt. Bereiche müssen sortiert, nicht überlappend und überlaufsicher sein.

Reservierung, 24 Bytes: base:u64, length:u64, kind:u32, reserved:u32=0. kind: 1=Kernel, 2=Bootdaten, 3=Seitentabellen, 4=Stack, 5=Loader, 6=initrd, 7=Framebuffer. Reservierungen können innerhalb eines Speichereintrags liegen; der Allokator zieht ihre Vereinigungsmenge von verfügbarem Speicher ab.

Nur freigegebene EfiConventionalMemory- sowie nach ExitBootServices grundsätzlich rückgewinnbare Loader-/BootServices-Seiten können als verfügbar normalisiert werden. Jede noch benötigte eigene Allokation ist zusätzlich reserviert. Runtime, ACPI NVS, MMIO, unbekannte und defekte Bereiche bleiben reserviert. ACPI reclaim wird erst nach expliziter Übernahme der benötigten Tabellen freigegeben.

## Paging und CPU beim Übergang

- Direktmapping-Basis: 0xffff800000000000. P1 mappt benötigte RAM-Bereiche unter 4 GiB; zusätzliche MMIO-Seiten nur gezielt.
- Die Loader-Ausführungsseiten behalten bis zum Sprung ihre nötige Identitätsabbildung. Der Kernel entfernt diese nach Übernahme seiner Tabellen.
- Eigener Kernelstack: 64 KiB, RW/NX; Guard Page bei 0xffffffffc0000000, nutzbarer Bereich ab 0xffffffffc0001000 bis exklusiv 0xffffffffc0011000. stack_top_virt ist dieses obere Ende und 16-Byte-ausgerichtet.
- Das RAM-Direktmapping ist NX; Aliase physischer Kernel-Code-/Konstantenseiten sind schreibgeschützt. Ein RX-Codebereich darf nicht über einen zweiten RW-Alias veränderbar bleiben.
- CR0.WP und NXE sind gesetzt; CR3 enthält die übergebene Tabelle; Interrupts aus, Direction Flag gelöscht.
- Der Framebuffer wird separat mit geeigneten Cacheattributen gemappt; P1 verwendet konservativ UC. Kein gleichzeitiges Mapping desselben MMIO-Bereichs mit widersprüchlichen Cachetypen.
- Loader und Kernel laufen zunächst ohne Floating-Point-/SIMD-Code.
- Sprung in kernel_entry ist ein Assemblerübergang mit RSP=stack_top_virt und RDI=Direktmapping(boot_info). kernel_entry richtet GDT/IDT für den minimalen Fehlerpfad ein und ruft kernel_main unter SysV AMD64 auf. RSP ist vor CALL 16-Byte-ausgerichtet.
- Firmwareaufrufe und der UEFI-Eintritt nutzen dagegen die Microsoft-x64-Aufrufkonvention. Beide ABIs werden nicht verwechselt.

## ExitBootServices-Vertrag

Vor dem finalen Aufruf werden Kernel, Stack, Seitentabellen, Rohspeicherkarte, normalisierte Karte, Reservierungen und Bootdaten vollständig reserviert. Der Kartenpuffer enthält zusätzlichen Platz; Kapazitätsgrenze 4096 Deskriptoren.

Danach: GetMemoryMap → normalisieren ohne neue Allokationen → ExitBootServices mit genau diesem MapKey. Zwischen letztem GetMemoryMap und ExitBootServices keine Protokoll-/Konsolenaufrufe oder Allokationen. Bei EFI_INVALID_PARAMETER wird in die vorhandenen Puffer eine neue Karte gelesen und erneut versucht, höchstens dreimal. Benötigt die Karte mehr als die reservierte Kapazität, kontrollierter Abbruch statt stiller Kürzung.

Nach erfolgreichem ExitBootServices keine Boot-Service-Aufrufe. Direktzugriff auf die serielle Diagnose bleibt zulässig. Erst danach CR3-/Stack-Übergang und Sprung. Der Kernel kopiert/übernimmt Bootdaten, bevor deren Seiten wiederverwendet werden.

## Diagnose und Fehlercodes

COM1: I/O 0x3f8, 115200 Baud, 8N1, Polling mit begrenztem Warten. Zeilenformat: OS|<Komponente>|<Ereignis>|<Details>. Vor ExitBootServices darf zusätzlich UEFI-Textausgabe genutzt werden. Auf HW-01 ohne UART: GOP-Fehleranzeige und dokumentierte Bildschirmdiagnose.

Pflichtmarker im Normalboot: LOADER_START, IMAGE_VALID, EXIT_BOOT_SERVICES_OK, KERNEL_ENTRY, BOOTINFO_VALID, P1_READY. Bei test=panic beziehungsweise test=pagefault wird nach BOOTINFO_VALID statt P1_READY der echte Fehler ausgelöst. Die Ausgabe enthält PANIC_TEST beziehungsweise PAGE_FAULT mit CR2/RIP; beide Negativläufe dürfen kein P1_READY enthalten.

Fehlercodes: B001=Firmwareprofil, B002=Datei fehlt, B003=ELF ungültig, B004=Speicher fehlt, B005=Speicherkarte ungültig/zu groß, B006=ExitBootServices gescheitert, B007=Bootinfo ungültig, B008=unerwartete Exception. Bei einem Fehler nie P1_READY ausgeben. Vor dem Firmwareübergang Fehler anzeigen und mit Fehlerstatus zurückkehren; nach dem Übergang Interrupts deaktivieren und kontrolliert halten.

## P1-Abnahme

- 100 unabhängige Normalboots mit frischen VARS, jeweils innerhalb 30 Sekunden bis P1_READY.
- RAM-Varianten 256/512/1024 MiB; keine feste physische Adresse voraussetzen.
- Fehlende Kerneldatei, abgeschnittene ELF-Datei, falsche Maschine, Überlauf/Segmentüberschneidung, W+X und ungültiger Einstieg werden abgewiesen.
- Hosttest injiziert mindestens einen veralteten MapKey; Wiederholung ohne Speicherverlust.
- Absichtlicher Panic und Page Fault liefern die erwartete Diagnose.
- Tests prüfen Image und Kontrollfluss, nicht lediglich vorab gedruckte Marker.
- Jede Ausführung protokolliert Tool-/Firmware-/Image-Hashes und tatsächlichen Exit-/Timeoutzustand.

Primärreferenzen: [UEFI-Spezifikationen](https://uefi.org/specifications), [Intel SDM](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html), [OVMF-Quellstand](https://github.com/tianocore/edk2/releases/tag/edk2-stable202605). Die konkrete Übergabestruktur ist unser eigener Vertrag.
