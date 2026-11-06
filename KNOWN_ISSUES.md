# KNOWN_ISSUES.md — Zentrale Fehler- und Verbesserungsdokumentation

> Zentrale Wissensdatenbank für alle bekannten Probleme, Bugs, technischen Schulden und Verbesserungsideen. Wird nach jeder relevanten Änderung aktualisiert. Vor jeder neuen Entwicklungssitzung muss diese Datei zusammen mit `PROJECT_CONTEXT.md`, `TASKS.md`, `CHANGELOG.md`, `AI_RULES.md` und `NEXT_SESSION.md` gelesen werden. Vor jeder Codeänderung prüfen, ob bekannte Probleme den betroffenen Bereich betreffen.

---

## 1. Kritische Fehler 🔴

### K-001: `dangerouslySetInnerHTML` ohne Sanitization in Blog-Komponenten — ✅ GELÖST

- **Datum der Lösung:** 2026-07-23
- **Lösung:** DOMPurify integriert. Neue Hilfsfunktion `src/lib/sanitizeHtml.ts` bereinigt alle HTML-Strings vor dem Rendern. In `BlogTemplate.tsx` und `BlogArticle.tsx` wird `sanitizeHtml()` auf alle via `dangerouslySetInnerHTML` gerenderten Inhalte angewendet.
- **Geänderte Dateien:**
  - `src/lib/sanitizeHtml.ts` (neu erstellt)
  - `src/components/BlogTemplate.tsx`
  - `src/pages/BlogArticle.tsx`
- **Siehe auch:** G-007 im Abschnitt „Erledigte Probleme"

---

## 2. Bekannte Bugs 🟠

### B-001: `as any`-Type-Casts in Analyse-Seite

- **Beschreibung:** Die Analyse-Seite verwendet an mehreren Stellen `as any`-Casts, um auf Eigenschaften zuzugreifen, die nicht im Typ definiert sind (`url`, `status`). Dies umgeht die TypeScript-Typprüfung und kann bei Refactorings zu Laufzeitfehlern führen.
- **Reproduzierbarkeit:** Immer — statisches Code-Problem
- **Betroffene Dateien:**
  - `src/pages/Analyse.tsx:585,586,834,836,846,851`
- **Workaround:** Keiner — Code funktioniert aktuell, aber ohne Typsicherheit
- **Geplante Lösung:** Typdefinitionen in `analyseData.ts` um `url?: string` und `status?: 'done' | 'pending'` erweitern, dann `as any`-Casts entfernen.

### B-002: Index-basierte React-Keys in Listen

- **Beschreibung:** Mehrere Komponenten verwenden den Array-Index als React-Key (`key={i}`). Dies kann bei dynamischen Listen (Hinzufügen/Entfernen/Sortieren) zu Rendering-Fehlern führen, da React Elemente falsch zuordnet.
- **Reproduzierbarkeit:** Bei dynamischen Listenänderungen — Elemente werden falsch zugeordnet, State bleibt am falschen Element hängen
- **Betroffene Dateien:**
  - `src/admin/pages/KooperationenAdmin.tsx:84,139`
  - `src/admin/pages/RegionaleAdmin.tsx:55,92`
  - `src/admin/pages/social/ContentAssistent.tsx:343,444,501,524-528`
  - `src/admin/pages/social/PostComposer.tsx` (mehrere)
  - `src/admin/pages/social/BeitraegeAdmin.tsx` (mehrere)
- **Workaround:** Keiner — Listen sind aktuell statisch genug, dass das Problem selten auftritt
- **Geplante Lösung:** Stabile, eindeutige Keys verwenden (z.B. `item.id`, `item.name` oder Kombination aus Index + Wert).

### B-003: `console.log` in Produktionscode — ✅ TEILWEISE GELÖST

- **Beschreibung:** `console.log` in `Seminare.tsx` entfernt. `webVitals.ts` verbleibt (ist als Performance-Monitoring legitim).
- **Verbleibende Datei:**
  - `src/lib/webVitals.ts:30` (`console.log` für Web-Vitals — bewusst behalten)
- **Status:** `Seminare.tsx` bereinigt, `webVitals.ts` verbleibt als bewusste Entscheidung

### B-004: Kein ErrorBoundary für Lazy-Loaded Seiten — ✅ GELÖST

- **Datum der Lösung:** 2026-07-23
- **Lösung:** `ErrorBoundary`-Komponente erstellt (`src/components/ErrorBoundary.tsx`). Um den Suspense-Wrapper der öffentlichen Website und um den Admin-Suspense-Wrapper in `App.tsx` gelegt. Fängt Render-Fehler ab und zeigt eine benutzerfreundliche Fehlerseite mit „Zurück zur Startseite"-Button.
- **Geänderte Dateien:**
  - `src/components/ErrorBoundary.tsx` (neu erstellt)
  - `src/App.tsx`

---

## 3. Technische Schulden ⚙️

### T-001: Sehr große Komponenten und Daten-Dateien

- **Beschreibung:** Mehrere Dateien sind ungewöhnlich groß und erschweren Wartung, Lesbarkeit und Code-Reviews. Die größten Dateien:
  - `src/data/blogPosts.ts` — 108KB / 1950 Zeilen
  - `src/admin/pages/social/ContentAssistent.tsx` — 74KB / 1622 Zeilen
  - `src/admin/pages/social/PostComposer.tsx` — 57KB / 1305 Zeilen
  - `src/pages/Home.tsx` — 53KB / 1061 Zeilen
  - `src/pages/Analyse.tsx` — 51KB / 910 Zeilen
  - `src/pages/KISeminareMain.tsx` — 35KB / 573 Zeilen
  - `src/pages/ThemeArea.tsx` — 34KB / 604 Zeilen
  - `src/data/analyseData.ts` — 33KB / 485 Zeilen
  - `src/pages/Partnerprogramm.tsx` — 32KB / 670 Zeilen
  - `src/pages/Wissensdatenbank.tsx` — 31KB / 595 Zeilen
- **Warum verbessert werden sollte:** Große Dateien sind schwer zu navigieren, fehleranfällig bei Änderungen, erschweren Code-Reviews und verlangsamen die IDE. Sie verstoßen gegen das Single Responsibility Principle.
- **Empfohlene Umsetzung:**
  - `ContentAssistent.tsx`: In Sub-Komponenten aufteilen (ContentAssistentForm, ContentAssistentPreview, ContentAssistentResults)
  - `PostComposer.tsx`: In PostComposerForm, PostComposerPreview, PostComposerScheduler aufteilen
  - `Home.tsx`: Sektionen (Hero, Features, Seminare, CTA) in separate Komponenten auslagern
  - `blogPosts.ts`: In mehrere Dateien nach Kategorie aufteilen oder in Supabase migrieren
  - `analyseData.ts`: In mehrere Dateien nach Analyse-Kategorie aufteilen

### T-002: Seminardaten parallel als statische Dateien und in Datenbank — 🔄 FORTSCHRETTEND

- **Beschreibung:** Seminardaten existieren sowohl als statische TypeScript-Dateien (`seminars.ts`, `allSeminars.ts`, `inhouseSeminarCatalog.ts`) als auch in Supabase-Tabellen (`seminars`, `seminar_dates`).
- **Aktueller Status:** `Seminare.tsx` und `ThemeArea.tsx` nutzen nun DB-First-Pattern mit statischem Fallback. Die DB enthält 43 Seminare (vs. 25 statische) und ist die primäre Quelle. `allSeminars.ts` (Navigations-Übersicht mit Kursnummern) und `inhouseSeminarCatalog.ts` (eigener Katalog mit Kategorie-System) bleiben bewusst statisch.
- **Verbleibende Arbeit:** Inhouse-Seminar-Seiten prüfen, ob diese ebenfalls DB-First nutzen sollten. Statische Fallbacks langfristig entfernen, wenn DB stabil.

### T-003: Keine Test-Infrastruktur

- **Beschreibung:** Das Projekt enthält keine Unit-Tests, Integrationstests oder End-to-End-Tests. Es gibt keine Test-Dateien, kein Test-Framework und keine Test-Scripts in `package.json`.
- **Warum verbessert werden sollte:** Keine automatisierte Qualitätssicherung. Regressionen werden erst spät entdeckt. Refactorings sind riskant.
- **Empfohlene Umsetzung:** Vitest für Unit-Tests einrichten (Vite-nativ). Kritische Pfade zuerst testen: Formular-Submit (Kontakt, Inhouse, Widerruf), Admin-Auth, Routing, Blog-Rendering. Playwright oder Cypress für E2E-Tests in Betracht ziehen.

### T-004: Direkte `fetch`-Aufrufe statt zentraler API-Client

- **Beschreibung:** Social-Media-Admin-Seiten und öffentliche Formulare rufen Edge Functions direkt via `fetch()` auf, mit manuell konstruierten URLs und Headern. Nur der Admin-Bereich nutzt den zentralen `adminDb`-Client. Das führt zu dupliziertem Code für URL-Konstruktion, Header-Setup und Fehlerbehandlung.
- **Warum verbessert werden sollte:** Code-Duplikation, inkonsistente Fehlerbehandlung, erschwert zentrale Änderungen (z.B. API-URL-Wechsel).
- **Empfohlene Umsetzung:** Zentralen API-Client erstellen (ähnlich `adminDb`, aber für öffentliche Edge Functions). Alle `fetch`-Aufrufe auf den Client umstellen.

### T-005: Custom Routing statt react-router (öffentliche Website)

- **Beschreibung:** Die öffentliche Website verwendet ein custom switch-basiertes Routing (`pathToPage`/`handleNavigate`) statt react-router. Der Admin-Bereich nutzt korrekt react-router-dom v6. Diese Zweigleisigkeit erschwert die Wartung und limitiert Routing-Funktionen (keine verschachtelten Routen, kein Query-String-Parsing, keine Route-Parameter).
- **Warum verbessert werden sollte:** Inkonsistente Architektur zwischen öffentlich und Admin. Limitierungen bei komplexen Routing-Szenarien. Höherer Wartungsaufwand.
- **Empfohlene Umsetzung:** Öffentliches Routing schrittweise auf react-router migrieren. Prerendering-Skript anpassen. Diese Migration ist aufwendig und sollte als eigenständiges Projekt behandelt werden.

### T-006: Fehlende Barrierefreiheit (Accessibility)

- **Beschreibung:** Die öffentliche Website hat eingeschränkte Barrierefreiheit. Zwar sind `focus:ring`-Styles in Admin-Formularen vorhanden, aber es fehlen durchgängige ARIA-Labels, `aria-live`-Regionen für dynamische Updates, Skip-Links, und semantische HTML-Strukturen auf manchen Seiten.
- **Warum verbessert werden sollte:** Rechtliche Anforderung (BFSG ab 28.06.2025 in Deutschland). Erschwert Nutzung für Menschen mit Behinderungen. Schlechtere SEO.
- **Empfohlene Umsetzung:** Accessibility-Audit durchführen. ARIA-Labels ergänzen, Skip-Link hinzufügen, `aria-live` für Chatbot und Formular-Feedback, semantisches HTML prüfen.

---

## 4. Verbesserungsideen 💡

### V-001: Bildoptimierung und lazy loading
- **Bereich:** Performance
- **Beschreibung:** Nur zwei Stellen verwenden `loading="lazy"` für Bilder. Eine systematische Lazy-Loading-Strategie für alle Bilder würde die Ladezeit verbessern. Responsive Bilder mit `srcset` für verschiedene Viewports.
- **Aufwand:** Mittel

### V-002: Service Worker für Offline-Funktionalität
- **Bereich:** Performance / UX
- **Beschreibung:** Ein Service Worker könnte statische Assets und Seminar-Daten cachen, um die Website auch bei schlechter Verbindung nutzbar zu machen.
- **Aufwand:** Hoch

### V-003: Open Graph und Twitter Card Meta-Tags — GELOEST
- **Bereich:** SEO / Social Media
- **Datum der Loesung:** 2026-09-28
- **Loesung:** Bereits in `updateMetaTags()` implementiert. og:locale hinzugefuegt.

### V-004: Strukturierte Daten (Schema.org) — GELOEST
- **Bereich:** SEO
- **Datum der Loesung:** 2026-09-28
- **Loesung:** Course-Schema auf allen 15 Seminar-Detailseiten + 8 Kooperations-Detailseiten, Article-Schema auf Blog-Artikeln, FAQ- und Breadcrumb-Schema auf relevanten Seiten, Organization- und Service-Schema auf Startseite.

### V-005: KI-gestützte Bildbeschreibungen
- **Bereich:** KI-Unterstützung / Barrierefreiheit
- **Beschreibung:** Der ContentAssistent könnte automatisch Alt-Texte für hochgeladene Bilder generieren, was Barrierefreiheit und SEO gleichzeitig verbessert.
- **Aufwand:** Mittel

### V-006: Zentrale Konfiguration für externe API-Keys
- **Bereich:** Sicherheit / Wartbarkeit
- **Beschreibung:** Publer- und OpenAI-API-Keys sollten zentral verwaltet werden (Supabase Vault oder `social_api_connections`-Tabelle) statt in Edge-Function-Umgebungsvariablen. Ermöglicht Key-Rotation ohne Redeploy.
- **Aufwand:** Mittel

### V-007: Code-Splitting für große Daten-Dateien
- **Bereich:** Performance
- **Beschreibung:** `blogPosts.ts` (108KB) wird in den Bundle einbezogen, auch wenn der Nutzer nicht auf den Blog-Seiten ist. Durch dynamische Imports oder DB-Migration könnte die initiale Bundle-Größe reduziert werden.
- **Aufwand:** Mittel

### V-008: Admin-Dark-Mode konsistent machen
- **Bereich:** Design
- **Beschreibung:** Der Admin-Bereich verwendet Dark-Mode-Styles, aber nicht alle Komponenten sind konsistent. Ein systematisches Dark-Mode-Theme würde die Wartung erleichtern.
- **Aufwand:** Niedrig

### V-009: Automatische Sitemap-Validierung
- **Bereich:** SEO / Automatisierung
- **Beschreibung:** Der Build-Prozess generiert die Sitemap, aber es gibt keine Validierung, dass alle gerenderten URLs auch in der Sitemap stehen. Ein CI-Check könnte das verifizieren.
- **Aufwand:** Niedrig

### V-010: Rate-Limiting für öffentliche Formulare
- **Bereich:** Sicherheit
- **Beschreibung:** Die Edge Functions für Kontakt-, Inhouse- und Widerrufsformulare haben kein Rate-Limiting. Ein Angreifer könnte das Formular massenhaft absenden und die Datenbank fluten.
- **Aufwand:** Mittel

---

## 5. Wiederkehrende Fehler

### W-003: PostgREST-Limit von 1000 Zeilen pro Abfrage

- **Ursache:** Supabase-REST-Abfragen (`.select()`) liefern standardmäßig höchstens 1000 Zeilen — ohne Fehlermeldung. Werden große Tabellen (z. B. `page_views`) im Code gezählt, sind die Ergebnisse still abgeschnitten.
- **Aufgetreten:** 2026-10-05 in der Edge Function `analytics` (Live-Analyse zeigte fast keinen September). Gelöst durch SQL-Aggregation in `get_page_view_stats`.
- **Dauerhafte Lösung:** Zählungen/Statistiken immer in SQL (RPC, `count`, `GROUP BY`) durchführen; falls Rohdaten nötig sind, mit `.range()` paginieren.
- **⚠️ Hinweis:** Bei jeder neuen Auswertung über wachsende Tabellen dieses Limit beachten.

### W-001: Fehlende oder index-basierte React-Keys

- **Ursache:** Entwickler verwenden häufig `key={i}` (Array-Index) als React-Key, weil es der einfachste Weg ist. Bei dynamischen Listen führt dies zu Rendering-Bugs.
- **Alle betroffenen Dateien:**
  - `src/admin/pages/KooperationenAdmin.tsx`
  - `src/admin/pages/RegionaleAdmin.tsx`
  - `src/admin/pages/social/ContentAssistent.tsx`
  - `src/admin/pages/social/PostComposer.tsx`
  - `src/admin/pages/social/BeitraegeAdmin.tsx`
  - `src/admin/pages/social/SocialKalender.tsx`
  - `src/admin/pages/social/SocialMedien.tsx`
- **Dauerhafte Lösung:** ESLint-Regel `react/no-array-index-key` aktivieren. Code-Review-Checkliste um "stabile Keys" ergänzen. Bei zukünftigen Listen-Komponenten immer eindeutige Keys verwenden.
- **⚠️ Hinweis:** Bei Änderungen an Listen-Komponenten immer prüfen, dass keine index-basierten Keys verwendet werden.

### W-002: `console.log` in Produktionscode

- **Ursache:** `console.log` wird oft für Debugging hinzugefügt und vor dem Commit vergessen.
- **Alle betroffenen Dateien:**
  - `src/pages/Seminare.tsx:68`
  - `src/lib/webVitals.ts:30`
- **Dauerhafte Lösung:** ESLint-Regel `no-console` mit `allow: ['error', 'warn']` aktivieren. Development-Logging via `import.meta.env.DEV` kapseln.
- **⚠️ Hinweis:** Beim Hinzufügen von Debug-Logs immer `import.meta.env.DEV`-Guard verwenden.

---

## 6. Risiken

### R-UWG-001: Offene Werbeaussagen auf der Startseite (Stand 2026-10-18)
- **Beschreibung:** Nach der UWG-Entschaerfung bleiben Punkte, die nur mit Nachweisen der Geschaeftsfuehrung geklaert werden koennen.
- **Art:** Rechtliches Risiko (Irrefuehrung, UWG)
- **Offen:**
  1. ERLEDIGT 2026-10-18: Unbelegte Teilnehmerstimmen entfernt. Neue Stimmen nur mit schriftlicher Freigabe + Hinweis zur Echtheitspruefung (§ 5b Abs. 3 UWG) einbauen
  2. ERLEDIGT 2026-10-18: "(TÜV)" auf der Startseite entfernt. Seminar-Detailseite Brandschutzbeauftragter noch pruefen
  3. OFFEN: "über 20 Jahre Erfahrung" (Olaf Jastrob, FAQ) vs. "10+ Jahre" (Kennzahl) -- Bestaetigung der Geschaeftsfuehrung fehlt
  4. ERLEDIGT 2026-10-18: Referenzen umformuliert zu "Gast-Dozent und Referent ... unter anderem bei" (kein "Partner" mehr)
  5. TEILWEISE: Kasten umbenannt in "Schulungsschwerpunkte"; "Veranstaltungsleitung, Aufsicht und Kontrolle" und "Inspektion, Event- und Baustellen-Koordination" stehen noch im Portfolio -- Entscheidung offen
  6. OFFEN: "20+ Fachdozenten" und "500+ Teilnehmer" belegbar halten
  7. ERLEDIGT 2026-10-18: "DACH" ersetzt durch "Bundesweit -- Seminare vor Ort & Inhouse"
- **Minderung:** Nachweise einholen, anwaltliche Endpruefung empfohlen

### R-001: Publer-API-Abhängigkeit
- **Beschreibung:** Das Social-Media-Publishing-Modul hängt vollständig von der Publer-API ab. Bei API-Änderungen, Ausfällen oder Kontingent-Erschöpfung ist das Publishing blockiert.
- **Art:** Externe Abhängigkeit
- **Auswirkung:** Social-Media-Posts können nicht veröffentlicht werden
- **Minderung:** Fehlerbehandlung in `publish-to-publer` prüfen. Publer-Status überwachen. Fallback manuellen Publishing dokumentieren.

### R-002: OpenAI-API-Kosten
- **Beschreibung:** Der Chatbot und der ContentAssistent nutzen die OpenAI-API. Bei hohem Nutzungsaufkommen können Kosten steigen. Keine Kostenschätzung oder Budget-Limitierung implementiert.
- **Art:** API-Limit / Kostenrisiko
- **Auswirkung:** Unerwartet hohe Rechnungen
- **Minderung:** Rate-Limiting implementieren. Nutzung monitoren. OpenAI-Usage-Limits setzen.

### R-003: Puppeteer-Prerendering bei CI/CD
- **Beschreibung:** Der Build-Prozess verwendet Puppeteer zum Prerendering von 110 URLs. Dies dauert ~3,5 Minuten und erfordert einen Chrome-Browser in der CI-Umgebung. Bei CI-Environment-Änderungen kann der Build fehlschlagen.
- **Art:** Build-Infrastruktur-Abhängigkeit
- **Auswirkung:** Build bricht fehl, Deployment blockiert
- **Minderung:** Prerendering als separaten Build-Step auslagern. Fallback auf reines SPA-Build ohne Prerendering.

### R-004: Kein Rate-Limiting auf öffentlichen Formularen
- **Beschreibung:** Die Edge Functions für Kontakt-, Inhouse-, Affiliate- und Widerrufsformulare haben kein Rate-Limiting. Bot-Angriffe könnten die Datenbank mit Spam füllen.
- **Art:** Sicherheitsrisiko
- **Auswirkung:** Datenbank-Verunreinigung, mögliche Performance-Degradation
- **Minderung:** Rate-Limiting via Supabase Edge Function Middleware oder Upstash Redis implementieren. Honeypot-Felder in Formularen ergänzen.

### R-005: RLS-Policy-Komplexität
- **Beschreibung:** 37 Migrationen enthalten zahlreiche RLS-Policy-Korrekturen und Security-Hardening-Schritte. Die Policy-Landschaft ist komplex und fehleranfällig bei zukünftigen Tabellen-Erstellungen.
- **Art:** Sicherheitsrisiko / Wartbarkeit
- **Auswirkung:** Fehlkonfigurierte RLS-Policies könnten Daten preisgeben
- **Minderung:** RLS-Policy-Templates standardisieren. Neue Tabellen immer gegen die 4-Policy-Vorlage (SELECT/INSERT/UPDATE/DELETE) erstellen. Regelmäßige Security-Audits.

### R-006: Skalierung der statischen Daten-Dateien
- **Beschreibung:** `blogPosts.ts` (108KB) und `analyseData.ts` (33KB) sind statische Dateien, die mit der Zeit wachsen. Bei Migration in die DB entfällt dieses Risiko.
- **Art:** Performance / Skalierung
- **Auswirkung:** Langsamere Builds, größere Bundles, längere Ladezeiten
- **Minderung:** Daten in Supabase migrieren. Bis dahin: Code-Splitting für große Daten-Dateien.

---

## 7. Erledigte Probleme (Gelöst)

### G-001: GraphQL-Schema-Exposure auf sensiblen Tabellen
- **Datum:** 2026-04-28 bis 2026-07-09 (verteilt über mehrere Migrationen)
- **Ursache:** Standardmäßig expose Supabase Tabellen über das GraphQL-Schema, was ungewollten Lesezugriff ermöglichte.
- **Lösung:** Mehrere Migrationen deaktivieren GraphQL-Exposure auf betroffenen Tabellen via `@graphql({"expose": false})`.
- **Geänderte Dateien:**
  - `supabase/migrations/20260428142912_disable_graphql_schema_exposure.sql`
  - `supabase/migrations/20260628124013_hide_blog_posts_from_graphql_schema.sql`
  - `supabase/migrations/20260708165844_20260708_fix_rls_and_blog_graphql_exposure.sql`
  - `supabase/migrations/20260708172942_fix_published_blog_posts_view_security.sql`
  - `supabase/migrations/20260708174034_fix_blog_graphql_exposure_via_rpc.sql`
  - `supabase/migrations/20260717173815_hide_seminar_dates_from_graphql_schema.sql`

### G-002: Exzessive Table-Grants
- **Datum:** 2026-04-28 bis 2026-04-29
- **Ursache:** Tabellen hatten übermäßig breite GRANT-Berechtigungen, die nicht mit den RLS-Policies übereinstimmten.
- **Lösung:** Grants widerrufen und auf das notwendige Maß reduziert.
- **Geänderte Dateien:**
  - `supabase/migrations/20260428114514_revoke_excessive_table_grants_and_secure_function.sql`
  - `supabase/migrations/20260428114542_revoke_execute_handle_new_admin_user_from_public.sql`
  - `supabase/migrations/20260428141333_fix_social_tables_excessive_grants.sql`
  - `supabase/migrations/20260428144224_revoke_authenticated_access_all_tables.sql`
  - `supabase/migrations/20260508130408_revoke_authenticated_select_on_chat_sessions.sql`

### G-003: RLS-Policy-Sicherheitslücken
- **Datum:** 2026-01-27 bis 2026-04-14
- **Ursache:** Ursprüngliche RLS-Policies hatten Lücken — fehlende Policies, `USING (true)` als Shortcut, fehlende Indizes.
- **Lösung:** RLS-Policies korrigiert, Foreign-Key-Indizes hinzugefügt, Performance-Optimierung.
- **Geänderte Dateien:**
  - `supabase/migrations/20260127124029_fix_security_issues.sql`
  - `supabase/migrations/20260127152014_fix_rls_policy_security.sql`
  - `supabase/migrations/20260214120632_fix_inhouse_seminar_requests_rls_security.sql`
  - `supabase/migrations/20260414133418_fix_rls_performance_and_security.sql`
  - `supabase/migrations/20260414135458_fix_security_issues_rls_and_indexes.sql`
  - `supabase/migrations/20260414140125_add_foreign_key_indexes.sql`
  - `supabase/migrations/20260414144643_add_foreign_key_indexes_activity_log_media_files.sql`

### G-004: HEIC/HEIF-MIME-Type-Support fehlte im Media-Bucket
- **Datum:** 2026-05-06
- **Ursache:** Supabase Storage-Bucket akzeptierte keine HEIC/HEIF-Bilder (Apple-Format).
- **Lösung:** MIME-Types in der Bucket-Konfiguration ergänzt.
- **Geänderte Dateien:**
  - `supabase/migrations/20260506142318_add_heic_heif_mime_types_to_media_bucket.sql`

### G-005: Blog-Posts-RPC-Sicherheitslücke
- **Datum:** 2026-07-08 bis 2026-07-09
- **Ursache:** Eine RPC-Funktion für veröffentlichte Blog-Posts umging RLS-Policies.
- **Lösung:** View-Security korrigiert, RPC entfernt und durch sichere View ersetzt.
- **Geänderte Dateien:**
  - `supabase/migrations/20260708172942_fix_published_blog_posts_view_security.sql`
  - `supabase/migrations/20260708174034_fix_blog_graphql_exposure_via_rpc.sql`
  - `supabase/migrations/20260709122100_drop_get_published_blog_posts_rpc.sql`

### G-006: Widerruf-Formular-Select-Grant fehlte
- **Datum:** 2026-07-08
- **Ursache:** `widerruf_submissions`-Tabelle hatte keinen SELECT-Grant für die Edge Function.
- **Lösung:** Grant korrigiert.
- **Geänderte Dateien:**
  - `supabase/migrations/20260708165232_20260708_fix_widerruf_submissions_select_grant.sql`

### G-007: XSS-Sicherheitslücke in Blog-Komponenten geschlossen

- **Datum:** 2026-07-23
- **Ursache:** Blog-Artikel-Inhalte wurden via `dangerouslySetInnerHTML` ohne Sanitization gerendert (Stored XSS-Risiko).
- **Lösung:** DOMPurify integriert. Neue Hilfsfunktion `src/lib/sanitizeHtml.ts` bereinigt alle HTML-Strings vor dem Rendern. In `BlogTemplate.tsx` und `BlogArticle.tsx` angewendet.
- **Geänderte Dateien:**
  - `src/lib/sanitizeHtml.ts` (neu erstellt)
  - `src/components/BlogTemplate.tsx`
  - `src/pages/BlogArticle.tsx`

### G-008: ErrorBoundary für lazy-loaded Seiten hinzugefügt

- **Datum:** 2026-07-23
- **Ursache:** Keine ErrorBoundary-Komponente vorhanden — Render-Fehler in lazy-loaded Seiten führten zu weißer Seite.
- **Lösung:** `ErrorBoundary`-Komponente erstellt und um beide Suspense-Wrapper (öffentlich + Admin) in `App.tsx` gelegt.
- **Geänderte Dateien:**
  - `src/components/ErrorBoundary.tsx` (neu erstellt)
  - `src/App.tsx`

### G-009: ThemeArea.tsx DB-First-Pattern und Bug-Fix

- **Datum:** 2026-07-23
- **Ursache:** `ThemeArea.tsx` nutzte nur statische Seminardaten. Zusätzlich war ein `theme === 'brandschutz'`-Vergleich innerhalb eines `theme === 'veranstaltungssicherheit'`-Blocks unreachable (TypeScript-Fehler).
- **Lösung:** DB-First-Pattern mit statischem Fallback hinzugefügt (analog zu `Seminare.tsx`). Unreachable Vergleich korrigiert.
- **Geänderte Dateien:**
  - `src/pages/ThemeArea.tsx`
