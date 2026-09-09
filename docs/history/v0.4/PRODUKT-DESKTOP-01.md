> Historischer Entwurf 0.4; nicht mehr normativ. Aktueller Stand: [Project OS](../../../README.md).

# Produktentwurf: erster nativer Desktop

Kennung: PROD-DESKTOP-01. Version: 0.1. Stand: 09.09.2026.

**Bestätigter Umfang:** allgemeiner Desktop mit Fenstern, Dateimanager, Einstellungen, Terminal und Texteditor. Eigene Implementierung der Oberfläche und zentralen Anwendungen. Professioneller Produktanspruch.

**Status der folgenden Details:** vorgeschlagene Produktspezifikation. Noch keine Freigabe sämtlicher Detailanforderungen und kein behaupteter Implementierungsstand.

## 1. Produktversprechen

Auf einer ausdrücklich unterstützten Plattform kann eine Person das eigene Betriebssystem starten, mehrere native Anwendungen benutzen, lokale Textdateien verwalten und bearbeiten sowie grundlegende Einstellungen dauerhaft speichern.

Dieses erste Produkt ist ein überprüfbarer Schritt zur langfristigen Desktop-Plattform. Es entspricht noch keiner Zusage vollständiger Windows-Funktionsgleichheit.

## 2. Referenzszenario

1. System aus dem vorgesehenen Testimage über den eigenen Bootloader starten.
2. Grafische Sitzung öffnen und Dateimanager sowie Texteditor starten.
3. Verzeichnis und UTF-8-Textdatei anlegen.
4. Datei öffnen, bearbeiten und speichern.
5. Zwischen Anwendungen wechseln und Text über die Zwischenablage übertragen.
6. Die Datei im Terminal auflisten und ihren Inhalt anzeigen.
7. Eine verfügbare Einstellung ändern und speichern.
8. System geordnet neu starten.
9. Datei, Inhalt und Einstellung unverändert wiederfinden.
10. Eine Anwendung gezielt zum Absturz bringen; die übrige Sitzung bleibt bedienbar.

Die einzelnen Schritte werden in Tests mit definierten Vorbedingungen, Daten und erwarteten Ergebnissen überführt.

## 3. Vorgeschlagene Funktionen

| ID | Bereich | Funktionsumfang | Abnahmebeispiel |
| --- | --- | --- | --- |
| D-001 | Start | Eigenes Image, eigener Loader, eigener Kernel und nachvollziehbarer Start der Sitzung | Durchgehender Start im freigegebenen VM-Profil und auf Referenzhardware |
| D-002 | Sitzung | Lokale Sitzung, Anwendungen starten/beenden, geordnetes Herunterfahren und Neustarten | Nach erneutem Start bleiben bestätigte Daten erhalten |
| D-003 | Fenster | Öffnen, schließen, bewegen, Größe ändern, minimieren, maximieren und Fokus wechseln | Mindestens drei Fenster einschließlich überlappender Fenster bedienen |
| D-004 | Eingabe | Maus, Tastatur, definierte Tastenkombinationen, deutsches und englisches Tastaturlayout | Layoutwechsel und Eingabe einschließlich Umlauten und Sonderzeichen |
| D-005 | Textgrundlage | UTF-8-Dateien, Unicode-fähige interne Textschnittstellen, festgelegter erster Zeichenvorrat | Text mit Umlauten und mehrbytekodierten Zeichen verlustfrei speichern |
| D-006 | Zwischenablage | Text zwischen kooperierenden Anwendungen übertragen | Texteditor → Terminal → Texteditor mit unverändertem Inhalt |
| D-007 | Dateimanager | Dateien/Ordner auflisten, erstellen, öffnen, kopieren, verschieben, umbenennen und löschen | Operationen einschließlich Namenskonflikt, fehlender Rechte und vollem Medium |
| D-008 | Texteditor | Neue UTF-8-Datei, öffnen, bearbeiten, rückgängig machen, suchen, speichern, Speichern unter | Ungespeicherte Änderungen beim Schließen erkennbar behandeln; Schreibfehler erhalten den bearbeiteten Text |
| D-009 | Terminal und Shell | Native Programme starten; Verzeichnisse wechseln; Dateien auflisten und lesen; definierte Umleitungen | Programmstart und Ausgabeumleitung in eine danach lesbare Datei |
| D-010 | Einstellungen | Verfügbare Eingabe-, Anzeige- und Sitzungseinstellungen anzeigen und dauerhaft ändern | Unterstützte Änderung überlebt Neustart; ungültiger Wert wird abgelehnt |
| D-011 | App-Fehler | Fehlerzustand einer Anwendung erkennen und deren Ressourcen kontrolliert freigeben | Andere App lässt sich weiter bedienen und kann eine Datei speichern |
| D-012 | Native Plattform | SDK, eigene Laufzeitbibliotheken und dokumentierter Start-/Beendigungsvertrag | Separat gebautes Beispielprogramm funktioniert ohne Einbau in den Kernel |

Die Unterstützung deutscher Texte bedeutet noch keine vollständige Unterstützung aller Schriftsysteme. Textmodell, Zeichenvorrat, Schriften, Eingabemethoden und Barrierefreiheit werden ausdrücklich spezifiziert und langfristig ausgebaut.

## 4. Benötigte Systemfähigkeiten

Das Produkt benötigt mindestens:

- Eigenen Loader und Kernel mit Speicherverwaltung, Prozessen, Threads und geprüften Systemaufrufen.
- Schutz zwischen Anwendungen sowie zwischen Anwendungen und privilegiertem Systemcode.
- Einen festgelegten Treiberpfad für Anzeige, Eingabe und persistente Speicherung.
- Dateisystemvertrag einschließlich Rechte-, Fehler- und Dauerhaftigkeitssemantik.
- Eigene Anwendungslaufzeit, Programmlader und Bibliotheken.
- Ein Kommunikationsprotokoll zwischen Apps, Fensterverwaltung und zuständigen Diensten.
- Definierte Regeln für Konfiguration, Prozessstart, Herunterfahren und Diagnose.

Welche dieser Funktionen im Kernel liegen und welche in User-Prozessen, ist eine Architekturentscheidung. Die Produktanforderungen selbst bleiben unabhängig davon überprüfbar.

## 5. Vorgeschlagene Grenze der ersten Abnahme

Für die erste Desktop-Abnahme werden ein Bildschirm, die ausgewählten Eingabegeräte, ein Systemdatenträger und die festgelegten Plattformprofile vorgesehen.

Browser, Office-Suite, Spiele, Windows-Binärkompatibilität, WLAN/Bluetooth, beschleunigte 3D-Grafik, mehrere Bildschirme, Drucker und umfassende Laptop-Energieverwaltung sind zunächst eigene spätere Arbeitspakete. Diese Abgrenzung ist ein Vorschlag und betrifft nur die erste Produktversion.

Eine frühe Desktop-Demonstration darf aus einem vorbereiteten Image starten. Eine an Anwender verteilte Vorschau benötigt zusätzlich die Installations-, Update- und Wiederherstellungsfunktionen aus P7 des Projektplans.

Windows-Treiberkompatibilität wurde zusätzlich als Plattformziel bestätigt. Welche konkrete Treiberunterstützung bereits für diese erste Desktop-Abnahme erforderlich ist, bleibt eine eigene Umfangsentscheidung. Das Ziel wird von Beginn an in der Architektur berücksichtigt; Windows-Anwendungskompatibilität bleibt separat.

## 6. Vorgeschlagene Fehler- und Qualitätsabnahme

| ID | Situation | Erwartetes Ergebnis |
| --- | --- | --- |
| A-001 | Anwendung greift auf fremden Speicher zu | Fehler bleibt gemäß Schutzmodell auf den Prozess begrenzt; Diagnose ist verfügbar |
| A-002 | Datenträger ist voll | Schreibfehler wird gemeldet; vorher gespeicherte Datei bleibt gemäß Speichervertrag gültig; ungespeicherter Editorinhalt bleibt verfügbar |
| A-003 | Dateiname oder Zielpfad ist ungültig | Definierter Fehler ohne Beschädigung anderer Dateien |
| A-004 | Viele wiederholte App-Starts und -Beendigungen | Ressourcenbilanz kehrt innerhalb definierter Caches/Grenzen zum erwarteten Zustand zurück |
| A-005 | Neustart nach explizit dauerhaft bestätigtem Speichern | Inhalt und relevante Metadaten entsprechen dem Dateisystemvertrag |
| A-006 | Ressourcenmangel bei Fenster- oder Prozessstart | Kontrollierte Ablehnung; bestehende Sitzung bleibt nach festgelegtem Verhalten nutzbar |
| A-007 | Zugriff ohne erforderliche Rechte | Zugriff wird verweigert; keine teilweise ausgeführte privilegierte Änderung |
| A-008 | Anwendung reagiert nicht mehr | Nutzer kann zu anderer Anwendung wechseln und den betroffenen Prozess kontrolliert beenden |

Die Tests sind Nachweise für konkrete Fälle. Sie ersetzen keine umfassende Sicherheitsbewertung oder einen Stabilitätsnachweis über andere Hardware und Lasten.

## 7. Offene Details vor Festschreibung

- Angenommene Kernelarchitektur und Anwendungsschnittstellen.
- Konkrete VM- und Hardwareprofile einschließlich Anzeige- und Eingabegeräten.
- Dateisystem, Rechte- und Dauerhaftigkeitsmodell.
- Gestaltung, Eingabekonventionen und Zugänglichkeitsanforderungen.
- Quantitative Leistungs-, Ressourcen- und Dauertestziele.
- Exakter Umfang des ersten SDK und der eigenen Standardbibliotheken.
