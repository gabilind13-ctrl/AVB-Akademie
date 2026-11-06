# PROJECT_CONTEXT.md — AVB Akademie

> Zentrale Wissensquelle des Projekts. Diese Datei wird bei jeder relevanten Änderung aktualisiert und bildet die Grundlage für alle zukünftigen Entwicklungsschritte. Wenn Chatverlauf und diese Datei voneinander abweichen, haben der aktuelle Code und diese Dokumentation Vorrang.

---

## 1. Projektziel

Die AVB Akademie ist eine deutschsprachige Webplattform für Sicherheits- und Veranstaltungsschulungen. Ziel ist es, Seminare online zu präsentieren, Anfragen und Buchungen entgegenzunehmen, Inhalte über ein CMS zu verwalten und Social-Media-Aktivitäten zentral zu steuern.

---

## 2. Projektübersicht

Die Plattform besteht aus zwei Hauptbereichen:

- **Öffentliche Website** (~60 Seiten): Marketing-Seiten, Seminarkatalog mit Detailseiten, Blog-System, Kooperationen, Partnerprogramm, Kontakt- und Widerrufsformulare, Terminkalender, regionale SEO-Landingpages, Wissensdatenbank, rechtliche Pflichtseiten, KI-Chatbot, Cookie-Banner.
- **Admin-CMS** (`/admin/*`): Vollwertiges Redaktionssystem mit drei Sektionen — Website & Inhalte (9 Seiten), Social Media (10 Seiten), System (3 Seiten) + Login-Seite.

---

## 3. Systemarchitektur

| Schicht | Technologie |
|---|---|
| Frontend | React 18 + TypeScript + Vite |
| Routing (öffentlich) | Custom `pathToPage`/`handleNavigate`-Pattern mit `switch`-Statement in `App.tsx` — NICHT react-router Routes |
| Routing (Admin) | react-router-dom v6 (`<Routes>`/`<Route>` mit nested routes, `<Outlet>`, `NavLink`) |
| State Management | React Context (`AuthContext` für Admin-Auth) + lokaler Komponenten-State (`useState`/`useEffect`). Kein Redux/Zustand. |
| Lazy Loading | Alle Seiten werden mit `React.lazy()` + `Suspense` geladen (`PageLoader`-Fallback) |
| Styling | Tailwind CSS mit Custom-Theme (primary/accent-Ramps, Inter-Font, Custom-Animationen), lucide-react Icons |
| Backend | Supabase (Postgres + Auth + Storage + Edge Functions/Deno) |
| Admin-Datenzugriff | Custom `adminDb`-Client (Supabase-mimicking Query-Builder mit SelectBuilder/InsertBuilder/UpsertBuilder/UpdateBuilder/DeleteBuilder, thenable Pattern) → `admin-api` Edge Function (service_role) |
| Auth | Supabase email/password, `admin_profiles`-Tabelle mit Rollen (admin/editor/content_creator/viewer) |
| SEO/Prerender | `scripts/generate-sitemap.js` (Sitemap), `scripts/prerender.js` (Puppeteer-basiertes Prerendering von 110 URLs), `scripts/routes.js` |
| Monitoring | web-vitals (initWebVitals in main.tsx) |

### Routing-Architektur (Detail)

Die öffentliche Website verwendet KEINE react-router-Routen. Stattdessen:
- `pathToPage(pathname)` wandelt URL-Pfade in Page-IDs um (z.B. `/seminare` → `seminare`)
- `handleNavigate(page)` navigiert via `useNavigate()` und setzt den State
- `renderPage()` verwendet ein `switch`-Statement über `currentPage`, um die entsprechende Komponente zu rendern
- Dynamische Routen: `inhouse-seminar-*` → InhouseSeminarDetail (slug extrahiert), `blog-*` → BlogArticle (articleId extrahiert)
- `IntersectionObserver` für Scroll-Animationen (`.animate-on-scroll` → `.visible`)

Admin-Bereich verwendet korrekt react-router-dom mit nested Routes:
- `/admin/login` → AdminLogin (ungeschützt)
- `/admin/*` → ProtectedRoute → AdminLayout (mit `<Outlet>`)
- `/admin` → Dashboard (index)
- `/admin/seminare`, `/admin/blog`, etc.
- `/admin/social`, `/admin/social/beitraege`, etc.

### Datenfluss
- Öffentliche Formulare → Edge Functions → Supabase-Tabellen (anon INSERT, service_role/admin SELECT)
- Admin-CRUD → `adminDb` (custom Query-Builder) → `admin-api` Edge Function (JWT-validiert, service_role) → Supabase
- Blog/Termine → direkte Supabase-Queries (anon SELECT auf veröffentlichte Daten)
- `adminSupabase.ts` stellt `signIn`, `signOut`, `signUp`, `resetPassword`, `getAdminProfile`, `logActivity` bereit

### Auth-Berechtigungen

| Rolle | canEdit | canPublish | canApprove |
|---|---|---|---|
| admin | ja | ja | ja |
| editor | ja | ja | ja |
| content_creator | ja | nein | nein |
| viewer | nein | nein | nein |

---

## 4. Verwendete Technologien

| Kategorie | Technologie |
|---|---|
| Framework | React 18.3 |
| Build-Tool | Vite 5.4 |
| Sprache | TypeScript 5.5 |
| Routing | react-router-dom 6.30 (nur Admin) |
| Styling | Tailwind CSS 3.4 |
| Icons | lucide-react 0.344 |
| Backend | @supabase/supabase-js 2.57 |
| Edge Runtime | Deno (Supabase Edge Functions) |
| PDF/Prerender | puppeteer 25.1 (devDependency, für prerender.js) |
| Monitoring | web-vitals 5.3 |

### Environment Variables
- `VITE_SUPABASE_URL` — Supabase-Projekt-URL
- `VITE_SUPABASE_ANON_KEY` — Supabase Anon-Key
- (Service-Role-Key serverseitig in Edge Functions, nicht im Frontend)

### Tailwind-Theme
- **Primary-Ramp**: Blue-Gray (#f2f7f9 bis #1a242e, 11 Stufen) — Hauptfarbe
- **Accent-Ramp**: Orange (#fef6ee bis #41130a, 11 Stufen) — Akzentfarbe
- **Font**: Inter (300-700), system-ui fallback
- **Custom-Animationen**: fade-in, slide-up, slide-in-right, scale-in, marquee
- **Custom-Utilities** (in index.css): animate-on-scroll, hover-lift, icon-bounce, icon-pulse, glow-on-hover, fade-in-up, stagger-1 bis 4, shimmer, scale-on-hover, icon-float, text-glow, gradient-shift, slide-in-left, slide-in-right, mobile-text-responsive, mobile-heading-fit, text-pulse, menu-item-hover

### Vite-Konfiguration
- `optimizeDeps.exclude: ['lucide-react']` — verhindert Pre-Bundling von lucide-react

---

## 5. Projektstruktur und Dateiaufbau

```
project/
├── src/
│   ├── App.tsx                 # Alle Routen: Custom switch-Routing (öffentlich) + react-router (Admin)
│   ├── main.tsx                # Entry, BrowserRouter, initWebVitals
│   ├── index.css               # Globale Styles + Tailwind + Custom-Utilities
│   ├── components/             # 36 Komponenten in 4 Verzeichnissen
│   │   ├── Header.tsx          # Navigations-Header
│   │   ├── Footer.tsx          # Footer
│   │   ├── ChatbotWidget.tsx   # KI-Chatbot (OpenAI thread-basiert)
│   │   ├── CookieBanner.tsx    # DSGVO-Cookie-Banner
│   │   ├── Modal.tsx           # Wiederverwendbare Modal-Komponente
│   │   ├── BlogTemplate.tsx    # Blog-Seiten-Template
│   │   ├── InhouseSeminarRequestForm.tsx # Inhouse-Seminar-Anfrageformular
│   │   ├── geo/                # 6 SEO/Geo-Komponenten (ComparisonTable, DefinitionBox, ExpertQuote, LastUpdated, SourcesSection, StatBox)
│   │   ├── kooperation/        # 7 Komponenten (KooperationHero, KooperationAbout, KooperationPortfolio, KooperationSeminareGrid, KooperationTrustBar, KooperationValueBanner, KooperationCTA, KooperationSeminarDetail)
│   │   ├── kooperation-felstead/ # 7 Felstead-spezifische Komponenten
│   │   └── kooperation-gruenberg/ # 7 Gruenberg-spezifische Komponenten
│   ├── pages/                  # 63 öffentliche Seiten
│   │   ├── Home.tsx            # Hauptseite (53KB)
│   │   ├── LandingPage.tsx     # Landing-Page
│   │   ├── Seminare.tsx        # Seminarkatalog
│   │   ├── InhouseSeminare.tsx # Inhouse-Seminar-Übersicht
│   │   ├── InhouseSeminarDetail.tsx # Dynamische Inhouse-Seminar-Detailseite
│   │   ├── Blog.tsx            # Blog-Übersicht
│   │   ├── BlogArticle.tsx     # Blog-Artikel nach ID (dynamisch via blog-* routing)
│   │   ├── BlogArticleNew.tsx  # Spezifische neue Blog-Artikel (articleId-basiert)
│   │   ├── BlogExamples.tsx    # Blog-Beispiel-Sammlungen nach Typ (articleType-basiert)
│   │   ├── Terminkalender.tsx  # Seminar-Terminkalender
│   │   ├── RegionalLanding.tsx # Regionale SEO-Landingpages (berlin, muenchen, hamburg, koeln, frankfurt, duesseldorf)
│   │   ├── ThemeArea.tsx       # Themenbereiche (veranstaltungssicherheit, brandschutz, arbeitssicherheit)
│   │   ├── Pillar.tsx          # Pfeiler-Seiten (praxis, expertise, engagement, flexibilitaet)
│   │   ├── Wissensdatenbank.tsx # Wissensdatenbank
│   │   ├── Analyse.tsx         # Analyse-Seite (50KB)
│   │   ├── Kontakt.tsx         # Kontaktformular
│   │   ├── Widerruf.tsx        # Widerrufsformular
│   │   ├── Partnerprogramm.tsx # Affiliate-Programm
│   │   ├── Kooperationen.tsx  # Kooperations-Übersicht
│   │   ├── KooperationAykeBroeker.tsx # Partner-Seite
│   │   ├── KooperationMarkFelstead.tsx # Partner-Seite
│   │   ├── KooperationSilvioGruenberg.tsx # Partner-Seite
│   │   ├── kooperation/        # 10 Kooperations-Seminar-Detailseiten (KoopVeranstaltungssicherheit, KoopBrandschutz, etc.)
│   │   ├── *Detail.tsx        # ~20 Seminar-Detailseiten (Brandschutz, ErsteHilfe, KI-Seminare, Krisenmanagement, etc.)
│   │   ├── WorkReportPDF.tsx   # PDF-Berichtsseite
│   │   ├── Datenschutz.tsx    # Datenschutzerklärung
│   │   └── Impressum.tsx      # Impressum
│   ├── data/                   # 10 statische Daten-Dateien
│   │   ├── seminars.ts         # Seminar-Katalog (12KB)
│   │   ├── allSeminars.ts      # Erweiterter Seminar-Katalog (6KB)
│   │   ├── inhouseSeminarCatalog.ts # Inhouse-Seminar-Katalog (16KB)
│   │   ├── blogPosts.ts        # Blog-Posts (108KB)
│   │   ├── blogArticles.ts     # Blog-Artikel (26KB)
│   │   ├── analyseData.ts      # Analyse-Daten (33KB)
│   │   ├── regionalData.ts     # Regionale SEO-Daten (8KB)
│   │   ├── kooperationSeminare.ts # Kooperations-Seminare (5KB)
│   │   ├── kooperationFelsteadSeminare.ts # Felstead-Seminare (4KB)
│   │   └── kooperationGruenbergSeminare.ts # Gruenberg-Seminare (5KB)
│   ├── lib/                    # Utilities
│   │   ├── supabase.ts         # Standard Supabase-Client (anon-Key, öffentliche Queries + Auth)
│   │   ├── adminSupabase.ts    # Auth-Funktionen (signIn, signOut, signUp, resetPassword, getAdminProfile, logActivity)
│   │   ├── adminDb.ts          # Custom Query-Builder für Admin-CRUD via admin-api Edge Function
│   │   ├── sanitizeFilename.ts # Dateinamen-Sanitizer
│   │   └── webVitals.ts        # Web-Vitals-Monitoring
│   ├── types/                  # index.ts (Seminar, BlogPost, CalendarEvent Interfaces)
│   ├── utils/                  # seo.ts
│   └── admin/
│       ├── context/            # AuthContext.tsx (Session, User, Profile, Rollen-Checks)
│       ├── components/         # AdminLayout, AdminSidebar, AdminHeader, ProtectedRoute
│       └── pages/             # 13 Admin-Seiten + social/ (10 Social-Media-Seiten)
├── supabase/
│   ├── functions/              # 11 Edge Functions
│   └── migrations/            # 37 SQL-Migrationen
├── public/                     # Statische Assets, Logos, PDFs, Sitemap, Robots
├── scripts/                    # generate-sitemap.js, prerender.js, routes.js
├── PROJECT_CONTEXT.md          # Diese Datei
├── TASKS.md                    # Aufgabenverwaltung
├── CHANGELOG.md                # Änderungsprotokoll
└── AI_RULES.md                 # Entwicklungsregeln
```

---

## 6. Bereits implementierte Funktionen

### Öffentliche Website
- [x] Home / LandingPage
- [x] Seminarkatalog + ~20 Seminar-Detailseiten (Brandschutz, ErsteHilfe, Evakuierung, Arbeitssicherheit, KI-Seminare, Krisenmanagement, Lithium-Ionen, Besuchersicherheit, Projektsteuerung, Veranstaltungssicherheit, etc.)
- [x] InhouseSeminare + InhouseSeminarDetail (dynamisch via `inhouse-seminar-*` Routing, Anfrageformular → `inhouse_seminar_requests`)
- [x] Blog-System:
  - Blog (Übersicht, liest aus `blog_posts`)
  - BlogArticle (individuelle Artikel via `blog-*` ID-Routing)
  - BlogArticleNew (spezifische neue Artikel mit fester articleId)
  - BlogExamples (Beispiel-Sammlungen nach articleType: gefaehrdungsbeurteilung, veranstaltungssicherheit, crowd-management)
  - Diese sind KEINE Duplikate — sie dienen unterschiedlichen Zwecken
- [x] Kooperationen + 3 Partnerseiten (Ayke Broeker, Mark Felstead, Silvio Gruenberg) + 10 Kooperations-Seminar-Detailseiten
- [x] Partnerprogramm (Affiliate-Formular → `affiliate_applications`)
- [x] Kontakt (→ `contact_form_submissions`)
- [x] Widerruf (→ `widerruf_submissions`)
- [x] Terminkalender (liest `seminar_dates`)
- [x] RegionalLanding (6 Städte: Berlin, München, Hamburg, Köln, Frankfurt, Düsseldorf)
- [x] ThemeArea (3 Themen: Veranstaltungssicherheit, Brandschutz, Arbeitssicherheit)
- [x] Pillar (4 Pfeiler: Praxis, Expertise, Engagement, Flexibilität)
- [x] Wissensdatenbank, Analyse
- [x] Rechtliche Seiten: Impressum, Datenschutz
- [x] ChatbotWidget (OpenAI thread-basiert, persistiert in `chat_sessions`)
- [x] CookieBanner (DSGVO)
- [x] WorkReportPDF (PDF-Berichtsseite)
- [x] SEO-Infrastruktur: Sitemap-Generierung, Puppeteer-Prerendering (110 URLs), robots.txt
- [x] Scroll-Animationen (IntersectionObserver, animate-on-scroll)
- [x] Web-Vitals-Monitoring

### Admin-CMS
- [x] Dashboard (Statistiken + Aktivitätslog)
- [x] SeminareAdmin (CRUD mit Suche)
- [x] BlogAdmin (CRUD mit Bild-URL, Lesezeit)
- [x] ThemenseitenAdmin (CRUD)
- [x] KooperationenAdmin (CRUD)
- [x] RegionaleAdmin (CRUD mit Keywords)
- [x] StatischeSeitenAdmin (CRUD)
- [x] AnfragenAdmin (Kontakt-, Inhouse- und Affiliate-Anfragen, Status-Wechsel)
- [x] MedienAdmin (Medienbibliothek)
- [x] BenutzerAdmin (Benutzerverwaltung)
- [x] EinstellungenAdmin (Einstellungen)
- [x] AktivitaetenAdmin (Aktivitätslog)
- [x] AdminLogin (email/password)
- [x] ProtectedRoute (Routenschutz für alle Admin-Seiten außer Login)
- [x] Collapsible Sidebar mit drei Sektionen (Website & Inhalte, Social Media, System)

### Social Media Modul
- [x] SocialDashboard (Übersicht)
- [x] BeitraegeAdmin (Post-CRUD mit Suche)
- [x] PostComposer (Post-Editor, auch für Edit via :id)
- [x] SocialKalender (Content-Kalender)
- [x] SocialMedien (Medienbibliothek)
- [x] KampagnenAdmin (Kampagnen)
- [x] AutomationenAdmin (Automatisierungsregeln)
- [x] ContentAssistent (KI-Content-Generierung, 74KB — größte Admin-Seite)
- [x] SocialAnalytics (Analytics)
- [x] PublerSettings (Publer-API-Konfiguration)
- [x] Live-Website-Analytics auf `/analyse`: Tracking via Edge Function `analytics` → Tabelle `page_views`; Auswertung per DB-Funktion `get_page_view_stats(p_days)` (SQL-Aggregation, Europe/Berlin, 1–90 Tage, nur service_role); Darstellung mit `src/components/analyse/TrafficChart.tsx`, Zeitraum 7/30/90 Tage

---

## 7. Aktuell in Entwicklung

- (keine aktiven Entwicklungsaufgaben dokumentiert)

---

## 8. Offene Aufgaben

- Datenquellen-Migration prüfen: Seminardaten existieren sowohl statisch (`seminars.ts`, `allSeminars.ts`, `inhouseSeminarCatalog.ts`) als auch in DB-Tabellen (`seminars`, `seminar_dates`) — Konsolidierung offen
- WorkReportPDF: PDF-Generierungsansatz verifizieren
- Publer-Integration: Veröffentlichung hängt von runtime-konfigurierten Credentials ab
- Kooperations-Seminar-Detailseiten (`src/pages/kooperation/`) auf Vollständigkeit prüfen

---

## 9. Bekannte Probleme

- Keine expliziten TODOs/FixMe im Code gefunden
- Social-Media-Modul funktional, aber Publer-Publishing nur bei korrekt konfigurierten `publer_account_id`-Werten und gültigen API-Credentials nutzbar
- Öffentliche Website verwendet custom Routing (switch-Statement) statt react-router — kann bei komplexen Routing-Szenarien limitierend sein

---

## 10. Wichtige Entscheidungen

- Admin-CRUD läuft über zentrale `admin-api` Edge Function (service_role) statt direkter Supabase-Calls — zusätzlicher Sicherheitslayer
- Custom `adminDb`-Client imitiert Supabase-Query-Builder-API (thenable Pattern) für nahtlose Integration
- RLS auf allen Tabellen aktiviert, GraphQL-Schema-Exposure deaktiviert
- Öffentliche Formulare nutzen Edge Functions als Zwischenschicht (CORS, Validierung)
- Statische Seminardaten parallel zu DB-Tabellen — Übergangsphase
- Öffentliche Website verwendet custom switch-basiertes Routing statt react-router (historische Entscheidung, vermutlich für einfachere Prerendering-Kompatibilität)
- Alle Seiten lazy-loaded für Code-Splitting und schnelle Initial-Load
- Puppeteer-basiertes Prerendering für SEO (110 URLs werden statisch generiert)
- Blog-System mit drei Komponententypen (BlogArticle, BlogArticleNew, BlogExamples) für unterschiedliche Use-Cases, keine Duplikate

---

## 11. Abhängigkeiten

### Runtime-Dependencies
- `@supabase/supabase-js` ^2.57.4
- `lucide-react` ^0.344.0
- `react` ^18.3.1
- `react-dom` ^18.3.1
- `react-router-dom` ^6.30.3
- `web-vitals` ^5.3.0

### DevDependencies
- `vite` ^5.4.2, `@vitejs/plugin-react` ^4.3.1
- `typescript` ^5.5.3
- `tailwindcss` ^3.4.1, `postcss` ^8.4.35, `autoprefixer` ^10.4.18
- `eslint` ^9.9.1, `typescript-eslint` ^8.3.0, `eslint-plugin-react-hooks`, `eslint-plugin-react-refresh`
- `puppeteer` ^25.1.0 (nur für prerender.js)
- `@types/react` ^18.3.5, `@types/react-dom` ^18.3.0

---

## 12. API-Integrationen

| Service | Verwendung | Edge Functions |
|---|---|---|
| **Supabase** | Primäres Backend (Postgres, Auth, Storage, Edge Functions) | — |
| **OpenAI** | KI-Chatbot (thread-basiert) + Social Content Assistant | `chatbot`, `social-ai-assistant` |
| **Publer** | Social-Media-Scheduling und Publishing | `publish-to-publer`, `publer-accounts` |

### Edge Functions (11)
| Function | Zweck |
|---|---|
| `admin-api` | Zentraler Admin-CRUD-Proxy (JWT-validiert, service_role, unterstützt query/insert/upsert/update/delete/rpc) |
| `contact-form-handler` | Kontaktformular → `contact_form_submissions` |
| `inhouse-seminar-request-handler` | Inhouse-Seminar-Anfragen (14KB — größte Edge Function) |
| `affiliate-form-handler` | Affiliate/Partner-Bewerbungen |
| `widerruf-form-handler` | Widerrufsformular |
| `chatbot` | OpenAI-Chatbot-Backend (thread-basiert) |
| `blog-posts` | Öffentliche Blog-Artikel (anon-lesbar) |
| `publish-to-publer` | Social Posts → Publer API (18KB) |
| `publer-accounts` | Verbundene Publer-Konten abrufen |
| `social-ai-assistant` | KI-Content-Generierung (23KB) |
| `sync-seminar-dates` | `next_date` aus `seminar_dates` berechnen und `seminars` aktualisieren |

---

## 13. Datenbankstruktur

### Tabellen (~24)

| Tabelle | Zweck | Zugriff |
|---|---|---|
| `contact_form_submissions` | Kontaktformular | anon INSERT, admin SELECT |
| `inhouse_seminar_requests` | Inhouse-Seminar-Anfragen | anon INSERT, admin SELECT |
| `affiliate_applications` | Partnerprogramm-Bewerbungen | service_role only (via Edge Function) |
| `widerruf_submissions` | Widerrufsformular | service_role only (via Edge Function) |
| `chat_sessions` | KI-Chatbot-Konversationen (OpenAI Thread-IDs) | authentifiziert |
| `seminar_dates` | Multi-Datum-Seminarplanung | anon SELECT |
| `seminars` | Seminarkatalog | anon SELECT |
| `blog_posts` | Blog-Artikel | anon SELECT (veröffentlicht), admin CRUD |
| `admin_profiles` | Admin-Benutzer mit Rollen | authentifiziert |
| `activity_log` | Audit-Log Admin-Aktionen | admin SELECT |
| `media_files` | Medienbibliothek-Metadaten | admin |
| `social_platforms` | Social-Media-Plattformen (mit `publer_account_id`) | admin |
| `social_posts` | Social-Media-Posts | admin |
| `social_post_platforms` | Post↔Plattform-Zuordnung | admin |
| `social_campaigns` | Kampagnen | admin |
| `social_media_attachments` | Medien-Anhänge | admin |
| `social_post_comments` | Post-Kommentare | admin |
| `social_automations` | Automatisierungsregeln | admin |
| `social_automation_log` | Automatisierungs-Log | admin |
| `social_api_connections` | API-Verbindungen | admin |
| `social_publish_log` | Veröffentlichungs-Log | admin |
| `social_analytics` | Analytics-Daten | admin |
| `social_hashtag_collections` | Hashtag-Sammlungen | admin |
| `social_content_templates` | Content-Vorlagen | admin |

### Migrationen: 37 Dateien
- Tabellen-Erstellung, RLS-Policies, Security-Hardening (Grants widerrufen, GraphQL deaktiviert), Foreign-Key-Indizes, Performance-Optimierungen
- HEIC/HEIF-MIME-Type-Support für Media-Bucket
- RPC für Chat-Message-Count-Increment
- Published-Blog-Posts-View-Security

---

## 14. Sicherheit und Berechtigungen

- **RLS** auf allen Tabellen aktiviert
- **Öffentliche Formulare**: anon INSERT only (mit `privacy_accepted = true` Checks)
- **Admin-Tabellen**: authentifiziert, Rolle über `admin_profiles` geprüft
- **Sensible Tabellen** (affiliate, widerruf): service_role only — Zugriff ausschließlich über Edge Functions
- **GraphQL-Schema-Exposure** auf den meisten Tabellen deaktiviert (`@graphql({"expose": false})`)
- **Admin-CRUD** läuft über `admin-api` Edge Function mit JWT-Validierung und service_role — anon-Key kann keine Admin-Daten lesen/schreiben
- **CORS**: Alle Edge Functions senden korrekte CORS-Header
- **ProtectedRoute**: Admin-Routen durch Auth-Kontext geschützt, Redirect zu Login bei fehlender Session
- Mehrere Security-Hardening-Migrationen (Grants widerrufen, excessive Permissions korrigiert, authenticated-Select auf chat_sessions widerrufen)

---

## 15. Nächste Entwicklungsschritte

1. Seminardaten-Konsolidierung (statisch → DB) abschließen
2. WorkReportPDF verifizieren
3. Publer-Credentials konfigurieren und Publishing testen
4. Kooperations-Seminar-Detailseiten auf Vollständigkeit prüfen
5. (Offen für neue Anforderungen)
