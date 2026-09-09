# Projektplan – Baseline 1.0

## Ziel und aktuelle Lieferung

Professionelle eigene Desktop-Plattform auf x86-64 mit eigenem Bootpfad und Kernel, eigenen Systemdiensten, Standardbibliotheken und Apps sowie gezielter Windows-Treiberkompatibilität. Diese Lieferung schließt die bisher offenen Grundsatzfragen und stellt den ersten Implementierungsauftrag bereit. Kein Implementierungserfolg wird vorweggenommen.

## Meilensteine

| Etappe | Zugeordnetes Ergebnis | Verbindliche Abnahme |
| --- | --- | --- |
| P0 – Spezifikation | Entscheidungen, Bootvertrag, ABI-Grundlagen, Qualität und Backlog | Keine unentschiedene Voraussetzung für B-001 bis B-008; Dokumente konsistent |
| P1 – Start | Eigener UEFI-Loader und minimaler Kernel in VM-BOOT | ELF/Übergabe-/Fehlerprüfungen und 100 wiederholte erfolgreiche Boots |
| P2 – Kernelgrundlage | Speicher, IRQs, Timer, Allokatoren, erster HW-01-Start | Grenz-/Fehlertests; reale Hardwaredaten und Log erfasst |
| P3 – Prozesse | Ring 3, Scheduler, Handles, IPC, Syscalls, libc/libos, SMP | Isolations- und Mehrprozessprüfungen, WDM-TEST-1 nach erfüllten Abhängigkeiten |
| P4 – Basissystem | VFS, ext2, Blocktreiber, Shell, Dienste und SDK | Dauerhafte Dateien nach geordnetem Neustart; Fehlerpfade und extern gebautes Programm |
| P5 – Desktop 0.1 | Software-Compositor, Eingabe, Toolkit und Basisapps | PROD-DESKTOP-01 auf VM-DESKTOP und HW-01; SMP- und App-Fehlertests |
| P6 – Kommunikation | Ethernet/IPv4/TCP/UDP/DNS/DHCP und Netzwerktools | Interoperabilität, Paketfehler, Zeitüberschreitungen und Wiederverbindung |
| P7 – Vorschau 0.2 | ext4/JBD2, Installer, Pakete, A/B-Updates, Recovery und TLS | Leermediuminstallation, Fehlereinspielung, Rückkehr zum vorigen System; Releasekriterien |
| P8 – Ausbau 0.3 | IPv6, Audio, breitere Eingabe-/Geräteprofile und Energieverwaltung | Je freigegebenem Profil vollständige Aufgaben-/Fehler-/Dauertestmatrix |
| Weiterentwicklung | GPU-Beschleunigung, weitere Treiberwelten, Browser, Multimedia und Apps | Eigene versionierte Produktanforderungen und Kompatibilitätsmatrizen |

P1 → P2 → P3 → P4. P5 und P6 bauen auf P4. P7 integriert beide. Windows-Treiberarbeit läuft nach dem zugehörigen T-Profil: WDM-Testvertrag zuerst, KMDF-Echo danach, NetKVM-Laborprofil anschließend. Ein automatischer Erfolg der nächsten Kompatibilitätsstufe wird nicht angenommen.

## Releaseumfang

0.1 ist ein internes Desktop-System mit eigenem Software-Rendering und nativen Basisapps. WDM-TEST-1 ist Pflicht; GPU-/WLAN-/Audio-Windows-Treiber sind kein 0.1-Kriterium. 0.2 darf erst nach den Installations-/Recovery-/Speicherkonsistenzkriterien verteilt werden. 0.3 enthält mindestens ein zusätzliches nachgewiesenes externes Windows-Treiberprofil.

Browser: langfristig Bestandteil des Produktprogramms, keine Eigenentwicklung einer vollständigen Web-Engine in 0.1–0.3. Multimedia, 3D, Druck und vollständige Laptop-Unterstützung werden nach 0.2 in eigenen Produktversionen aufgebaut. Ein eigener Compiler ist außerhalb des aktuellen Programms; Self-Hosting wird zuerst durch Portierung der erlaubten externen Toolchain auf unser OS verfolgt.

## Hardware und Kosten

Zunächst keine Käufe. VM-Profile sind vollständig benannt. HW-01 definiert die später nötigen Geräteeigenschaften, ohne einen unbekannten Rechner als vorhanden auszugeben. Hardwaretests sind ein eigener Auftrag mit inventarisierter Hardware und ausgewähltem Testdatenträger.

## Fortschrittsregeln

Meilensteine statt erfundener Termine. Jede fertiggestellte Aufgabe besitzt einen Quellstand, tatsächliche Testresultate und bekannte Grenzen. Performancewerte sind Produktbudgets; gemessene Werte werden getrennt erfasst. Änderung von Umfang oder Verträgen erfolgt über das Entscheidungsdokument.

Die konkrete Ausführungsreihenfolge steht im [Backlog](BACKLOG.md).
