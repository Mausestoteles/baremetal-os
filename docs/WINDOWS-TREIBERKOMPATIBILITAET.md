# SPEC-019 – Windows-Treiberkompatibilität

Baseline 1.0. Ziel ist unveränderte x64-Binärausführung für ausdrücklich definierte Profile. Der eigene Kernel bleibt die Plattform. Allgemeine Windows-App-Unterstützung ist davon getrennt.

## Architektur und Referenz

Referenz: Windows 11 24H2 x64 und WDK/SDK 10.0.26100. Der Windows-DDI-Bereich läuft zunächst im privilegierten Kernelraum. Er enthält PE32+-Lader, ABI-Fassadenobjekte und eigene Implementierungen benötigter NT/WDM-Verträge; später KMDF und Gerätekategorien. Eine Modulgrenze schützt nicht vor beliebigen Treiberfehlern.

Native Kernelobjekte sind kein öffentliches Windows-Layout. Öffentliche Treiberstrukturen werden nach dem festgelegten WDK-Layout abgebildet und durch sizeof/offsetof-Referenztests geprüft. Das genutzte WDK-Patchlevel und die tatsächlich kompilierten Testbinärdateien werden beim entsprechenden Implementierungsauftrag mit Hashes eingefroren.

Microsoft beschreibt die direkte Bindung von WDM-Treibern an Systemroutinen/-strukturen und die zusätzlichen Framework-Verträge von WDF: [WDM und WDF](https://learn.microsoft.com/en-us/windows-hardware/drivers/wdf/differences-between-wdm-and-kmdf). Die daraus folgende Architektur ist eine eigene Projektentscheidung.

## WDM-TEST-1 – Pflicht für 0.1

Ein eigenes, unter dem Referenz-WDK gebautes Testpaket oswdmtest.sys wird ohne Neukompilierung unter unserem OS ausgeführt. Softwaregerät ohne Hardware/DMA; x64, WDM, keine KMDF-/NDIS-/WDDM-Abhängigkeit. Testsignatur und Paketintegrität werden in der ausdrücklich eingerichteten Entwicklungsumgebung behandelt; keine Änderungen an Windows-Sicherheitseinstellungen durch normale OS-Buildwerkzeuge.

DDI-Mindestumfang:

| Vertrag | Benötigte Funktionen / Verhalten |
| --- | --- |
| Lebenszyklus | DriverEntry, DriverUnload, DRIVER_OBJECT-Dispatchtabelle |
| Geräte | IoCreateDevice, IoDeleteDevice, IoCreateSymbolicLink, IoDeleteSymbolicLink |
| I/O | IRP_MJ_CREATE, CLOSE, DEVICE_CONTROL; IoGetCurrentIrpStackLocation-Vertrag; IoCompleteRequest / IofCompleteRequest entsprechend tatsächlichem WDK-Import |
| Speicher | ExAllocatePool2, ExFreePoolWithTag; geprüfte Größen/Tags, nicht ausgelagerter Speicher |
| Synchronisation | KeInitializeEvent, KeSetEvent, KeResetEvent, KeWaitForSingleObject |
| Kontext/Zeit | KeGetCurrentIrql, KeQueryPerformanceCounter |
| Diagnose | DbgPrintEx mit begrenztem Format-/Pufferumfang |
| Laufzeit | memcpy/memmove/memset/memcmp und nachgewiesene Compiler-Hilfsroutinen |

Der konkrete Importabschluss muss vollständig dokumentiert sein. Ein zusätzlich vom Compiler erzeugter Import wird sichtbar in den Vertrag aufgenommen, durch passende Compile-Konfiguration vermieden oder führt zu klarer Ablehnung. Niemals unbekannte Funktionen mit Dummy-Erfolg ersetzen.

Test-IOCTLs: ECHO_BUFFERED, ALLOC_FREE, EVENT_SIGNAL_WAIT, QUERY_CONTEXT, FAIL_REQUEST. METHOD_BUFFERED zuerst. Eingabegrößen 0,1,4096,65536 und oberhalb des Limits; Ausgabegrößen kleiner/gleich/größer als benötigt. Größenlimit 65536 Bytes, Overflows vor Allokation abweisen.

Prüfungen: 100 Lade-/Entladezyklen, 10.000 Requests, parallele Requests aus vier Threads, Speicherfehler, Warte-Timeout, abgeschnittener PE, fehlender Import, ungültige Relokation, unerwarteter Abschluss. Referenzausführung auf Windows und auf unserem OS vergleicht beobachtete Status-/Inhalts-/Lebensdauerverträge; Windows-Statuswerte werden nicht durch native OS-Fehlernummern ersetzt.

## WDM-IO-2 – Asynchrones I/O

Vor breiterem Gerätebetrieb: IRP-Stapel, Pending/Completion, Cancel-Routinen und genau-einmaliger Abschluss; DPC-/Timer-/Spinlock-Verträge und Poolzugriff nach IRQL. THREAD-, DPC- und IRQ-Kontexte unseres Kernels werden ausdrücklich auf die verwendeten Windows-IRQL-Regeln abgebildet.

Zusätzliche IRPs, MDL-/DMA-Verträge, PnP/Power und Bustreiber werden nur zusammen mit einem konkreten Gerätekontext implementiert und abgenommen. Erkennung → Ressourcen → Start → I/O → Stop/Remove ist ein zustandsbehafteter Vertrag, nicht eine Sammlung gleichnamiger Funktionen.

## KMDF-ECHO-1 – Pflicht für 0.2

Ziel: KMDF 1.33, Windows-x64-Referenz, ein Echo-Treiber nach dem [offiziellen KMDF-Echo-Sample](https://learn.microsoft.com/en-us/samples/microsoft/windows-driver-samples/kmdf-echo-sample/). Ein unter Windows gebautes Referenzbinär bleibt für den Vergleich unverändert.

Profilumfang: Framework-Bindung/Version, Driver-/Device-/Queue-Objekte, sequenzielle Read/Write-Queue, Request-/Memory-Zugriff, Callback-Kontext, Cleanup und Reference-Lifecycle. Die Frameworkimplementierung wird als Kernel-Kompatibilitätskomponente behandelt; erlaubte Codeübernahme benötigt Herkunft und zugehörige Lizenzbedingungen.

Der dokumentierte Umfang ist ein kompatibles Teilprofil, keine Behauptung vollständiger KMDF-1.33-Abdeckung. Benötigt das ausgewählte Echo-Binär mehr als diese Menge, muss die vollständige Abhängigkeit vor einem Erfolgsurteil implementiert oder das Profil ausdrücklich neu versioniert werden.

Framework-Versionen sind Teil des tatsächlichen Ladevertrags: [Microsoft Framework-Versionierung](https://learn.microsoft.com/en-us/windows-hardware/drivers/wdf/framework-library-versioning).

## NETKVM-LAB-1 – Externes Geräteprofil für 0.3

Erstes ausgewähltes externes Treiberprojekt: NetKVM aus virtio-win, Quellrelease mm320 als Ausgangsreferenz, Windows-11-amd64-Paket. Gerät: QEMU VirtIO-Net PCI. Quelle: [virtio-win-Releases](https://github.com/virtio-win/kvm-guest-drivers-windows/releases).

Vor Implementierung erfasst B-060 das zugehörige tatsächliche Binärpaket, Hashes, INF, Imports, NDIS-Version und zusätzliche Abhängigkeiten. Diese Fakten werden nicht aus dem Projektnamen geraten. mm320 ist eine Laborreferenz; daraus wird keine Stabilitätsfreigabe abgeleitet. Ist kein passendes Binärpaket beschaffbar, bleibt dieses Profil nachweislich blockiert; Ersatz erfolgt per dokumentierter Profilrevision.

Abnahme: unverändertes externes Binär wird geladen, steuert das virtuelle Gerät, sendet/empfängt über unseren Netzwerkpfad, behandelt Last/Abbruch/Neustart und besteht die definierte Funktionsmatrix. Ein Windows-Binär für virtuelle Hardware beweist nicht automatisch echte PCIe-/GPU-/WLAN-Kompatibilität.

NDIS wird erst entsprechend dem erfassten Treiberumfang ergänzt. Die Versionsabhängigkeit wird anhand der [NDIS-Referenz](https://learn.microsoft.com/en-us/windows-hardware/drivers/network/overview-of-ndis-versions) dokumentiert.

## Weitere Kategorien

WDDM-Grafik, Audio, WLAN, Storage und UMDF sind getrennte Folgeprofile nach 0.3, keine Zusage pauschaler Unterstützung. WDDM verlangt zusammenwirkende User-/Kernelkomponenten einschließlich Grafik-Scheduling und Videospeicherverwaltung: [WDDM-Architektur](https://learn.microsoft.com/en-us/windows-hardware/drivers/display/windows-vista-and-later-display-driver-model-architecture).

Herstellerspezifische Installationsprogramme und komplette Windows-Benutzeroberflächen werden nicht allgemein ausgeführt. Pakete werden über einen eigenen Import-/Gerätezuordnungsweg aufgenommen. Erforderliche zusätzliche Dienste werden als explizite Profilabhängigkeit erfasst.

## Kompatibilitätsmatrix

Jeder Eintrag enthält Profilversion, Treiber-/Firmware-Hashes, Gerät/IDs, Referenz-Windows/WDK, getestete Funktionen, fehlende Funktionen, Fehlerresultate und verwendeten OS-Commit. Status: geplant, implementiert, Referenzvergleich bestanden, Geräteprofil bestanden, freigegeben. Nur der jeweilige belegte Status darf gemeldet werden.

Alle Profile sind in diesem Repository-Stand geplant. Keine Treiberbinärdatei wurde geladen oder getestet.
