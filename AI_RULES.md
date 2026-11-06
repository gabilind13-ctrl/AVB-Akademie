# AI_RULES.md — Entwicklungsregeln für AVB Akademie

> Diese Regeln gelten dauerhaft für alle KI-gestützten und manuellen Entwicklungen an diesem Projekt. Sie müssen bei jeder Änderung eingehalten werden.

---

1. **Bestehenden Code bevorzugen.** Vor jeder neuen Implementierung prüfen, ob bereits eine Lösung existiert und diese erweitert wird.

2. **Keine doppelten Funktionen erstellen.** Wenn eine ähnliche Funktionalität bereits existiert, wird sie wiederverwendet oder erweitert — niemals parallel neu erstellt.

3. **Vor Änderungen den vorhandenen Code analysieren.** Jede Änderung erfordert ein Verständnis des umgebenden Codes, der Abhängigkeiten und der betroffenen Komponenten.

4. **Komponenten wiederverwenden.** Gemeinsame UI-Elemente, Hilfsfunktionen und Typen werden zentral definiert und importiert — nicht dupliziert.

5. **Dateien sauber strukturieren.** Eine Datei hat einen klaren Zweck. Zusammengehörige Logik wird zusammengefasst, getrennte Verantwortlichkeiten getrennt.

6. **Verständliche Kommentare schreiben.** Kommentare erklären das WARUM, nicht das WAS. Nur bei nicht offensichtlichen Zusammenhängen, versteckten Constraints oder Workarounds.

7. **Keine unnötigen Bibliotheken installieren.** Vor jeder neuen Abhängigkeit prüfen, ob das Problem mit bestehenden Mitteln lösbar ist. Neue Dependencies nur bei echtem Mehrwert.

8. **Bestehende Architektur beibehalten.** React + Vite + Supabase + Tailwind. Keine abweichenden Patterns oder Frameworks einführen, ohne dies explizit zu begründen.

9. **Sicherheitsregeln einhalten.** RLS auf allen Tabellen, keine service_role-Keys im Frontend, Admin-CRUD nur über `admin-api` Edge Function, CORS-Header in allen Edge Functions, Eingabevalidierung an Systemgrenzen.

10. **Performance berücksichtigen.** Keine unnötigen Re-Renders, effiziente Queries, Indizes prüfen, Lazy Loading wo sinnvoll, Bundle-Size im Blick behalten.

11. **Mobile First beachten.** Jede UI wird zuerst für mobile Viewports entwickelt und dann für größere Screens erweitert. Responsive Breakpoints konsequent nutzen.

12. **Änderungen niemals durchführen, ohne die Auswirkungen auf das Gesamtsystem zu prüfen.** Jede Änderung wird auf Side-Effects, abhängige Komponenten, RLS-Policies, Edge Functions und Datenbank-Integrität geprüft.

---

## Arbeitsweise

### Zu Beginn jeder neuen Unterhaltung:
1. `PROJECT_CONTEXT.md` lesen
2. `TASKS.md` lesen
3. `CHANGELOG.md` lesen
4. `NEXT_SESSION.md` lesen
5. `KNOWN_ISSUES.md` lesen
6. `AI_RULES.md` lesen
7. Aktuellen Code analysieren
8. Tatsächlichen Projektstand als einzige Quelle der Wahrheit nutzen
9. Vor jeder Codeänderung prüfen, ob bekannte Probleme oder Risiken aus `KNOWN_ISSUES.md` den betroffenen Bereich betreffen

### Nach jeder Sitzung:
1. `PROJECT_CONTEXT.md` aktualisieren
2. `TASKS.md` aktualisieren
3. `CHANGELOG.md` aktualisieren
4. `NEXT_SESSION.md` aktualisieren
5. `KNOWN_ISSUES.md` aktualisieren — neue Probleme/Risiken ergänzen, gelöste Probleme in Abschnitt "Gelöst" verschieben, Prioritäten und Status aktualisieren
6. Neue Architekturentscheidungen dokumentieren
7. Offene Punkte ergänzen
8. Erledigte Aufgaben abhaken
9. Kurze Sitzungszusammenfassung speichern
