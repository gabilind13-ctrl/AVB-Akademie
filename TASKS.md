# TASKS.md — AVB Akademie

> Aufgabenverwaltung für das Projekt. Erledigte Aufgaben werden automatisch in den entsprechenden Bereich verschoben.

---

## ✅ Erledigt

- [x] Seminarheft 2027: Download-Link auf Startseite auf neue PDF umgestellt, Textbezuege aktualisiert -- 2026-10-27
- [x] Startseite UWG-Entschaerfung (Statistik-Kacheln, Trust-Badges, Leistungsportfolio, "zertifiziert"-Aussagen) -- 2026-10-18 (offene Nachweise siehe KNOWN_ISSUES R-UWG-001)
- [x] Projektstruktur analysiert und dokumentiert
- [x] PROJECT_CONTEXT.md erstellt und gegen Codebasis verifiziert (zentrale Wissensquelle)
- [x] TASKS.md erstellt
- [x] CHANGELOG.md erstellt
- [x] AI_RULES.md erstellt
- [x] NEXT_SESSION.md erstellt (8 Bereiche: Projektstatus, Heute erledigt, Offene Aufgaben, Nächster Schritt, Betroffene Dateien, Bekannte Probleme, Architekturänderungen, Hinweise)
- [x] AI_RULES.md um NEXT_SESSION.md im Pflichtworkflow ergänzt
- [x] KNOWN_ISSUES.md erstellt (7 Bereiche: Kritische Fehler, Bugs, Technische Schulden, Verbesserungsideen, Wiederkehrende Fehler, Risiken, Erledigte Probleme)
- [x] AI_RULES.md um KNOWN_ISSUES.md im Pflichtworkflow ergänzt
- [x] Codebase-Audit durchgeführt (Sicherheit, Dateigrößen, TypeScript, Accessibility, Tests)
- [x] Öffentliche Website (~60 Seiten, 63 Page-Dateien) implementiert
- [x] Custom switch-basiertes Routing für öffentliche Website implementiert
- [x] react-router-basiertes Admin-Routing implementiert
- [x] Lazy Loading für alle Seiten (React.lazy + Suspense)
- [x] Admin-CMS (13 Admin-Seiten) implementiert
- [x] Social Media Modul (10 Seiten) implementiert
- [x] 11 Edge Functions deployed
- [x] 37 Datenbank-Migrationen angewendet
- [x] RLS auf allen Tabellen aktiviert
- [x] Security-Hardening durchgeführt (GraphQL deaktiviert, Grants widerrufen)
- [x] KI-Chatbot (OpenAI) integriert
- [x] Publer-Integration für Social Media implementiert
- [x] DSGVO-CookieBanner implementiert
- [x] SEO-Landingpage-System (RegionalLanding 6 Städte / ThemeArea 3 Themen / Pillar 4 Pfeiler)
- [x] SEO-Infrastruktur: Sitemap-Generierung, Puppeteer-Prerendering (110 URLs), robots.txt
- [x] Web-Vitals-Monitoring integriert
- [x] Scroll-Animationen (IntersectionObserver)
- [x] Custom Tailwind-Theme (primary/accent-Ramps, Inter-Font, Custom-Animationen)
- [x] 10 Kooperations-Seminar-Detailseiten (src/pages/kooperation/)
- [x] 36 Komponenten in 4 Verzeichnissen (components/, geo/, kooperation/, kooperation-felstead/, kooperation-gruenberg/)
- [x] 10 statische Daten-Dateien (src/data/)
- [x] XSS-Sicherheitslücke in Blog-Komponenten geschlossen (DOMPurify) — K-001 gelöst
- [x] ErrorBoundary für lazy-loaded Seiten erstellt und eingebunden — B-004 gelöst
- [x] ThemeArea.tsx auf DB-First-Pattern umgestellt (mit statischem Fallback)
- [x] ThemeArea.tsx: Unreachable TypeScript-Vergleich korrigiert
- [x] console.log in Seminare.tsx entfernt — B-003 teilweise gelöst
- [x] Neue Abhängigkeit: dompurify + @types/dompurify
- [x] TikTok Clip Studio: Upload-Zentrale mit Drag-and-drop, Mehrfach-Upload und Medien-Galerie
- [x] TikTok Clip Studio: Medienbibliothek-Auswahldialog fuer Wiederverwendung vorhandener Dateien
- [x] Neue Tabelle tiktok_clip_media fuer mehrere Medien pro Clip
- [x] Individuelle Buchungslinks pro Seminar: "Jetzt buchen"-Button oeffnet direkt den Kurskalender
- [x] Seminar umbenannt: "Sicherheitskonzepte erstellen nach § 43 MVStättVO"
- [x] Seminar umbenannt: "Brandpräventionsbeauftragter" -> "Brandschutz beim Umgang mit Lithium-Ionen-Batterien (26101950)"
- [x] Buchungslinks fuer 5 weitere Seminare (Lithium-Ionen, Evakuierungshelfer, KI-Beauftragter, SAP-Fortbildung, KI-Alltag)
- [x] 3 weitere Seminar-Umbenennungen (Sicherheitsbeauftragten Erstschulung, Sicherheitskonzepte § 43, KI in der Sicherheit)
- [x] 3 weitere Buchungslinks (Sicherheitsbeauftragten, Grossveranstaltungen, KI in der Sicherheit)
- [x] 5 weitere Buchungslinks (Notfall Hochschulen, Notfall KMU, Fortbildung Brandschutz, Sportgeraete, Spielplaetze) + 2 Kursnummern-Ergaenzungen
- [x] SEO: Strukturierte Daten (Course, FAQ, Breadcrumb Schema) auf allen 15 Seminar-Detailseiten
- [x] SEO: addStructuredData-Bug auf 13 Seiten behoben (falsches Aufrufmuster)
- [x] SEO: Sitemap-Prioritaeten optimiert (Kernbereiche auf 0.9, Seminare auf 0.8)
- [x] SEO: Neuer Bereich "Unsere Kernkompetenzen" auf Startseite
- [x] SEO: Footer erweitert (Besuchersicherheit-Link, "Beliebte Seminare" Abschnitt)
- [x] SEO: Interne Verlinkung "Verwandte Seminare" auf allen 15 Seminar-Detailseiten
- [x] SEO: BreadcrumbNav-Komponente auf 25+ Detailseiten
- [x] SEO: Article-Schema fuer Blog-Artikel
- [x] SEO: og:locale Meta-Tag, createArticleSchema-Hilfsfunktion
- [x] SEO: RelatedSeminars-Komponente erstellt
- [x] Performance: loading="lazy" fuer Bilder und YouTube-iframe auf Startseite
- [x] SEO: Google Search Console / Google Business Profile Massnahmenplan erstellt
- [x] Webinar: Altes Datum (28.09.2026) entfernt, nur 12.10.2026 aktiv
- [x] Kooperations-Detailseiten: BreadcrumbNav, Course-Schema, Canonical URL auf alle 8 Seiten
- [x] Seminardaten-Konsolidierung: allSeminars.ts mit DB abgeglichen, 3 fehlende Seminare ergaenzt, 2 falsche Routen korrigiert
- [x] Blog-Planung im Memberbereich (Status Entwurf/Geplant/Veroeffentlicht, automatische Veroeffentlichung am Datum)
- [x] Live-Analyse repariert: 1000-Zeilen-Limit behoben (SQL-Aggregation), Berlin-Zeitzone, Kurvendiagramme, Zeitraum 7/30/90 Tage

---

## 🔄 In Bearbeitung

- (keine aktiven Aufgaben)

---

## ⏳ Geplant

- [ ] Seminardaten-Konsolidierung Phase 2: statische `seminars.ts` langfristig durch reine DB-Quelle ersetzen
- [ ] WorkReportPDF: ggf. aus Produktion entfernen (internes Tool)
- [ ] Publer-Credentials konfigurieren und Publishing-Flow testen
- [ ] Kooperations-Seminar-Detailseiten (`src/pages/kooperation/`) auf Vollständigkeit prüfen

---

## ❌ Blockiert

- (keine blockierten Aufgaben)
