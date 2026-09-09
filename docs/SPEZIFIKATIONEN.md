# Spezifikationsindex – Baseline 1.0

MUSS ist verbindlich, SOLL erlaubt eine dokumentierte Abweichung, KANN ist optional. Die Baseline wurde auf delegierten Auftrag festgelegt. Historische Vorschläge sind nicht normativ.

## Eigentümer der Spezifikationsbereiche

| ID | Thema | Maßgebliches Dokument |
| --- | --- | --- |
| SPEC-001 | Plattform und Hardwareprofile | [Boot und Plattform](specs/BOOT-UND-PLATTFORM.md) |
| SPEC-002 | Bootvertrag | [Boot und Plattform](specs/BOOT-UND-PLATTFORM.md) |
| SPEC-003 | Diagnose | [Boot und Plattform](specs/BOOT-UND-PLATTFORM.md) |
| SPEC-004 | Build und Tests | [Build und Tests](specs/BUILD-UND-TESTS.md) |
| SPEC-005 | Speicher | [Kernel und ABI](specs/KERNEL-UND-ABI.md) |
| SPEC-006 | CPU/Interrupts/SMP | [Kernel und ABI](specs/KERNEL-UND-ABI.md) |
| SPEC-007 | Prozesse und Scheduler | [Kernel und ABI](specs/KERNEL-UND-ABI.md) |
| SPEC-008 | Systemaufrufe und IPC | [Kernel und ABI](specs/KERNEL-UND-ABI.md) |
| SPEC-009 | Sicherheit | [Kernel und ABI](specs/KERNEL-UND-ABI.md) und [Qualität](QUALITAET-UND-RELEASES.md) |
| SPEC-010 | Programme und SDK | [Kernel und ABI](specs/KERNEL-UND-ABI.md) |
| SPEC-011 | Treibermodell | [Kernel und ABI](specs/KERNEL-UND-ABI.md) und [Windows-Treiber](WINDOWS-TREIBERKOMPATIBILITAET.md) |
| SPEC-012 | Speicherung | [System und Desktop](specs/SYSTEM-UND-DESKTOP.md) |
| SPEC-013 | Grafik und Eingabe | [System und Desktop](specs/SYSTEM-UND-DESKTOP.md) |
| SPEC-014 | Netzwerk | [System und Desktop](specs/SYSTEM-UND-DESKTOP.md) |
| SPEC-015 | Pakete, Installation, Updates | [System und Desktop](specs/SYSTEM-UND-DESKTOP.md) |
| SPEC-016 | Desktop und Basisapps | [Desktop-Abnahme](PRODUKT-DESKTOP-01.md) |
| SPEC-017 | Qualität und Releases | [Qualität und Releases](QUALITAET-UND-RELEASES.md) |
| SPEC-018 | Anwendungskompatibilität | [Entscheidungen](ENTSCHEIDUNGEN.md), O-003 |
| SPEC-019 | Windows-Treiberkompatibilität | [Windows-Treiber](WINDOWS-TREIBERKOMPATIBILITAET.md) |

## Anforderungen und Nachweise

| Anforderung | Festlegung | Nachweis |
| --- | --- | --- |
| R-001/R-002 | Eigener Kernel, echter Bare-Metal-Start | P1 und HW-01 in P2 |
| R-003/R-010 | Professionelle Windows-Alternative als Langfristziel | Versionierte Produkt- und Vergleichsmatrix, kein pauschaler Erfüllungsstatus |
| R-004 | Aktuelle Lieferung Planung/Specs | Dieser Stand enthält keine OS-Implementierung |
| R-005/R-011 | x86-64, VM zuerst, eigene Startkette | VM-BOOT und Bootvertrag |
| R-006/R-013 | Eigene Dienste, Bibliotheken, Desktop und Apps | Komponentenherkunft plus Produktabnahme |
| R-007/R-012 | Externe Werkzeuge und Kernelcode erlaubt | Werkzeugmanifest und Herkunftsregister |
| R-008 | Native Apps zuerst; Windows-Apps später | ELF/libos-Abnahme |
| R-009 | Erster allgemeiner Desktop | PROD-DESKTOP-01 |
| R-014 | Windows-Treiberkompatibilität | WDM-TEST-1 und folgende Profile |

Die früheren R-101 bis R-115 werden durch die konkreten Boot-, Kernel-, Desktop- und Release-Abnahmen dieser Baseline präzisiert. IDs bleiben als historische Referenz erhalten; maßgeblich sind die konkreten Tests.

## Fertigstellungsgrade der Specs

P1 enthält konkret festgelegte Strukturen, Kontrollfluss, Fehlercodes, Artefakte und Tests. Für spätere Etappen sind Architektur und Lieferumfang entschieden. Neue Funktionen benötigen vor ihrem Code eine Detailergänzung innerhalb dieser Entscheidungen, etwa konkrete GUI-Widget-Verträge oder weitere Treiber-DDIs. Das ist reguläre schrittweise Spezifikation, keine Behauptung, ein komplettes Windows-Äquivalent sei bereits bis auf jedes Datenfeld entworfen.

Der nächste Entwickler muss für B-001 bis B-008 keine Grundsatzentscheidung aus der alten Offene-Fragen-Liste mehr treffen.
