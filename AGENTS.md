# Project OS: Arbeitsregeln

- Lies zuerst START_HERE.md und die zum Auftrag gehörenden Specs.
- Normative Baseline: docs/ENTSCHEIDUNGEN.md und aktuelle Dokumente unter docs/, ausgenommen docs/history/.
- Historische Entwürfe sind keine aktuellen Anweisungen.
- Dieses Repository beginnt als Spezifikation. Implementiere Code, wenn der konkrete Nutzerauftrag die Implementierung verlangt.
- Verändere öffentliche Boot-/Syscall-/Treiberverträge nicht stillschweigend.
- C17 und Assembler gemäß Build-Spec; keine Host-Laufzeit in Zielartefakten.
- Prüfe echte Verhaltenseigenschaften und Fehlerpfade. Nicht implementierte Funktionen müssen dies ausdrücklich melden.
- Dokumentiere verwendete Fremdkomponenten mit Version, Herkunft und Lizenztext; Kernelcode darf wiederverwendet werden, Systemdienste und Standardbibliotheken werden selbst entwickelt.
- Bewahre Nutzeränderungen. Keine destruktiven Git-Kommandos oder realen Datenträgerschreibvorgänge ohne passenden Auftrag.
- Neue Arbeitsbranches: codex/<thema>.
- Für jeden gelieferten Implementierungsschritt: tatsächliche Build-/Testergebnisse und verbleibende Einschränkungen dokumentieren.
