# Systemdienste, Speicherung und Desktop

Baseline 1.0. Die folgenden Entscheidungen bestimmen die Umsetzung ab P4.

## Pfade und VFS

Interne Pfade sind UTF-8, maximal 4096 Bytes, Komponenten maximal 255 Bytes. NUL und / in Komponenten verboten. Vergleich im Kernel bytegenau und case-sensitive; keine stillschweigende Unicode-Normalisierung. Benutzeroberflächen stellen Namen unverändert dar und warnen bei visuell verwechselbaren Konflikten.

Verzeichnisse: /System, /System/Drivers, /Applications, /Users/<name>, /Config, /Devices, /Volumes, /Temp, /Logs. /System wird in Vorschau-Releases nur durch den Updater geändert. /Temp ist nicht dauerhaft zugesagt. Gerätezugriff erfolgt über berechtigte Handles.

VFS-Operationen: lookup/open/read/write/stat/readdir/create/mkdir/rename/unlink/sync. Namensauflösung und Rechteprüfung besitzen dokumentierte Sperr-/Lebensdauerregeln. Symlinks erst ab 0.2; dann maximal 40 Auflösungen. rename innerhalb eines Dateisystems ist im laufenden System atomar; Dateisystemgrenzen liefern ENOTSUP. Dauerhaftigkeit erfordert Datei- und gegebenenfalls Verzeichnis-sync.

## Dateisysteme

0.1: eigene ext2-Revision-1-Implementierung, Blockgröße 4096, Inodegröße 128 oder 256 beim Lesen; Formatter verwendet 256. Zulässige Features: FILETYPE; sparse_super und large_file im dokumentierten Umfang. Nicht unterstützte incompat-/ro_compat-Flags führen zum Abbruch beziehungsweise explizit schreibgeschütztem Mount. Keine ext4-Extents oder Journaldaten als ext2 interpretieren.

Schreibumfang: normale Dateien/Verzeichnisse, direkte/einfach/doppelt indirekte Blöcke, Größenlimit 1 GiB pro Datei im ersten Release, Dateirechte/UID/GID, Zeitstempel. Freie Blöcke/Inodes über Bitmap; Allokation und Größenarithmetik geprüft. Editor speichert zunächst in eine neue Datei und ersetzt nach erfolgreichem sync; voller Datenträger darf den bisherigen Inhalt nicht vorzeitig überschreiben.

ext2 bietet in dieser Phase keine zugesagte Absturzkonsistenz. Dirty-Mounts sind read-only, bis das eigene Offline-Prüfwerkzeug Konsistenz bestätigt oder ein frisches Testimage erstellt wurde. Ein normaler geordneter Neustart ist Bestandteil der 0.1-Abnahme; Stromausfallfreigabe erst mit dem nächsten Pfad.

0.2: eigene ext4-Implementierung mit Extents und JBD2-Metadatenjournal; data=ordered. Baseline-Features: extents, filetype, sparse_super, large_file und Journal; 64bit, metadata_csum, inline_data, encryption, bigalloc und weitere Erweiterungen werden anfänglich nicht im Formatter gesetzt und müssen beim Mount ausdrücklich erkannt/abgewiesen werden. Das System darf kein Feature still überlesen. Die genauen Diskstrukturen folgen den [ext4-/JBD2-Formatreferenzen](https://www.kernel.org/doc/html/latest/filesystems/ext4/index.html).

Write/flush/barrier-Semantik wird bis zum Blockgerät durchgereicht. Wiederherstellung replayt ausschließlich vollständige gültige Transaktionen. Alte Daten bleiben bis Abschluss von Ersatzschreibvorgängen erreichbar. Dateisystemfehler lösen schreibgeschützten Betrieb und Diagnose aus.

Umstieg ext2→ext4: offline Kopie auf ein frisch formatiertes ext4-Volume mit Inhaltsprüfung, kein automatisches In-place-Upgrade. FAT32 bleibt Boot-/Austauschformat ohne zugesagte Unix-Rechte. Keine dauerhaft sensible Benutzerkonfiguration auf der ESP.

## Systemdienste

init ist PID 1. Es startet serviced; serviced startet deviced, configd, sessiond, danach inputd/displayd und die Sitzung. netcfgd, packaged und updated kommen mit den entsprechenden Meilensteinen hinzu. Dienste sind eigene Programme.

Dienstmanifest: UTF-8-Datei mit format_version, name, executable, arguments, dependencies, identity, capabilities, restart_policy. Zunächst eigene minimale key=value-Syntax mit eindeutigen Schlüsseln, Arrays als wiederholte einzeln geprüfte argument/dependency-Zeilen; keine Ausführung von Konfigurationstext.

Zustände STOPPED→STARTING→RUNNING→STOPPING oder FAILED. Abhängigkeiten als DAG; Zyklen sind Konfigurationsfehler. Höchstens 3 automatische Neustarts in 60 Sekunden, danach FAILED und Diagnose. init selbst wird bei kritischem internen Fehler nicht in einer stillen Endlosschleife neu gestartet.

Konfiguration über configd: versionierte Schlüssel/Werte und atomarer Ersatz der Konfigurationsdatei mit sync. Maximal 64 KiB pro Wert und 4 MiB pro Dokument. Keine frei interpretierte Shell. Geheimnisse gehören nicht in allgemeine Logs.

## Grafischer Pfad

Kernel liefert Display-/Input-Gerätehandles an autorisierte Dienste. displayd läuft als User-Prozess, besitzt das Display und ist eigener CPU-Compositor. Apps zeichnen über die eigene libui in geteilte Puffer; direkter GOP-/Framebufferzugriff ist ihnen nicht erlaubt.

Pixel: XRGB8888, Little Endian, Stride in Bytes. Flächen werden auf höchstens 4096×4096 und 64 MiB pro Puffer begrenzt. Pro Oberfläche zwei Puffer, Eigentum wechselt durch present(sequence,buffer,dirty_rects). Der Compositor bestätigt release(sequence), bevor der Client den betreffenden Puffer erneut beschreibt. Bei App-Absturz werden ihre Oberflächen entfernt.

UI-Protokoll über native Ports, Major 1: HELLO, CREATE_SURFACE, RESIZE_REQUEST, RESIZE_ACK, PRESENT, BUFFER_RELEASE, INPUT, FOCUS, CLOSE_REQUEST, DESTROY. Alle Nachrichten besitzen den Kernel-IPC-Kopf und versionierte Payloads. Surface-ID enthält Generation; unbekannte/stale IDs liefern Fehler. Rechte und Flächengrößen werden serverseitig geprüft.

Fensteraktionen: bewegen, skalieren, minimieren, maximieren, schließen, Z-Reihenfolge und Tastaturfokus. Maus-/Tastaturevents gehen nur an den fokussierten beziehungsweise vom Pointer getroffenen berechtigten Client; globale Eingabeüberwachung nur privilegiert. Kein systemweites stilles Input-Abonnement normaler Apps.

inputd verarbeitet rohe Keycodes separat von Text. Layouts de-DE und en-US; Keydown/keyup/Modifiers plus Unicode-Textevents. Text-Clipboard mit explizitem Eigentümer, maximal 16 MiB; kein Dateitransfer im ersten Protokoll.

libui: eigene Layouts, Buttons, Labels, Textfelder, Menüs, Listen/Scrollbereiche und Dialoge. Erstes Schriftprofil: eigener Rasterizer und selbst erstellter Basisschriftsatz für ASCII + Latin-1. UTF-8 bleibt das Dateiformat; nicht unterstützte Glyphen werden sichtbar ersetzt, Bytes beim Dateispeichern aber erhalten. Vollständige Schrift-/Zugänglichkeitsunterstützung wächst ab 0.2.

0.1 ein Bildschirm, 1280×720 Referenz, 1×Skalierung. Software-Rendering, keine DirectX/Vulkan/OpenGL-Zusage. 0.2 ergänzt 2×Skalierung und Tastaturnavigation über alle Bedienelemente; beschleunigte 3D-/Mehrmonitorfunktionen erst in eigenen späteren Releases.

## Apps und eigene Laufzeit

Eigene libc zuerst: memcpy/memmove/memset/memcmp, Strings, malloc/free/calloc/realloc, Integerumwandlung, begrenztes printf, errno-Mapping und Start/Ende. Jeder Teil bekommt Größen-/Overflow-/Overlap-/OOM-Tests. Kein vollständiges ISO-hosted-C-Versprechen, bis alle nötigen Teile implementiert und geprüft sind.

libos enthält native Objekt-/Datei-/Prozess-/IPC-Funktionen; libui die Grafik-/Interaktionsschnittstelle. Standardbibliotheken und Dienste werden selbst implementiert. Kernelabhängigkeiten sind gemäß Entscheidungsregister zulässig und werden nicht automatisch Teil der User-Laufzeit.

Terminal/Shell: pwd, cd, ls, cat, mkdir, cp, mv, rm, echo, help, ps, kill; externe Programme, Returncodes, einfache Anführungszeichen und >/<-Umleitung. Keine POSIX-Shell-Komplettheit. Löschen erfordert eindeutige Zielpfade; kein implizites rekursives rm.

Texteditor: UTF-8-Datei bis 16 MiB, öffnen/speichern/suchen, Undo/Redo mit 64 MiB Budget, ungespeicherte Änderungen sichtbar. Dateimanager: Navigation, Dateiarbeit, Konflikt-/Fehlerdialoge. Einstellungen: Anzeige, Layout, Sitzung. Systemübersicht: Prozessliste, CPU-/Speicherwerte, berechtigtes Beenden.

## Netzwerk

Kernelmechanismen: Ethernet, ARP, IPv4, ICMP Echo, UDP und TCP. netcfgd verwaltet DHCPv4/Adresskonfiguration und DNS-Resolver. Zuerst ein Interface und MTU 1500; Offloads und IPv4-Fragmentreassembly nicht im ersten Laborprofil. Fragmente werden explizit verworfen, statt als vollständige Pakete verarbeitet.

TCP ist ein eigener RFC-basierter Stack mit Sequenz-/Fensterprüfung, Retransmission, Backoff, RTO und Connection-State-Machine. Grenzwerte pro Prozess/System, begrenzte Queues, Timeouts und Lasttests sind Pflicht. DNS-Antworten einschließlich Kompressionszeigern werden überlauf-/zyklusgeprüft.

P6 beginnt im isolierten Testnetz. Öffentlicher sicherer Netzwerkbetrieb verlangt TLS 1.3, eigene Trust-/Zertifikatsverwaltung und Negativtests vor P7-Freigabe. TLS-Protokoll und User-Bibliothek werden selbst entwickelt; standardisierte Algorithmen und Testvektoren verwenden. Kein selbst erfundenes Verschlüsselungsverfahren. Ein kontrollierter Entropiepool und DRBG müssen vor Schlüsseln/Sitzungen bereitstehen; Boot-ID und Uhrzeit sind keine alleinige Entropiequelle.

IPv6 folgt in 0.3 einschließlich ICMPv6/NDP, Adresskonfiguration und klar definierter Fragment-/Firewallregeln. Netzschnittstellen für Apps gehen über libos; keine Abhängigkeit von einer Windows-App-Laufzeit.

## Pakete, Installation und Updates ab 0.2

Disklayout: GPT, 512-MiB-ESP, 8-GiB-System-A, 8-GiB-System-B, verbleibender Platz Daten; Mindesttestdisk 32 GiB. 0.1 verwendet ausschließlich Testimages. Installer verarbeitet nur ein ausdrücklich ausgewähltes Ziel und zeigt die geplante Partitionierung vor Ausführung an.

Paketformat: eigener versionierter Container aus Manifestlänge, Manifest, Inhaltsverzeichnis und Payload-Blobs. SHA-256 je Blob, Ed25519-Signatur über kanonisches Manifest und Inhaltsverzeichnis. Pfade müssen relativ sein; kein .., kein absoluter Pfad, keine doppelten Einträge. Limits und Dekompressionsbudgets werden vor Paketimplementierung im Formatvertrag festgehalten. Anfangs unkomprimierte Payloads.

Systemupdates schreiben ausschließlich den inaktiven Slot, prüfen alle Inhalte, flushen und markieren ihn als pending. Bootstatus auf der ESP enthält zwei unabhängig prüfbare versionierte Kopien mit Sequenz und Prüfsumme. Nach drei erfolglosen pending-Starts kehrt der Loader zum letzten bestätigten Slot zurück. Bestätigung erfolgt durch den Health-Dienst nach Start der Kerneldienste.

Benutzerdaten liegen getrennt. Rückkehr zum alten System setzt kompatible Daten-/Konfigurationsversionen voraus; irreversible Datenmigrationen sind in 0.2 untersagt. Recovery startet aus separatem geprüften Image. Defekter Updatezustand darf nie beide bestätigten Systemstände gleichzeitig zerstören.

Release-Schlüssel werden erst im Releaseauftrag erzeugt und außerhalb des Repositories verwahrt. Dieser Plan erfindet oder enthält keine echten Schlüssel.
