# Kernel, Sicherheit und native ABI

Baseline 1.0; normative Architektur für P2–P4. Bootdetails werden ausschließlich durch die Boot-Spec festgelegt.

## Speicher

Seite 4096 Bytes. User-Adressen: ab 0x0000000000010000 bis exklusiv 0x0000800000000000; oberste Seiten für Stack/Guards reserviert. Kerneltext ab 0xffffffff80000000, Direktmapping ab 0xffff800000000000. Kernelmappings sind supervisor-only; keine User-Abbildung physischer Adressen.

Physische Verwaltung: Buddy-Allocator, Ordnungen 0–18, getrennte Zonen DMA32 und NORMAL. Bootreservierungen werden vor erster Ausgabe abgezogen. Kleine Kernelobjekte: Slab-Klassen 16,32,64,128,256,512,1024,2048 Bytes; größere Allokationen über Seiten. Größen-/Ausrichtungsarithmetik wird geprüft; Ressourcenmangel liefert Fehler, keine implizite Panic in normalen Anfragen.

Virtueller Speicher: zunächst eager allocation; Demand Paging und Copy-on-Write ab 0.2. map/unmap/protect sind gegen parallele Zugriffe synchronisiert; TLB-Shootdown mit bestätigter Ausführung auf allen betroffenen CPUs. Kein Freigeben einer Seite vor beendetem Shootdown und letzter Referenz.

W^X gilt für native Programme; Nullseite bleibt unmapped. User-Pointer werden auf kanonischen Bereich, Überläufe und Rechte geprüft und über fehlertolerante copyin/copyout kopiert. Überprüfung und spätere Verwendung dürfen kein ungeschütztes TOCTOU-Fenster auf Kernelobjekte schaffen.

## CPU, Interrupts und Sperren

Eigene GDT/TSS, IDT und IST-Stacks für Double Fault/NMI. Legacy-PIC nach APIC-Initialisierung maskieren. Local APIC und IOAPIC zuerst; HPET oder kalibrierter APIC-Timer als Zeitquelle, invariant TSC erst nach Prüfung/Kalibrierung als Optimierung.

Explizite Kontexte: THREAD, DPC, IRQ. Blockierende Operationen nur in THREAD. IRQ-Handler quittieren Hardware und delegieren längere Arbeit. DPCs blockieren nicht. Windows-IRQL ist ein separater Vertrag, der auf Maskierung und zulässige Operationen abgebildet wird; es ist keine Thread-Priorität.

Mutexe für Threadkontext, Spinlocks für kurze nicht blockierende Bereiche, atomare Referenzen für Lebensdauer. Globale Sperrreihenfolge: Prozess/Adressraum → Objekt → Gerät/Queue. Keine Allokation oder fremden Callbacks unter IRQ-Spinlocks. Debugbuilds prüfen Sperrreihenfolge und Kontext.

SMP: ACPI-MADT auswerten, zusätzliche CPUs über INIT/SIPI starten, pro CPU Stack/TSS/Runqueue, synchronisierter Start. FXSAVE/FXRSTOR-Zustand pro Thread vor Zulassung von User-SSE2; kein AVX-Vertrag in 0.1.

## Prozesse und Scheduling

Prozess: Adressraum, Handle-Tabelle, Identität, Threads und Exitstatus. Threadzustände NEW, READY, RUNNING, BLOCKED, DYING, DEAD. Jeder Übergang besitzt einen eindeutigen Besitzer; nur READY ist in einer Runqueue.

Präemptiver Scheduler: Prioritäten 0–31, höhere Zahl bevorzugt, normale Apps 16, interaktive Dienste 18, Systemarbeit 20–23, 24–31 nur privilegiert. Gleiche Priorität Round Robin mit 4 ms Quantum. Wartende normale Threads erhalten begrenztes Aging; jede runnable normale Aufgabe muss im Lasttest Fortschritt machen. Echtzeitgarantien werden nicht zugesagt.

Jede CPU hat eine Runqueue; periodischer Ausgleich bei ungleicher Last. Blockieren entfernt den Thread atomar aus Ausführung und registriert sein Wait-Objekt. Wakeup/Timeout/Abbruch dürfen denselben Wait nur einmal abschließen.

## Handles und Berechtigungen

Handle ist u64: untere 32 Bit Index, obere 32 Bit Generation. Null ungültig; Generation liegt zwischen 1 und 0x7fffffff. Damit bleibt ein erfolgreich zurückgegebenes Handle im nichtnegativen i64-Bereich der Syscall-ABI. Ein Slot wird nach Verwendung der maximalen Generation dauerhaft ausgemustert, statt zu einem alten Wert zurückzuspringen. Jedes Handle gehört zu genau einer Prozesstabelle.

Rechtebits: READ=1, WRITE=2, EXEC=4, MAP=8, WAIT=16, SIGNAL=32, DUP=64, TRANSFER=128, ADMIN=256. Objektarten schränken gültige Rechte weiter ein. Duplizieren/Übertragen darf Rechte nur verringern; keine automatische Vererbung aller Elternhandles. Prozessstart benennt vererbte Handles ausdrücklich.

Benutzer-ID/Gruppen-ID u32; UID 0 ist Systemverwaltung. User-Apps laufen als normale UID. Dateizugriff verbindet Identität/Modusrechte mit bereits gewährten Handle-Rechten. Das Laden eines Kerneltreibers verlangt einen ADMIN-Zugang des Verwaltungsdienstes.

## Native Funktionen und Programme

Native Objekt-/Programmformate ELF64/SysV AMD64, Little Endian, C17-Datenlayout. Programme in 0.1 statisch ET_EXEC, eigener crt0, eigene libc/libos; kein Host-libc. App-Ladebereich ab 0x400000; gleiche ELF-Grenzprüfungen wie beim Kernel, aber ausschließlich User-Adressen.

Initialer User-Stack: argc:u64, argv-Zeigerliste inklusive Null, envp-Liste inklusive Null, Hilfsvektor aus u64-Paaren bis (0,0); Strings liegen vollständig im User-Stack. Stack 1 MiB initial, untere Guard Page; maximal 8 MiB pro Thread, explizite Vergrößerung. RSP beim Übergang 16-Byte-ausgerichtet; crt0 stellt vor C-Aufrufen die SysV-Regeln her.

Auxv-Schlüssel: 6=PAGESZ(4096), 9=ENTRY, 0x5000=ABI_MAJOR(1), 0x5001=BOOTSTRAP_INFO_PTR. Letzterer zeigt auf eine versionierte User-Kopie der ausdrücklich vererbten Handles und Prozessmetadaten. Keine Kernelzeiger im Stack.

crt0 initialisiert libc, ruft main(argc,argv) auf und beendet über process_exit. Anfangs kein dynamischer Linker, keine dlopen- oder POSIX-Komplettzusage. 0.2 führt ELF-PIE/dynamisches Linken und ASLR mit eigenen Bibliotheken ein; ABI-Wechsel werden versioniert.

## Syscall-ABI 1

SYSCALL: RAX=Nummer, Argumente in RDI,RSI,RDX,R10,R8,R9. RCX/R11 werden zerstört; übrige Register außer Rückgabe werden bewahrt. Rückgabe RAX: nichtnegative Ergebnisse oder negatives OS-Fehlerenum. Rückkehr nach geprüfter kanonischer User-RIP/RSP und erlaubten RFLAGS; ungültiger Zustand beendet den Prozess. IRETQ darf als sicherer Rückkehrpfad verwendet werden.

Pointer sind u64, Längen u64; Strukturen beginnen mit size:u32 und version:u32, sofern sie nicht ausdrücklich Bytepuffer sind. Kein ABI-Feld mit compilerabhängigem enum/size_t/bool.

| Nr. | Name | Argumente | Ergebnis |
| --- | --- | --- | --- |
| 0 | abi_query | output, capacity | Bytes der Antwort |
| 1 | process_exit | exit_code | kehrt nicht zurück |
| 2 | thread_yield | keine | 0 |
| 3 | clock_monotonic | keine | Nanosekunden seit Kernelstart |
| 4 | handle_close | handle | 0 |
| 5 | handle_duplicate | handle, reduced_rights | neues Handle |
| 6 | vm_map_anon | hint, bytes, flags | User-Adresse |
| 7 | vm_unmap | page_aligned_address, bytes | 0 |
| 8 | port_create | queue_capacity | Handle |
| 9 | port_send | port, message, bytes, handles, count, deadline_ns | 0 |
| 10 | port_receive | port, buffer, capacity, handles_out, handle_capacity, deadline_ns | Bytes; siehe IPC-Vertrag |
| 11 | object_wait | handle, event_mask, deadline_ns | eingetretene Bits |
| 12 | process_spawn | versioned_spec, spec_bytes | Prozesshandle |
| 13 | thread_create | user_entry, user_arg, stack_top | Threadhandle |
| 14 | thread_exit | exit_code | kehrt nicht zurück |
| 15 | file_open | directory_handle, utf8_path, bytes, flags, mode | Dateihandle |
| 16 | file_read | handle, buffer, bytes, offset | gelesene Bytes |
| 17 | file_write | handle, buffer, bytes, offset | geschriebene Bytes |
| 18 | file_sync | handle | 0 nach dauerhaftem Abschluss |
| 19 | file_stat | handle, output, capacity | Bytes |
| 20 | directory_read | handle, cookie, output, capacity | Bytes |
| 21 | file_rename | old_directory, old_path, old_bytes, new_directory, new_path, new_bytes | 0 |
| 22 | file_unlink | directory, path, bytes, flags | 0 |
| 23 | directory_create | directory, path, bytes, mode | 0 |
| 24 | memory_share | address, bytes, reduced_rights | Shared-Memory-Handle |
| 25 | device_control | handle, code, in_buffer, in_bytes, out_buffer, out_bytes | Bytes |
| 26 | process_wait | process_handle, deadline_ns | Exitcode |
| 27 | debug_write | utf8_buffer, bytes | Bytes; privilegiert außer Testprofil |
| 28 | thread_sleep | deadline_ns | 0 |
| 29 | vm_map_shared | memory_handle, hint, flags | User-Adresse |

Nummern ab 30 reserviert und liefern ENOSYS, bis eine Spec sie vergibt. Prozess-/Thread-Exitcodes sind u32 im Bereich 0–255; höhere Werte werden auf die unteren 8 Bit begrenzt. Dadurch kollidiert process_wait nicht mit negativen Syscall-Fehlern. Netzwerk beginnt später über versionierte device_control-/Serviceverträge, ohne heimliche neue Nummern.

Fehlerenum: EPERM=1, ENOENT=2, EIO=3, ENOMEM=4, EINVAL=5, EBADF=6, EFAULT=7, EAGAIN=8, ETIMEDOUT=9, ECANCELED=10, ENOSYS=11, EBUSY=12, EEXIST=13, ENOSPC=14, EOVERFLOW=15, ENOTSUP=16, EMSGSIZE=17, EPIPE=18. Fehlerwerte sind unsere eigenen, keine zufällige POSIX-Zahlenübernahme.

vm_map flags READ=1,WRITE=2,EXEC=4; W+X verboten. hint=0 wählt eine freie Adresse, sonst nur ein unverbindlicher Vorschlag. vm_unmap muss vollständig gemappte ganze Seiten erfassen und scheitert ansonsten ohne Teiländerung. file_open flags READ=1,WRITE=2,CREATE=4,EXCLUSIVE=8,TRUNCATE=16,DIRECTORY=32. TRUNCATE benötigt WRITE. Pfade relativ zum Directory-Handle; absolutes / benötigt das explizit erhaltene Root-Handle.

## IPC

Ports sind begrenzte FIFO-Queues: 1–256 Nachrichten, je höchstens 4096 Nutzbytes und 8 Handles. Nachricht und alle Handle-Übertragungen werden vollständig angenommen oder gar nicht. TRANSFER übergibt eine neue Referenz mit reduzierten Rechten; das Senderhandle bleibt bestehen.

deadline_ns: 0=nicht blockieren, UINT64_MAX=unbegrenzt, sonst absolute monotone Frist. Eine abgelaufene blockierende Operation liefert ETIMEDOUT. Schließen des letzten Empfangszugangs beendet wartende Sender mit EPIPE.

port_receive kopiert bei zu kleinem Nutz-/Handle-Puffer nichts und entfernt die Nachricht nicht; EMSGSIZE. Empfangene Handleanzahl wird im Nachrichtenkopf angegeben: size:u32,version:u16=1,handle_count:u16,request_id:u64,opcode:u32,flags:u32, gefolgt vom Payload. Kernel prüft mindestens 24 Bytes und konsistente Größe. Nutzlänge inklusive Kopf ist der Rückgabewert.

Spawn-, Stat- und Verzeichniseintragslayouts werden vor B-025 als ABI-1-Ergänzung festgehalten; ihre Semantik und Nummern sind hier fixiert. Sie sind keine Voraussetzung für P1.

## Native Treiber

driver_ops_v1: size/version, probe(device), start(device), submit(request), cancel(request), stop(device), remove(device). Module statisch verlinkt in 0.1; dynamischer Lader ab 0.2. Requestzustände CREATED→QUEUED→ACTIVE→COMPLETING→DONE; Abbruch konkurriert mit Abschluss über atomaren Zustandsgewinn. Genau ein Abschluss, danach keine Geräte-DMA in freigegebene Puffer.

PCI-Erkennung über ACPI/ECAM mit geprüften Ressourcen; MMIO und DMA nur über Kernelzuweisung. IRQ-Routinen dürfen weder blockieren noch User-Pointer dereferenzieren. P2/P4 zuerst native COM1/APIC/AHCI/xHCI/VirtIO-Treiber nach Meilenstein; Windows-Treiber gehören zum separaten Kompatibilitätsvertrag.

## Kernelabnahme

Ungültige Pointer/Handles/Längen und nicht implementierte Aufrufe; Trennung zweier Prozesse; Endlosschleife neben interaktivem Prozess; wiederholte Spawn/Exit-Zyklen; synchronisierte 4-CPU-Allokation und unmap/shootdown; Queuevoll/Timeout/Abbruch; absichtlich doppelte I/O-Abschlüsse. 10.000 deterministische Lebensdauerzyklen ohne dauerhaftes Wachstum außerhalb dokumentierter Cachebudgets.

Primärreferenzen für CPU und Aufrufkonventionen: [Intel SDM](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html), [Microsoft x64-ABI](https://learn.microsoft.com/en-us/cpp/build/x64-calling-convention?view=msvc-170). Syscalls, Fehlernummern und Portprotokoll sind eigene Projektentscheidungen.
