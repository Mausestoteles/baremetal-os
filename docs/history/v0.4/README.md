> Historischer Entwurf 0.4; nicht mehr normativ. Aktueller Stand: [Project OS](../../../README.md).

# Eigenes Betriebssystem

Langfristiges Projekt eines eigenständigen Betriebssystems mit eigenem Kernel, direktem Betrieb auf echter Hardware und einem angestrebten Funktionsumfang auf Windows-Niveau.

**Aktueller Stand: Planung und Spezifikation, Version 0.4, 09.09.2026.**

Es gibt noch keine Implementierung. Der Projektname ist offen. „Windows-Level“ ist zunächst die Zielvision; konkrete Funktions-, Qualitäts- und Kompatibilitätsanforderungen werden gemeinsam definiert.

## Planungsunterlagen

| Dokument | Inhalt |
| --- | --- |
| [Projektplan](PROJEKTPLAN.md) | Zielbild, Umfang, Etappen, Abnahmekriterien und Arbeitsweise |
| [Entscheidungen](ENTSCHEIDUNGEN.md) | Bestätigte Vorgaben, offene Grundsatzfragen und Vorschläge |
| [Spezifikationen](SPEZIFIKATIONEN.md) | Anforderungskatalog, Systembereiche und Aufbau der technischen Specs |
| [Architekturoptionen](ARCHITEKTUROPTIONEN.md) | Vergleich der Kernelkonzepte und Vorschlag für den ersten Systemaufbau |
| [Erste Desktop-Version](PRODUKT-DESKTOP-01.md) | Konkreter Produktumfang, Nutzungsszenarien und vorgeschlagene Abnahme |
| [Qualität und Releases](QUALITAET-UND-RELEASES.md) | Produktreife, Prüfung, Kompatibilitätsregeln und langfristige Wartbarkeit |
| [Windows-Treiberkompatibilität](WINDOWS-TREIBERKOMPATIBILITAET.md) | Kompatibilitätsziel, Auswirkungen auf den Kernel und gestufter Nachweis |

## Bestätigte Basis

- Desktop-PC auf x86-64, zunächst in einer VM, danach auf ausgewählter echter Hardware.
- Vorhandene Hardware und Firmware; eigener Bootloader von Beginn an.
- Eigener Kernel, dabei dürfen Bibliotheken und Code übernommen werden.
- Sämtliche Systemdienste und Standardbibliotheken selbst entwickeln; eigene Grafikoberfläche und zentrale Anwendungen.
- Externe Entwicklungswerkzeuge sind erlaubt.
- Vergleichbarer Komfort und Funktionsumfang mit nativen Apps; Windows-Apps werden später gesondert bewertet.
- Kompatibilität zu Windows-Treibern ist ein ausdrückliches Projektziel; genaue Treibermodelle und Versionen werden spezifiziert.
- Erstes nutzbares Produkt: allgemeiner Desktop mit Fenstern, Dateimanager, Einstellungen, Terminal und Texteditor.
- Anspruch: professionelle, ernstzunehmende Konkurrenz zu Windows als langfristiges Produktziel.

## Verbindlichkeit

- **Bestätigt:** ausdrücklich aus den Vorgaben des Projektinhabers übernommen oder später gemeinsam entschieden.
- **Vorschlag:** Diskussionsgrundlage; noch keine Architekturentscheidung.
- **Offen:** noch zu klären; keine stillschweigende Festlegung.

Ein ausgearbeiteter Vorschlag ist keine bestätigte Vorgabe. Beschlossene Änderungen werden mit Datum und Begründung im Entscheidungsdokument festgehalten.

## Nächster gemeinsamer Schritt

Zuerst die Auswirkungen der Windows-Treiberkompatibilität auf Kernel, I/O und Synchronisation konkretisieren. Darauf aufbauend Kernelarchitektur, Programmiersprachen und Plattformdetails festlegen sowie Bootvertrag, Diagnose und Build-/Testprofil umsetzungsreif spezifizieren. Ressourcen wurden nicht beziffert; deshalb enthält die Planung keine Termin- oder Kapazitätszusage.

Die derzeitige Beauftragung umfasst ausschließlich Planung und Spezifikationen. Implementierung beginnt erst auf spätere Anweisung des Projektinhabers.
