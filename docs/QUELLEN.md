# Technische Primärquellen

Stand der Planung: 09.09.2026. Die Baseline wählt konkrete Versionen als Ausgangspunkt; verlinkte Live-Dokumentationen können sich ändern. Bei Implementierung werden relevante Quell-/Dokumentstände und Prüfsummen in den jeweiligen Manifesten festgehalten.

| Bereich | Primärquelle | Verwendung |
| --- | --- | --- |
| CPU | [Intel SDM](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html) | Paging, Schutz, Interrupts, APIC, SMP |
| Firmware | [UEFI und ACPI](https://uefi.org/specifications) | Loader, Bootdienste, Speicherkarte, ACPI |
| OVMF | [edk2-stable202605](https://github.com/tianocore/edk2/releases/tag/edk2-stable202605) | Gewählter VM-Firmwarequellstand |
| Compiler | [LLVM 22.1.8](https://github.com/llvm/llvm-project/releases/tag/llvmorg-22.1.8) | Gewählte Cross-Toolchain |
| Build | [CMake 3.31.8](https://github.com/Kitware/CMake/releases/tag/v3.31.8) | Gewähltes Buildsystem |
| Ninja | [Ninja 1.12.1](https://github.com/ninja-build/ninja/releases/tag/v1.12.1) | Gewählter Executor |
| Emulator | [QEMU Downloads](https://www.qemu.org/download/) und [Systememulation](https://www.qemu.org/docs/master/system/introduction.html) | QEMU 11.1.1 und Plattformprofile |
| ELF/native ABI | [x86-64 psABI](https://gitlab.com/x86-psABIs/x86-64-ABI) | Native ELF64/SysV-Aufrufe |
| Windows ABI | [Microsoft x64 Calling Convention](https://learn.microsoft.com/en-us/cpp/build/x64-calling-convention?view=msvc-170) | UEFI-/Windows-Treiber-Aufrufkonvention |
| Windows Treiber | [WDM/WDF](https://learn.microsoft.com/en-us/windows-hardware/drivers/wdf/differences-between-wdm-and-kmdf) | Kernel-/Framework-Verträge |
| Frameworkversionen | [Microsoft Versionierung](https://learn.microsoft.com/en-us/windows-hardware/drivers/wdf/framework-library-versioning) | Kompatibilitätsprofile |
| KMDF-Test | [Echo-Sample](https://learn.microsoft.com/en-us/samples/microsoft/windows-driver-samples/kmdf-echo-sample/) | Referenzverhalten |
| Grafiktreiber | [WDDM](https://learn.microsoft.com/en-us/windows-hardware/drivers/display/windows-vista-and-later-display-driver-model-architecture) | Spätere GPU-Abhängigkeiten |
| Externer Treiber | [virtio-win](https://github.com/virtio-win/kvm-guest-drivers-windows/releases) | NetKVM-Laborprofil |
| Netzwerk-DDI | [NDIS-Versionen](https://learn.microsoft.com/en-us/windows-hardware/drivers/network/overview-of-ndis-versions) | Abhängigkeiten von NetKVM |
| Dateisystem | [ext4 und JBD2](https://www.kernel.org/doc/html/latest/filesystems/ext4/index.html) | Disk-/Journalformat |
| TCP | [RFC 9293](https://www.rfc-editor.org/rfc/rfc9293.html) | Eigener TCP-Stack |
| TLS | [RFC 8446](https://www.rfc-editor.org/rfc/rfc8446.html) | TLS 1.3 |
| Signaturen | [RFC 8032](https://www.rfc-editor.org/rfc/rfc8032.html) | Ed25519 |

Primärquellen beschreiben bestehende Standards und Komponenten. Unsere Bootstruktur, Syscalls, Fehlernummern, Produktbudgets und Arbeitspakete sind eigene Designfestlegungen. Die Verwendung einer Quelle besagt nicht, dass fremder Code bereits übernommen oder eine Implementierung geprüft wurde.
