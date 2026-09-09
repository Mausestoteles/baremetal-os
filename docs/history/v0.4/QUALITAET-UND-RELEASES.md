> Historischer Entwurf 0.4; nicht mehr normativ. Aktueller Stand: [Project OS](../../../README.md).

# Qualität, Schnittstellen und Releases

Version: 0.1. Stand: 09.09.2026. Vorgeschlagene technische Produktregeln.

Der professionelle Produktanspruch ist bestätigt. Die folgenden Regeln konkretisieren ihn zur Diskussion; konkrete Messwerte, Rollen und Releasezusagen sind noch zu vereinbaren.

## 1. Produktreife getrennt von Funktionsfortschritt

| Reifestufe | Erwarteter Nachweis |
| --- | --- |
| Interner Prototyp | Definierter Zweck, reproduzierbarer Build, Diagnose und dokumentierte Grenzen |
| Integrierte Desktop-Demonstration | Durchgehende Nutzungsszenarien aus PROD-DESKTOP-01; bekannte Plattform und Fehlergrenzen |
| Entwicklervorschau | Installations-/Update-/Recovery-Pfad, veröffentlichtes SDK mit Versionsregeln, nachvollziehbare Hardwareunterstützung und Fehlerliste |
| Beta | Festgelegter Funktionsumfang; quantitative Testziele, Migrations- und Wiederherstellungsprüfungen auf allen zugesagten Profilen |
| Stabile Version | Vereinbarte Releasekriterien erfüllt, keine offenen Freigabeblocker, dokumentierte Support- und Sicherheitsprozesse |

Ein erreichter Funktionsmeilenstein ist nicht automatisch eine stabile Veröffentlichung. Die Bezeichnungen legen keine Termine fest.

## 2. Architekturqualität

- Jeder Systembereich hat eine klare Verantwortung und dokumentierte erlaubte Abhängigkeiten.
- Öffentliche Schnittstellen besitzen definierte Parameter, Besitzregeln, Fehlerfälle und Versionierung.
- Architekturentscheidungen nennen Alternativen und überprüfbare Gründe.
- Ein kleines eigenes Referenzprogramm verwendet das SDK außerhalb des Betriebssystem-Quellbaums.
- Die Kompatibilität öffentlicher Schnittstellen wird ab einem ausdrücklich beschlossenen Stabilitätspunkt zugesagt.
- Interne Kernelmodule werden von öffentlichen Treiberschnittstellen und der App-ABI unterschieden.

Die genaue Umsetzung hängt von der Kernelentscheidung ab. „Modular“ beschreibt zunächst eine Code- und Schnittstellenstruktur, keine garantierte Isolation.

## 3. Sicherheitsarbeit von Beginn an

Vor dem ersten User-Programm wird ein Bedrohungsmodell beschrieben. Es umfasst mindestens fehlerhafte oder bösartige Anwendungen, unzulässige Ressourcenzugriffe sowie manipulierte Eingaben an privilegierten Schnittstellen. Später werden Netzwerk, externe Datenträger, Pakete und Updatequellen ergänzt.

Jede relevante Spec beschreibt dazu:

- Welche Daten und Komponenten vertrauenswürdig sind.
- Wer eine Operation ausführen darf und wo dies geprüft wird.
- Wie Speicher und andere Ressourcen zwischen Schutzbereichen übergeben werden.
- Was bei ungültigen Eingaben, Ressourcenmangel oder Abbruch passiert.
- Welche Fehlertests und Überprüfungen den jeweiligen Vertrag prüfen.

Bootvertrauen, Kryptografie, Entropie, Schlüsselverwaltung und Updateverifikation sind eigene technische Themen. Anforderungen daran werden vor den jeweils abhängigen Funktionen festgeschrieben. Die aktuelle Phase behauptet keine Sicherheitsgarantie.

## 4. Prüfstrategie

| Ebene | Geeignete Prüfungen |
| --- | --- |
| Hardwareunabhängige Logik | Hostbasierte Tests für Parser, Datenstrukturen und reine Algorithmen |
| Kernel | Boot-, Speicher-, Syscall-, Synchronisations-, Prozess- und Isolationsprüfungen im festgelegten VM-Profil |
| Eingabeschnittstellen | Grenzfälle und später Fuzzing für Dateien, Programme, Pakete, Nachrichten und Protokolle |
| Fehlerbehandlung | Gezielter Ressourcenmangel, Abbruch von Operationen, I/O-Fehler und definierte Unterbrechung von Schreib-/Updatevorgängen |
| Systemintegration | Durchgehende Desktop-, Installations-, Update- und Wiederherstellungsszenarien |
| Echte Hardware | Start, Interrupts, Geräte, Last und später Energiezustände auf dokumentierten Profilen |
| Leistung und Dauerbetrieb | Vergleichbare Workloads, Ressourcenverlauf, Antwortzeiten und Durchsatz unter festgelegten Bedingungen |

Tests werden mit Anforderungen verknüpft. Ein bestandenes VM-Szenario gilt nicht automatisch für echte Geräte. Für SMP, Dateisystemkonsistenz und asynchrone I/O sind eigene konkurrierende und fehlerhafte Ablaufvarianten vorzusehen.

## 5. Nachvollziehbare Builds und Abhängigkeiten

Jeder freizugebende Stand dokumentiert Quellstand, Werkzeuge, Versionen, Zielprofil, Buildoptionen und enthaltene Fremdbestandteile. Zunächst wird ein nachvollziehbar wiederholbarer Build angestrebt. Bitidentische Builds werden als eigener, überprüfbarer Qualitätsanspruch behandelt.

Das Komponentenverzeichnis unterscheidet selbst entwickelten Code, Entwicklungswerkzeuge, mitgelieferte Laufzeitbestandteile, Firmware und sonstige Assets. Die Erlaubnis eines Compilers wird nicht als pauschale Erlaubnis sämtlicher mitgelieferter Bibliotheken ausgelegt.

Für den Kernel ist die Übernahme von Bibliotheken und Code ausdrücklich erlaubt. Die vollständige eigene Entwicklung von Systemdiensten und Standardbibliotheken bleibt bestätigt. Bestehende Windows-Treiber werden als eigene externe Komponentenkategorie mit Version, Geräteprofil und Kompatibilitätsnachweis geführt.

## 6. Updates, Daten und Kompatibilität

Vor einer verteilbaren Vorschau müssen folgende Verträge existieren:

- Format und Identität von Systemversionen und Softwarepaketen.
- Vertrauens- und Prüfregeln für Updates.
- Verhalten bei unterbrochenem Download, vollem Datenträger und abgebrochener Installation.
- Bootbarer Wiederherstellungsweg und Rückkehr zu einem funktionierenden Systemstand.
- Umgang mit geänderten Konfigurations- und Dateiformaten.
- Grenzen der Rückkehr zu älteren Versionen, wenn Daten bereits migriert wurden.
- Versions- und Kompatibilitätsregeln für native Anwendungen und Treiber.

Ein Rollback von Programmdateien allein reicht nicht als Datenmigrationsstrategie. Beide Verträge werden gemeinsam getestet.

## 7. Releaseentscheidung

Ein Releasebericht enthält die zugehörigen Anforderungen, bestandene Prüfungen, tatsächliche Plattformmatrix, bekannte Einschränkungen und offene Fehler. Fehler mit Datenverlust, Verletzung zugesagter Isolation oder unbrauchbarem Installations-/Recovery-Pfad gelten für eine stabile Freigabe als vorgeschlagene Blocker.

Rollen für technische Prüfung, Freigabe, Wartung und Sicherheitsmeldungen werden vor einer öffentlichen Veröffentlichung benannt. Es werden derzeit weder ein Team noch personelle Kapazitäten vorausgesetzt.

## 8. Langfristiger Wettbewerbsnachweis

Der Vergleich mit Windows wird pro Produktbereich konkretisiert: benannte Nutzeraufgabe, Referenzhardware, Vergleichsversion, gleiche Randbedingungen und messbare Ergebnisse. Kandidaten sind Start und Anmeldung, App-Start, Dateiarbeit, Reaktionszeit unter Last, Energieverbrauch, Installation und Wiederherstellung.

Für Anwendungsauswahl und Geräteunterstützung werden eigene Matrizen geführt. Ein Vorteil in einem Benchmark ersetzt keine fehlenden Nutzerfunktionen. Die erste Desktop-Version liefert den Anfang dieser Vergleichsbasis; eine pauschale Wettbewerbsfähigkeit wird daraus noch nicht abgeleitet.
