# Qualität und Releases – Baseline 1.0

## Verbindliche Kriterien

Jeder Implementierungsauftrag benötigt passende Funktions- und Fehlerprüfungen, nachvollziehbare Artefakte und dokumentierte Einschränkungen. Ungelöste Speicherbeschädigung, Datenverlust im zugesagten Modell und Verletzung zugesagter Isolation blockieren Freigaben.

Keine Erfolgsmeldung für nicht implementierte Funktionen. Unbekannte Syscalls/DDIs liefern dokumentierte Fehler; ein Treiber wird nicht mit Dummy-Erfolgen zum scheinbaren Start gebracht. Ein Quellcodeport ist kein Nachweis unveränderter Binärkompatibilität.

## Reifestufen

| Version | Freigabe |
| --- | --- |
| P1 | 100 unabhängige Boots; vollständige positive/negative P1-Suite |
| 0.1 intern | PROD-DESKTOP-01, WDM-TEST-1, 4-CPU-Tests, 8 Stunden Referenzworkload; keine Aussage zur ext2-Stromausfallsicherheit |
| 0.2 Entwicklervorschau | ext4/JBD2-Wiederherstellung, Installer, A/B-Update/Recovery, TLS-Negativtests, 24 Stunden Referenzworkload, KMDF-ECHO-1 |
| 0.3 erweiterte Vorschau | IPv6, festgelegte Audio-/Energie-/Geräteprofile, mindestens ein externes Treiberprofil, 72 Stunden Last-/Leerlaufwechsel |
| stabile Produktversion | Separater benannter Nutzer-/Hardware-/Supportumfang; mindestens 7 Tage Dauertest und keine offenen Freigabeblocker |

Zeitbudgets sind Testspezifikationen. Dauertests werden automatisiert ausgeführt und liefern Logs; sie sind keine Aufforderung, einen Gesprächsagenten tagelang blockierend warten zu lassen.

## Ressourcen und Leistung

Kernel-/Desktopbudgets stehen in PROD-DESKTOP-01. P1 erreicht P1_READY innerhalb 30 s im festgelegten QEMU-Profil. 10.000 Prozess-/Handle-/Allokationszyklen dürfen kein monotones unbeschränktes Wachstum zeigen. Restwachstum nach Cache-Warmup höchstens 1 MiB pro Test, sonst Untersuchung statt pauschaler Freigabe.

Performancevergleich nur mit identischem Workload und dokumentierter Umgebung. Kein Vergleich von QEMU-TCG mit nativem Windows als Leistungsbeleg.

## Sicherheitsmodell

Nicht privilegierte Apps und externe Eingaben sind nicht vertrauenswürdig. Parser, Paketpfade, Syscall-Argumente und IPC-Nachrichten benötigen Grenz-, Größen- und Lebensdauerprüfungen. W^X, Nullseiten-/Guard-Schutz und getrennte Adressräume ab User-Mode-Start. Stack-Canaries ab P2, ASLR/PIE und sichere Boot-/Updateketten vor verteilbarer 0.2-Freigabe.

Privilegierte Treiber liegen in der Vertrauensbasis; die Modulgrenze garantiert keine Isolation. Das Laden erfordert Systemverwaltung, geprüftes Paketprofil und Diagnose. Treiberfehler werden separat von App-Isolationstests bewertet.

Kryptografie verwendet Standards und unabhängige bekannte Testvektoren. Eigene Implementierungen werden vor sicherheitsrelevanter Produktfreigabe überprüft; keine Sicherheitsbehauptung aus bloßer Algorithmusauswahl.

## Builds und Herkunft

Quellcommit, Toolchain-/Firmware-Versionen und Prüfsummen gehören in den Buildnachweis. Übernommener Kernelcode enthält Herkunft, Version und zugehörigen Lizenztext. Eigene Dienste/Standardbibliotheken werden als eigene Implementierungen geführt. Das private Repository erteilt in dieser Baseline keine öffentliche Lizenz für den eigenen Projektcode.

## Updates und Daten

Vor 0.2 müssen Stromausfalltests an Journal-/Flush-/Updategrenzen und Wiederanlauf auf den letzten bestätigten Systemstand bestehen. Recovery umfasst Programmdateien und kompatible Datenformate. Testmatrix mindestens: Abbruch vor Downloadende, vor/nach Slotprüfung, vor/nach pending-Markierung, drei Fehlstarts und erfolgreicher Health-Commit.

## Veröffentlichung und Betrieb

Öffentliche Veröffentlichung, neue Lizenzfreigabe, Käufe und dauerhafte externe Dienste sind nicht Teil des aktuellen Uploadauftrags. Für spätere Releases benennt der Projektinhaber Support-/Freigabeverantwortliche. Im jetzigen privaten Projekt ist er Entscheidungseigentümer; es werden keine erfundenen Teammitglieder oder Kapazitäten eingetragen.

Tests und Entscheidungen können eine Weiterentwicklung begründen. Sie ersetzen keinen noch nicht erbrachten Nachweis.
