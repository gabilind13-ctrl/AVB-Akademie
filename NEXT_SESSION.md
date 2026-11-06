# NEXT_SESSION.md — Automatische Sitzungsübergabe

> Diese Datei dient als Übergabe für die nächste Entwicklungssitzung. Sie wird am Ende jeder relevanten Änderung vollständig aktualisiert. Vor jeder neuen Entwicklung müssen `PROJECT_CONTEXT.md`, `TASKS.md`, `CHANGELOG.md`, `KNOWN_ISSUES.md` und diese Datei zuerst gelesen werden. Anschließend ist der aktuelle Quellcode zu analysieren. Der Quellcode und die Dokumentation bilden gemeinsam die einzige Quelle der Wahrheit. Chatverläufe dienen ausschließlich als zusätzliche Information und dürfen niemals Vorrang vor dem tatsächlichen Projektstand haben.

---

## 1. Aktueller Projektstatus

### Zusammenfassung
Die AVB Akademie ist eine vollständige, produktionsreife deutschsprachige Webplattform für Sicherheits- und Veranstaltungsschulungen. Frontend (React 18 + Vite + TypeScript + Tailwind), Backend (Supabase), Admin-CMS und Social-Media-Modul sind implementiert und der Build läuft fehlerfrei durch (1684 Module, 110 prerendered URLs, 0 Fehler).

### Vollständig funktionierende Module
- **Öffentliche Website** (~60 Seiten): Home, LandingPage, Seminarkatalog mit ~20 Detailseiten, InhouseSeminare + dynamische Detailseiten, Blog-System (Blog, BlogArticle, BlogArticleNew, BlogExamples), Kooperationen (3 Partner + 10 Seminar-Detailseiten), Partnerprogramm, Kontakt, Widerruf, Terminkalender, 6 regionale SEO-Landingpages, 3 Themenbereiche, 4 Pfeiler-Seiten, Wissensdatenbank, Analyse, Impressum, Datenschutz, WorkReportPDF
- **Admin-CMS** (13 Seiten): Dashboard, Seminare, Blog, Themenseiten, Kooperationen, Regionale Seiten, Statische Seiten, Anfragen, Medien, Einstellungen, Aktivitäten, Benutzer, Login
- **Social Media Modul** (10 Seiten): SocialDashboard, BeitraegeAdmin, PostComposer, SocialKalender, SocialMedien, KampagnenAdmin, AutomationenAdmin, ContentAssistent, SocialAnalytics, PublerSettings
- **Telegram Bot**: Webhook-basierter Bot fuer Social-Media-Content-Erstellung, Freigabe-Workflow mit Konversations-Status, Admin-UI mit Beitragsvorschau/Freigabe/Publer-Versand
- **CRM-Modul**: Kontakte (Auto-Import aus 5 Quellen, Deduplizierung) und Aufgaben (Kanban-Board, Prioritaeten, Faelligkeiten)
- **Edge Functions** (12): admin-api, contact-form-handler, inhouse-seminar-request-handler, affiliate-form-handler, widerruf-form-handler, chatbot, blog-posts, publish-to-publer, publer-accounts, social-ai-assistant, sync-seminar-dates, telegram-webhook
- **Datenbank** (~28 Tabellen, 38+ Migrationen): RLS auf allen Tabellen, Security-Hardening abgeschlossen
- **SEO-Infrastruktur**: Sitemap-Generierung, Puppeteer-Prerendering (110 URLs), robots.txt
- **KI-Chatbot**: OpenAI thread-basiert, persistiert in `chat_sessions`
- **DSGVO**: CookieBanner implementiert
- **Monitoring**: web-vitals integriert

### Module in Entwicklung
- (keine aktiven Entwicklungsaufgaben)

---

## 2. Heute erledigt

### Live-Analyse repariert & neue Traffic-Diagramme (2026-10-05)
- **Bug:** Live-Traffic zeigte fast keinen September — Edge Function `analytics` war durch das 1000-Zeilen-Limit der REST-Schnittstelle abgeschnitten (siehe KNOWN_ISSUES W-003)
- **Loesung:** DB-Funktion `get_page_view_stats(p_days)` aggregiert alles in SQL (Europe/Berlin), nur fuer service_role ausfuehrbar; Edge Function neu deployed
- **UI:** `TrafficChart`-Komponente (Kurvendiagramm mit Tooltip), Zeitraum-Auswahl 7/30/90 Tage auf `/analyse`
- **Hinweis:** Zahlen weichen von Bolt Analytics ab (Bolt zaehlt serverseitig inkl. Bots/Adblocker; eigenes Tracking nur echte Browser mit wiedererkannter Session) — Verlauf ist vergleichbar
- **Blog-Planung** im Memberbereich und **Breadcrumb** auf Blog-Uebersicht ebenfalls heute umgesetzt
- **Build:** 117 prerendered URLs, 0 Fehler

### Umfassende SEO-Optimierung (2026-09-28)
- **Strukturierte Daten:** Course/FAQ/Breadcrumb-Schema auf allen 15 Seminar-Detailseiten, Article-Schema auf Blog-Seiten
- **Bug behoben:** addStructuredData-Aufrufmuster auf 13 Seiten korrigiert
- **Interne Verlinkung:** RelatedSeminars-Komponente auf allen 15 Detailseiten, BreadcrumbNav auf 25+ Seiten
- **Startseite:** Neuer "Unsere Kernkompetenzen"-Bereich (4 Kernbereiche prominent)
- **Footer:** Besuchersicherheit-Link, "Beliebte Seminare"-Abschnitt
- **Sitemap:** Prioritaeten optimiert, 116 URLs
- **Performance:** loading="lazy" auf Startseite
- **Neue Komponenten:** RelatedSeminars.tsx, BreadcrumbNav.tsx
- **Neue Hilfsfunktionen:** createArticleSchema in seo.ts, og:locale
- **Build:** 116 prerendered URLs, 0 Fehler

### Weitere Seminar-Buchungslinks und Umbenennung (frueher)
- **Umbenannt:** "Brandpräventionsbeauftragter" -> "Brandschutz beim Umgang mit Lithium-Ionen-Batterien (26101950)", Datum 19.10.2026, Link zur Kurskalender-Ansicht 757
- **4 weitere Buchungslinks hinzugefuegt:**
  - Brandschutz- und Evakuierungshelfer -> kurskalender.avb-akademie.de/de/classes?course=Brandschutz
  - KI-Beauftragter im Brandschutz und Arbeitsschutz -> kurskalender.avb-akademie.de/de/classes?course=Brandschutz
  - Jaehrliche Wiederholungsunterweisung SAP/ZUP -> kurskalender.avb-akademie.de/de/classes?course=j%C3%A4hrliche (Datum auf 21.09.2026 aktualisiert)
  - Kuenstliche Intelligenz im Alltag nutzen – Grundlagenkurs -> kurskalender.avb-akademie.de/de/classes?course=K%C3%BCnstliche%20Intelligenz%20im%20Alltag%20nutzen%20%E2%80%93%20Grundlagenkurs
- Code und Datenbank wurden beide aktualisiert

### Individuelle Buchungslinks pro Seminar (frueher heute)
- **Neues Feld `booking_url`:** Seminare koennen nun einen individuellen Buchungslink hinterlegt bekommen
- **"Jetzt buchen"-Button umgestellt:** Bei offenen Seminaren oeffnet der Button direkt die richtige Seite im Kurskalender (kurskalender.avb-akademie.de) in einem neuen Browser-Fenster
- **Fallback:** Seminare ohne hinterlegten Link nutzen den generellen Kurskalender
- **Seminar umbenannt:** "Basiswissen Sicherheitskonzepte erstellen nach § 43 MVStättVO" -> "Sicherheitskonzepte erstellen nach § 43 MVStättVO"
- Inhouse-Seminare bleiben unververaendert (Anfrage-Formular)

### Geaenderte Dateien
- `src/types/index.ts` — Neues optionales Feld `bookingUrl` im Seminar-Interface
- `src/data/seminars.ts` — Buchungslinks fuer 8 Seminare insgesamt, 2 Umbenennungen, Datum-Aktualisierungen
- `src/pages/Seminare.tsx` — Button zu externem Link umgebaut, `booking_url` im DB-Select ergaenzt
- `supabase/migrations/20260823120000_add_booking_url_to_seminars.sql` — Neue Migration (booking_url Spalte)

### Verifikation
- `npm run build` erfolgreich: 111 prerendered URLs, 0 Fehler

---

## 3. Offene Aufgaben

### 🔴 Kritisch
- (keine kritischen Aufgaben)

### 🟠 Wichtig
- [ ] **Seminardaten-Konsolidierung**: Statische Daten-Dateien (`seminars.ts`, `allSeminars.ts`, `inhouseSeminarCatalog.ts`) mit DB-Tabellen (`seminars`, `seminar_dates`) abgleichen — aktuell existieren beide parallel als Übergangsphase
- [ ] **Publer-Credentials konfigurieren**: Publer-Publishing-Flow testen — `publer_account_id`-Werte in `social_platforms` und gültige API-Credentials erforderlich
- [ ] **WorkReportPDF verifizieren**: PDF-Generierungsansatz prüfen und ggf. korrigieren

### 🟢 Optional
- [ ] Kooperations-Seminar-Detailseiten (`src/pages/kooperation/`) auf Vollständigkeit prüfen
- [ ] Öffentliche Website: Custom switch-Routing ggf. zu react-router migrieren (nur bei Bedarf — aktuell funktional, kann aber bei komplexen Routing-Szenarien limitierend sein)
- [ ] Blog-System: Konsolidierung von BlogArticle/BlogArticleNew/BlogExamples prüfen (aktuell bewusst unterschiedliche Use-Cases, aber Wartbarkeit über Zeit beobachten)

---

## 4. Nächster sinnvoller Entwicklungsschritt

**Empfehlung: Seminardaten-Konsolidierung**

Die technisch sinnvollste nächste Aufgabe ist die Konsolidierung der Seminardaten. Aktuell existieren Seminardaten sowohl als statische TypeScript-Dateien (`seminars.ts`, `allSeminars.ts`, `inhouseSeminarCatalog.ts`) als auch in Supabase-Tabellen (`seminars`, `seminar_dates`). Das führt zu potenzieller Inkonsistenz und doppeltem Wartungsaufwand.

**Vorgehen:**
1. `src/data/seminars.ts` und `src/data/allSeminars.ts` analysieren — welche Felder werden von welchen Seiten verwendet?
2. `src/pages/Seminare.tsx` und `src/pages/Terminkalender.tsx` prüfen — lesen diese bereits aus der DB oder nur aus statischen Dateien?
3. `sync-seminar-dates` Edge Function verstehen — wie wird `next_date` berechnet?
4. Datenstruktur in DB mit statischen Daten abgleichen
5. Seiten schrittweise auf DB-Queries umstellen (mit Fallback auf statische Daten während der Migration)
6. Statische Dateien nach vollständiger Migration entfernen

**Alternative**: Falls der Nutzer neue Funktionen wünscht, sollten diese zuerst spezifiziert werden — die Konsolidierung kann parallel laufen.

---

## 5. Betroffene Dateien (nächste Sitzung)

### Seminardaten-Konsolidierung
- `src/data/seminars.ts` — Haupt-Seminar-Katalog (12KB)
- `src/data/allSeminars.ts` — Erweiterter Katalog (6KB)
- `src/data/inhouseSeminarCatalog.ts` — Inhouse-Katalog (16KB)
- `src/pages/Seminare.tsx` — Seminarkatalog-Seite (liest vermutlich aus statischen Daten)
- `src/pages/Terminkalender.tsx` — Terminkalender (liest `seminar_dates` aus DB)
- `src/pages/InhouseSeminare.tsx` — Inhouse-Übersicht
- `src/pages/InhouseSeminarDetail.tsx` — Dynamische Detailseite
- `supabase/functions/sync-seminar-dates/index.ts` — Datum-Synchronisation
- `src/types/index.ts` — Seminar-Interface ggf. erweitern

### Bei anderen Aufgaben
- `src/pages/WorkReportPDF.tsx` — PDF-Generierung prüfen
- `src/admin/pages/social/PublerSettings.tsx` — Publer-Konfiguration
- `supabase/functions/publish-to-publer/index.ts` — Publer-Publishing
- `src/pages/kooperation/Koop*.tsx` — 10 Kooperations-Detailseiten

---

## 6. Bekannte Probleme

### Fehler
- Keine expliziten TODOs/FixMe im Code gefunden
- Keine Build-Fehler

### Risiken
- **Dateninkonsistenz**: Seminardaten existieren parallel statisch und in DB — Änderungen müssen an beiden Stellen vorgenommen werden
- **Publer-Abhängigkeit**: Social-Media-Publishing hängt von externen Credentials ab — bei Ablauf oder Fehlkonfiguration schlägt Publishing still fehl
- **Custom Routing**: Öffentliche Website nutzt switch-basiertes Routing statt react-router — bei komplexeren Routing-Anforderungen (z.B. verschachtelte Parameter, Query-String-Handling) limitierend

### Technische Schulden
- Statische Seminardaten als Übergangsphase — sollten vollständig in DB migriert werden
- `analyseData.ts` (33KB) und `blogPosts.ts` (108KB) sind sehr große statische Dateien — ggf. in DB migrieren
- `ContentAssistent.tsx` (74KB) und `PostComposer.tsx` (57KB) sind sehr große Komponenten — ggf. in kleinere Sub-Komponenten aufteilen

### Unfertige Funktionen
- Publer-Publishing: Implementiert aber nicht verifiziert (Credentials fehlen)
- WorkReportPDF: Funktionsweise unklar, nicht verifiziert

### Temporäre Workarounds
- Keine bekannten Workarounds im Code

---

## 7. Architekturänderungen

### Diese Sitzung
- Keine Architekturänderungen am Code — nur Dokumentationsaktualisierung

### Aktuelle Architektur (dokumentiert)
- **Routing (öffentlich)**: Custom `pathToPage`/`handleNavigate`-Pattern mit `switch`-Statement in `App.tsx` — NICHT react-router
- **Routing (Admin)**: react-router-dom v6 mit nested Routes, `<Outlet>`, `NavLink`
- **Admin-Datenzugriff**: Custom `adminDb`-Client (Supabase-mimicking Query-Builder, thenable Pattern) → `admin-api` Edge Function (service_role)
- **Auth**: Supabase email/password, `admin_profiles` mit Rollen (admin/editor/content_creator/viewer), `AuthContext` mit `canEdit`/`canPublish`/`canApprove`-Flags
- **Lazy Loading**: Alle Seiten via `React.lazy()` + `Suspense`
- **SEO**: Puppeteer-Prerendering (110 URLs), Sitemap-Generierung, robots.txt
- **Styling**: Tailwind mit Custom-Theme (primary blue-gray, accent orange, Inter-Font)

### Neue Ordner/Komponenten/Services (gesamtbestand)
- `src/components/geo/` — 6 SEO/Geo-Komponenten
- `src/components/kooperation/` — 7 Kooperations-Komponenten
- `src/components/kooperation-felstead/` — 7 Felstead-Komponenten
- `src/components/kooperation-gruenberg/` — 7 Gruenberg-Komponenten
- `src/pages/kooperation/` — 10 Kooperations-Seminar-Detailseiten
- `src/admin/pages/social/` — 10 Social-Media-Admin-Seiten

---

## 8. Hinweise für die nächste KI-Sitzung

### Aktueller Projektstand
Die AVB Akademie ist eine vollständige React/Supabase-Anwendung mit ~60 öffentlichen Seiten, einem Admin-CMS (23 Admin-Seiten inkl. Social Media), 11 Edge Functions, ~24 DB-Tabellen und 37 Migrationen. Der Build läuft fehlerfrei (110 prerendered URLs). Alle Kernfunktionen sind implementiert. Die Dokumentation (`PROJECT_CONTEXT.md`, `TASKS.md`, `CHANGELOG.md`, `NEXT_SESSION.md`) wurde in dieser Sitzung gegen den Quellcode verifiziert und korrigiert.

### Wichtigste Entscheidungen
- Admin-CRUD läuft über zentrale `admin-api` Edge Function (service_role) — zusätzlicher Sicherheitslayer
- Öffentliche Website nutzt custom switch-basiertes Routing (nicht react-router) — historische Entscheidung
- Blog-System hat drei Komponententypen (BlogArticle, BlogArticleNew, BlogExamples) für unterschiedliche Use-Cases — keine Duplikate
- RLS auf allen Tabellen, GraphQL-Schema-Exposure deaktiviert
- Alle Seiten lazy-loaded für Code-Splitting

### Wichtigste Regeln (aus AI_RULES.md)
1. Bestehenden Code bevorzugen — vor neuer Implementierung prüfen, ob eine Lösung existiert
2. Keine doppelten Funktionen erstellen — wiederverwenden oder erweitern
3. Vor Änderungen den vorhandenen Code analysieren
4. Keine unnötigen Bibliotheken installieren
5. RLS auf allen Tabellen, keine service_role-Keys im Frontend
6. Admin-CRUD nur über `admin-api` Edge Function
7. CORS-Header in allen Edge Functions
8. Mobile First — jede UI zuerst für mobile Viewports entwickeln
9. Änderungen niemals ohne Prüfung der Auswirkungen auf das Gesamtsystem durchführen

### Offene Risiken
- Seminardaten parallel statisch und in DB — Inkonsistenzrisiko
- Publer-Credentials nicht konfiguriert — Publishing ungetestet
- Custom Routing kann bei komplexeren Anforderungen limitierend sein

### Empfohlener erster Entwicklungsschritt
Seminardaten-Konsolidierung: Statische Seminar-Daten-Dateien mit DB-Tabellen abgleichen und schrittweise auf DB-Queries umstellen. Beginnen mit Analyse von `src/data/seminars.ts` und `src/pages/Seminare.tsx`, um zu verstehen, welche Felder verwendet werden und ob die DB-Struktur kompatibel ist.

### Arbeitsregel
Am Ende jeder Sitzung werden automatisch aktualisiert: `PROJECT_CONTEXT.md`, `TASKS.md`, `CHANGELOG.md`, `NEXT_SESSION.md`, `KNOWN_ISSUES.md`. Vor jeder neuen Entwicklung müssen diese Dateien zuerst gelesen werden. Anschließend ist der aktuelle Quellcode zu analysieren. Der Quellcode und die Dokumentation bilden gemeinsam die einzige Quelle der Wahrheit. Chatverläufe dienen ausschließlich als zusätzliche Information und dürfen niemals Vorrang vor dem tatsächlichen Projektstand haben.
