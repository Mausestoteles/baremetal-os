# Gewählte Systemarchitektur

Baseline 1.0. Der Dateiname bleibt zur Linkkompatibilität erhalten; die Architekturentscheidung ist jetzt getroffen.

## Verantwortlichkeiten

| Schutzbereich | Komponenten |
| --- | --- |
| Firmware / Loader | Vorhandene UEFI-Firmware; eigener Loader bis ExitBootServices |
| Kernel | CPU/MMU, Interrupts, physischer/virtueller Speicher, Threads/Scheduler, Handles/IPC, VFS, Dateisystem- und Geräte-I/O, Netzwerkmechanismen |
| Kernelmodule | Statisch registrierte native Treiber; später versionierte dynamische Module; Windows-DDI-/WDM-/KMDF-Kompatibilitätsmodule |
| Privilegierte User-Dienste | init, serviced, deviced, sessiond, configd, packaged, updated, netcfgd |
| Grafische Sitzung | displayd als Compositor, inputd für Layout/Verteilung, eigener Toolkit-Client |
| Anwendungen | Terminal/Shell, Dateimanager, Texteditor, Einstellungen, Systemübersicht |
| Bibliotheken | Eigene libos, libc und libui; keine direkte Abhängigkeit von internen Kernelstrukturen |

## Kommunikationsregeln

Apps rufen libos/libc auf. libos verwendet die native Syscall-ABI. Kommunikation mit Diensten erfolgt über Kernel-Ports; große Grafikpuffer über explizit freigegebene Shared-Memory-Handles. Das Recht zum Übergeben von Handles ist separat vom Nutzungsrecht.

Kernelmodule benutzen versionierte interne C-Schnittstellen. Die Windows-Treiberumgebung exportiert ausschließlich ihre dokumentierten DDIs; sie übersetzt auf native Mechanismen und hält erforderliche Windows-Fassadenobjekte selbst. Direkte gemeinsame Strukturlayouts zwischen nativen Apps und Treibern sind unzulässig.

## Abhängigkeiten und Reihenfolge

Hardware → eigener Loader → Kernelmechanismen → native User-Laufzeit → Dienste → Desktop/Apps. Windows-DDI-Verträge fließen schon in Kernel-Synchronisation und I/O ein; die Binärausführung beginnt nach den zugehörigen Mechanismen. Der eigene erste Bootpfad hängt nicht von funktionierender Windows-Kompatibilität ab.

## Bewusst akzeptierte Kosten

C erfordert zusätzliche Arbeit zur Vermeidung von Speicherfehlern. Privilegierte Treiber können das Gesamtsystem beschädigen. Der Kernel wird deshalb nicht als fehlertolerant gegenüber beliebigen Fremdtreibern bezeichnet. Eine spätere isolierte Treiberumgebung wäre eine neue Architekturentscheidung mit eigener Abnahme.

Mikrokernel, C++ und Rust bleiben dokumentierte Alternativen, werden aber für diese Baseline nicht parallel implementiert. Details stehen in [Entscheidungen](ENTSCHEIDUNGEN.md), [Kernel und ABI](specs/KERNEL-UND-ABI.md) und [Windows-Treiber](WINDOWS-TREIBERKOMPATIBILITAET.md).
