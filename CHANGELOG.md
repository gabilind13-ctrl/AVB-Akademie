# CHANGELOG.md — AVB Akademie

> Änderungsprotokoll. Nach jeder Arbeitssitzung wird hier dokumentiert, was geändert wurde.

---

## 2026-10-27 — Seminarheft 2027 aktualisiert

- Download-Link auf der Startseite ("Zur Seminarprogramm PDF") von externer Google-Drive-URL auf lokale Datei `/Seminarprogramm_2027_A.V.B._Akademie_(2).pdf` umgestellt
- Button-Beschriftung zu "Zur Seminarprogramm PDF 2027" ergaenzt
- Textbezuege "Seminarprogramm 2026" in `analyseData.ts` und `WorkReportPDF.tsx` auf "2027" aktualisiert
- Neue PDF-Datei bereits im `public/`-Verzeichnis vorhanden

---

## 2026-10-19 — Webinar-Bild getauscht und Upload-Funktion im Admin

- Webinar-Banner-Bild in der Datenbank umgestellt: `/PHOTO-2026-09-06-14-25-27.jpg` -> `/Webinar-Erinnerung_mit_Alarmuhr_(1).png`
- Upload-Button im Webinar-Admin-Formular hinzugefuegt: Bilder koennen nun direkt aus dem Admin-Bereich hochgeladen werden (Supabase Storage, media-Bucket, Pfad `webinar/`)
- URL-Feld bleibt weiterhin manuell editierbar
- Build: 117 prerendered URLs, 0 Fehler

---

## 2026-10-18 — Startseite: UWG-Entschaerfung Teil 2

- Unbelegte Teilnehmerstimmen (M. Weber, S. Bergmann, T. Hoffmann, je 5 Sterne) entfernt
- "Brandschutzbeauftragter (TÜV)" in der Kernkompetenz-Kachel ohne "(TÜV)"
- Referenzen: "Gast-Dozent und Partner renommierter Institutionen" -> "Olaf Jastrob ist als Gast-Dozent und Referent taetig, unter anderem bei:"
- Kennzahl "DACH" -> "Bundesweit -- Seminare vor Ort & Inhouse"
- Kasten "Beratung/Erstellung/Validierung fuer:" -> "Schulungsschwerpunkte:"

---

## 2026-10-18 — Startseite: UWG-Entschaerfung der Werbeaussagen

- Statistik-Band "40% / EUR 50k+ / 100% rechtliche Absicherung" ersetzt durch drei Kacheln: Praxisnahe Praevention, Nachweisbare Qualifikation, Staerkung der Sorgfaltspflicht
- Karten "Haftungsreduzierung", "Produktivitaetssteigerung", "Kostenersparnis" sachlich umformuliert; Untertitel ohne "Compliance"-Versprechen
- Trust-Badges: "TUEV-zertifizierte Seminarinhalte" -> "Fachdozenten im Einsatz bei TUEV-Gesellschaften und anderen Bildungseinrichtungen"; "Anerkannt durch Berufsgenossenschaften" -> "Inhalte gemaess Vorgaben der Berufsgenossenschaften und Unfallkassen (DGUV)"; "DGUV-konforme Seminare" -> "Seminare auf Grundlage der DGUV-Regelwerke"
- Leistungsportfolio Olaf Jastrob: Qualitaetskontrolle/Stellungnahmen, Abnahmen/Begehungen und Sicherheitstechnischer Dienst entfernt; Hinweis auf Schwestergesellschaft BPG Jastrob ergaenzt
- "zertifiziert" entfernt (Meta-Description, FAQ-Schema, Kernkompetenzen, Dozenten, Seminarkarten-Etikett -> "Seminar mit Teilnahmebescheinigung")
- "einer der fuehrenden Anbieter" -> "spezialisierter Anbieter"; "garantieren wir" und "rechtssicheres Fachwissen" entschaerft
- Kennzahl "DACH" mit passendem Label versehen
- Interne Analyse-Daten (analyseData.ts) konsistent angepasst

---

## 2026-10-15 — Webinar-Termin aktualisiert

- Naechster Webinar-Termin von 12.10.2026 auf Montag, 26.10.2026 geaendert (Datenbank: webinar_settings)
- Hinweisleiste, Webinar-Bubble und Webinar-Modal lesen den Termin zentral aus der Datenbank -- keine Codeaenderung noetig
- Folgetermin bleibt leer

---

## 2026-10-05 — Live-Analyse repariert (1000-Zeilen-Limit), neue Traffic-Diagramme

### Ursache
- Die Edge Function `analytics` lud alle `page_views` per REST-Abfrage. PostgREST liefert maximal 1000 Zeilen pro Abfrage → bei ~11.000 Aufrufen in 30 Tagen wurden nur die ältesten 1000 gezählt (Rest des Septembers = 0).

### Aenderungen
- Neue DB-Funktion `public.get_page_view_stats(p_days int)` (Migration `create_page_view_stats_function`): komplette Aggregation in SQL (daily, today, totals, topPages, topReferrers, topSearchQueries), Tage nach `Europe/Berlin`, p_days 1–90. EXECUTE nur für `service_role`. Index `idx_page_views_created_at`.
- `supabase/functions/analytics/index.ts`: GET ruft nur noch die RPC auf (neu deployed), `Cache-Control: no-store`.
- `src/lib/analytics.ts`: `AnalyticsData` um `days`, `startDate`, `endDate` erweitert, Antwort-Validierung.
- Neu: `src/components/analyse/TrafficChart.tsx` — SVG-Kurvendiagramm mit Flächenfüllung, Y-Skala, Datumsachse, Tooltip (Maus + Touch).
- `src/pages/Analyse.tsx`: Zeitraum-Auswahl 7/30/90 Tage, zwei Diagramme (Unique Visitors / Seitenaufrufe) statt Balkendiagramm, dynamische Labels, letzte Daten bleiben bei fehlgeschlagener Aktualisierung sichtbar.
- `src/data/analyseData.ts`: `lastUpdated` 2026-10-05, Arbeitsprotokoll-Eintrag ergänzt.

### Verifikation
- RPC-Summen für 7/30/90 Tage identisch mit direkter DB-Zählung (3.067 / 11.229 / 17.089 Aufrufe).
- Build: 117 prerendered URLs, 0 Fehler.

---

## 2026-08-23 — BreadcrumbNav auf alle 25 Seminar-Detailseiten

### Aenderungen
- `BreadcrumbNav`-Komponente auf 13 verbleibende Seminar-Detailseiten eingebunden (Import + Komponente als erstes Kind in `<div className="pt-20">`)
- Lazy-Loading-Pruefung der `<img>`-Tags in 4 Dateien: Alle 4 haben nur ein Hero-Bild (above the fold) — kein `loading="lazy"` noetig

### Betroffene Dateien (neu mit BreadcrumbNav)
- `src/pages/GesundheitspraeventionDetail.tsx` — Gesundheitspraevention
- `src/pages/ProjektsteuerungDetail.tsx` — Projektsteuerung
- `src/pages/InhouseSeminarDetail.tsx` — Inhouse-Seminare
- `src/pages/KIBeauftragterDetail.tsx` — KI-Beauftragter
- `src/pages/KIDokumentationAuditsDetail.tsx` — KI Dokumentation & Audits
- `src/pages/KIEvakuierungDetail.tsx` — KI-gestuetzte Evakuierung
- `src/pages/KIGenKIBerufsalltagDetail.tsx` — Generative KI im Berufsalltag
- `src/pages/KIKompetenzBerufspraxisDetail.tsx` — KI-Kompetenz in der Berufspraxis
- `src/pages/KILehrenLernenDetail.tsx` — KI in Lehre und Lernen
- `src/pages/KISicherheitDetail.tsx` — KI & Sicherheit
- `src/pages/KIUnternehmerDetail.tsx` — KI fuer Unternehmer
- `src/pages/NotfallHochschulenDetail.tsx` — Notfallmanagement Hochschulen
- `src/pages/NotfallKMUDetail.tsx` — Notfallmanagement KMU

### Verifikation
- TypeScript-Check (`tsc --noEmit`) bestanden, 0 Fehler
- Alle 25 Detail-Seiten haben jetzt BreadcrumbNav

---

## 2026-08-23 — Article Structured Data auf createArticleSchema umgestellt

### Aenderungen
- **BlogArticle.tsx**: `createArticleSchema` zum Import hinzugefuegt; manuell erstelltes Article-Schema durch `createArticleSchema()`-Hilfsfunktion ersetzt
- **BlogArticleNew.tsx**: SEO-Imports (`updateMetaTags`, `addStructuredData`, `removeStructuredData`, `createArticleSchema`) hinzugefuegt; manuelle DOM-Manipulation fuer Meta-Tags und Schema durch Utility-Funktionen ersetzt; Variablenname-Shadowing (`articleId`) behoben

### Geaenderte Dateien
- `src/pages/BlogArticle.tsx`
- `src/pages/BlogArticleNew.tsx`

### Verifikation
- TypeScript-Kompilierung erfolgreich, 0 Fehler

---

## 2026-08-23 — Breadcrumb-Navigation auf 6 Seminar-Detailseiten

### Aenderungen
- `BreadcrumbNav`-Komponente auf 6 Seminar-Detailseiten eingebunden
- Import von `BreadcrumbNav` aus `../components/BreadcrumbNav` hinzugefuegt
- Breadcrumb als erstes Element innerhalb des `<div className="pt-20">` platziert

### Betroffene Dateien
- `src/pages/BrandschutzDetail.tsx` — Breadcrumb: Brandschutz > Brandschutz Seminare
- `src/pages/BesuchersicherheitDetail.tsx` — Breadcrumb: Veranstaltungssicherheit > Besuchersicherheit
- `src/pages/KISeminareMain.tsx` — Breadcrumb: KI Seminare
- `src/pages/ArbeitssicherheitFuehrungskraefteDetail.tsx` — Breadcrumb: Arbeitssicherheit > Arbeitssicherheit fuer Fuehrungskraefte
- `src/pages/EvakuierungshelferDetail.tsx` — Breadcrumb: Brandschutz > Evakuierung von Gebaeuden
- `src/pages/SicherheitGrossveranstaltungenDetail.tsx` — Breadcrumb: Veranstaltungssicherheit > Sicherheit bei Grossveranstaltungen

### Verifikation
- TypeScript-Check (`tsc --noEmit`) bestanden, 0 Fehler

---

## 2026-09-30 — Seminardaten-Konsolidierung und Aufräumen

### Seminardaten-Abgleich
- Seminarliste auf Startseite um 3 fehlende Seminare erweitert (Gesundheitsprävention, Erste Hilfe, Projektsteuerung)
- Route für "Arbeitssicherheit für Führungskräfte" korrigiert (verwies auf falsche Themenseite)
- Route für "Sicherheit bei Großveranstaltungen" korrigiert (konsistentes URL-Format ohne Umlaute)
- Statische Datei und Datenbank-Seminare abgeglichen — alle Seminare mit Detailseiten sind jetzt auf der Startseite sichtbar

### WorkReportPDF
- Als internes Dokumentations-Tool eingestuft — funktioniert wie vorgesehen (druckbare Berichtsseite)

### Verifikation
- npm run build erfolgreich: 116 prerendered URLs, 0 Fehler

---

## 2026-09-30 — Webinar-Datum und Kooperationsseiten-SEO

### Webinar-Datum
- Altes Webinar-Datum (28.09.2026) entfernt
- Naechster Termin: 12.10.2026 (einziger aktiver Termin)
- Folgetermin auf NULL gesetzt

### Kooperations-Seminar-Detailseiten (8 Seiten)
- BreadcrumbNav auf allen 8 Kooperations-Detailseiten (via gemeinsame Komponente)
- Course-Schema (JSON-LD) und Breadcrumb-Schema auf allen 8 Seiten
- Canonical URL fuer alle 8 Seiten
- 2 Tippfehler korrigiert: "Qualifikung" -> "Qualifikation" (KoopBrandschutz, KoopKIBeauftragter)
- SeminarDetailData-Interface um slug und category erweitert

### Geaenderte Dateien
- Datenbank: webinar_settings aktualisiert
- src/components/kooperation/KooperationSeminarDetail.tsx — Breadcrumb, Schema, Canonical
- src/pages/kooperation/Koop*.tsx (alle 8) — slug + category hinzugefuegt
- src/pages/kooperation/KoopBrandschutz.tsx — Tippfehler
- src/pages/kooperation/KoopKIBeauftragter.tsx — Tippfehler

### Verifikation
- npm run build erfolgreich: 116 prerendered URLs, 0 Fehler

---

## 2026-09-28 — Umfassende SEO-Optimierung und Google-Sichtbarkeit

### Strukturierte Daten (Schema.org)
- Course-Schema, FAQ-Schema und Breadcrumb-Schema auf allen 15 Seminar-Detailseiten
- Article-Schema auf BlogArticle.tsx und BlogArticleNew.tsx
- Bug behoben: addStructuredData wurde auf 13 Seiten mit falschem Aufrufmuster verwendet (2 Argumente statt 1)
- Neue Hilfsfunktionen in seo.ts: createCourseSchema, createFAQSchema, createBreadcrumbSchema, createArticleSchema
- olafJastrobSchema als wiederverwendbare Konstante
- og:locale Meta-Tag hinzugefuegt

### Interne Verlinkung
- RelatedSeminars-Komponente erstellt (src/components/RelatedSeminars.tsx)
- "Verwandte Seminare" Abschnitt auf allen 15 Seminar-Detailseiten mit jeweils 3 thematisch passenden Links
- BreadcrumbNav-Komponente erstellt (src/components/BreadcrumbNav.tsx)
- Sichtbare Breadcrumb-Navigation auf 25+ Detailseiten

### Startseite und Footer
- Neuer Bereich "Unsere Kernkompetenzen" auf Startseite (Arbeitssicherheit, Brandschutz, Besuchersicherheit, KI Seminare)
- Footer erweitert: Besuchersicherheit als eigener Link, neuer Abschnitt "Beliebte Seminare" mit 5 Direktlinks

### Sitemap
- Prioritaeten optimiert: Kernbereiche auf 0.9, Seminare auf 0.8
- /checklisten als neue Route (Prioritaet 0.8)
- 116 URLs insgesamt

### Performance
- loading="lazy" fuer Bilder und YouTube-iframe auf Startseite

### Geaenderte Dateien
- src/utils/seo.ts — Neue Hilfsfunktionen, og:locale
- src/components/RelatedSeminars.tsx — Neu
- src/components/BreadcrumbNav.tsx — Neu
- src/components/Footer.tsx — Erweitert
- src/pages/Home.tsx — Kernkompetenzen-Bereich, lazy loading
- 15 Seminar-Detailseiten — Schema, Breadcrumb, RelatedSeminars
- 13 weitere Detailseiten — BreadcrumbNav
- 2 Blog-Seiten — Article-Schema
- scripts/routes.js — Prioritaeten
- public/sitemap.xml — Regeneriert

### Verifikation
- npm run build erfolgreich: 116 prerendered URLs, 0 Fehler

---

## 2026-08-23 — Weitere Seminar-Buchungslinks und Umbenennung (Runde 3)

### Titel-Ergaenzungen mit Kursnummern
- "Notfall- und Krisenmanagement an Hochschulen" -> "(27030550)" hinzugefuegt, Buchungslink kurskalender?course=notfall, Datum 14.12.2026
- "Notfall- und Krisenmanagement fuer KMU" -> "(26101450)" hinzugefuegt, gleicher Buchungslink, Datum 14.10.2026

### Neue Buchungslinks
- **Fortbildungs-Seminar Brandschutzbeauftragter** -> kurskalender.avb-akademie.de/de/classes/order_by/class_startdate/ASC?course=fortbildung (Datum 10.12.-11.12.2026)
- **Befaehigte Person zur visuellen Pruefung von Sportgeraeten und Sportstaetten** -> kurskalender.avb-akademie.de/de/classes/order_by/class_startdate/ASC?course=visuelle
- **Visuelle Routine-Inspektionen fuer sichere Spielplaetze und Spielgeraete** -> kurskalender.avb-akademie.de/de/classes/order_by/class_startdate/ASC?course=visuelle

### Geaenderte Dateien
- `src/data/seminars.ts` — 2 Titel-Ergaenzungen, 5 neue Buchungslinks

### Datenbank-Aenderungen
- Titel, booking_url und next_date fuer 5 Seminare aktualisiert

### Verifikation
- `npm run build` erfolgreich: 111 prerendered URLs, 0 Fehler

---

## 2026-08-23 — Weitere Seminar-Umbenennungen und Buchungslinks (Runde 2)

### Umbenennungen
- "Besuchersicherheit und Crowd Management" -> "Sicherheitsbeauftragten: Erstschulung" (mit Buchungslink)
- "Konflikt- und Kommunikationstraining" -> "Sicherheitskonzepte erstellen nach § 43 MVStättVO" (Datum 19.01.2027)
- "Grundlagenwissen Auditierung" -> "KI in der Sicherheit" (mit Buchungslink, Datum 11.02.2027)

### Neue Buchungslinks
- **Sicherheitsbeauftragten: Erstschulung** -> kurskalender.avb-akademie.de/de/classes/order_by/class_startdate/ASC?course=sicherheit
- **Sicherheit bei Grossveranstaltungen** -> kurskalender.avb-akademie.de/de/classes/order_by/class_startdate/ASC?course=sicherheit
- **KI in der Sicherheit** -> kurskalender.avb-akademie.de/de/classes/order_by/class_startdate/ASC?course=sicherheit

### Geaenderte Dateien
- `src/data/seminars.ts` — 3 Umbenennungen, 3 neue Buchungslinks, 2 Datum-Aktualisierungen

### Datenbank-Aenderungen
- Titel, booking_url und next_date fuer 4 Seminare aktualisiert

### Verifikation
- `npm run build` erfolgreich: 111 prerendered URLs, 0 Fehler

---

## 2026-08-23 — Weitere Seminar-Buchungslinks und Umbenennung

### Umbenennung
- "Brandpräventionsbeauftragter" -> "Brandschutz beim Umgang mit Lithium-Ionen-Batterien (26101950)"
- Neues Datum: 19.10.2026
- Buchungslink: https://kurskalender.avb-akademie.de/de/classes/view/757

### Neue Buchungslinks fuer 4 weitere Seminare
- **Brandschutz- und Evakuierungshelfer** -> https://kurskalender.avb-akademie.de/de/classes?course=Brandschutz
- **KI-Beauftragter im Brandschutz und Arbeitsschutz** -> https://kurskalender.avb-akademie.de/de/classes?course=Brandschutz
- **Jährliche Wiederholungsunterweisung SAP/ZUP** -> https://kurskalender.avb-akademie.de/de/classes?course=j%C3%A4hrliche (Datum aktualisiert auf 21.09.2026)
- **Künstliche Intelligenz im Alltag nutzen – Grundlagenkurs** -> https://kurskalender.avb-akademie.de/de/classes?course=K%C3%BCnstliche%20Intelligenz%20im%20Alltag%20nutzen%20%E2%80%93%20Grundlagenkurs

### Geaenderte Dateien
- `src/data/seminars.ts` — Umbenennung, Buchungslinks und Datum-Aktualisierungen

### Datenbank-Aenderungen
- Titel, booking_url und next_date fuer 5 Seminare in der Datenbank aktualisiert

### Verifikation
- `npm run build` erfolgreich: 111 prerendered URLs, 0 Fehler

---

## 2026-08-23 — Individuelle Buchungslinks pro Seminar

### Neue Funktionen
- **Buchungslinks pro Seminar:** Der "Jetzt buchen"-Button bei offenen Seminaren oeffnet nun direkt die richtige Seite im Kurskalender (kurskalender.avb-akademie.de) in einem neuen Browser-Fenster
- Jedes Seminar kann einen individuellen Buchungslink hinterlegt bekommen
- Seminare ohne hinterlegten Link nutzen den generellen Kurskalender als Fallback
- Inhouse-Seminare bleiben unververaendert (Anfrage-Formular)

### Umbenennung
- "Basiswissen Sicherheitskonzepte erstellen nach § 43 MVStättVO" -> "Sicherheitskonzepte erstellen nach § 43 MVStättVO"

### Erste drei Seminare mit Buchungslinks
- Veranstaltungsleitung nach § 38 Abs. 2 MVStättVO (ZUP)
- Sicherheitskonzepte erstellen nach § 43 MVStättVO
- Arbeitssicherheit fuer Fuehrungskraefte

### Geaenderte Dateien
- `src/types/index.ts` — Neues optionales Feld `bookingUrl` im Seminar-Interface
- `src/data/seminars.ts` — Buchungslinks fuer 3 Seminare, Umbenennung Sicherheitskonzepte
- `src/pages/Seminare.tsx` — Button zu externem Link umgebaut, `booking_url` im DB-Select ergaenzt
- `supabase/migrations/20260823120000_add_booking_url_to_seminars.sql` (neue Migration)

### Datenbank-Aenderungen
- Neue Spalte `booking_url` (text, nullable) in `seminars`-Tabelle
- Buchungslinks und Titel-Aenderung fuer 3 Seminare in der Datenbank aktualisiert

### Verifikation
- `npm run build` erfolgreich: 111 prerendered URLs, 0 Fehler

---

## 2026-08-23 — TikTok Clip Studio: Erweiterte Upload-Funktionen

### Neue Funktionen
- **Upload-Zentrale mit Drag-and-drop**: Bilder und Videos koennen direkt per Drag-and-drop auf die Seite gezogen werden, oder per Klick ausgewaehlt werden
  - Mehrere Dateien gleichzeitig hochladbar
  - Upload-Fortschritt wird pro Datei angezeigt (Lade-Animation, Erfolg, Fehler)
  - Dateigroessenpruefung (max. 100 MB) und Formatpruefung (MP4, WebM, MOV fuer Videos; PNG, JPEG, WebP, GIF, HEIC fuer Bilder)
  - Fehlerhafte Uploads werden mit verstaendlicher Meldung angezeigt
- **Medien-Galerie**: Alle hochgeladenen Bilder und Videos fuer einen Clip werden als Vorschau-Kacheln angezeigt
  - Klick auf eine Kachel setzt das Medium als aktiven Hintergrund
  - Aktives Medium ist markiert ("Aktiv"-Badge)
  - Einzelne Medien koennen aus der Galerie entfernt werden (mit Loesch-Button bei Hover)
  - Dateiname und Groesse werden unter jeder Kachel angezeigt
  - Beim Loeschen eines Clips werden auch alle zugehoerigen Medien geloescht
- **Medienbibliothek-Auswahldialog**: Zugriff auf alle bereits in der Medienbibliothek vorhandenen Dateien
  - Filter nach Typ (Alle, Bilder, Videos) und Suchfeld fuer Dateinamen
  - Klick auf eine Datei setzt sie als Hintergrund
  - Wiederverwendung bereits hochgeladener Dateien ohne erneuten Upload
- **Auto-Save beim Upload**: Beim Hochladen von Medien wird der Clip automatisch gespeichert, falls noch nicht gespeichert
  - Hochgeladene Medien werden mit dem Clip verknuepft und beim erneuten Oeffnen wieder angezeigt
- **Automatische Hintergrund-Auswahl**: Das erste hochgeladene Medium wird automatisch als Hintergrund gesetzt, wenn noch keins vorhanden ist

### Geaenderte Dateien
- `supabase/migrations/20260823110000_create_tiktok_clip_media_table.sql` (neue Migration — Tabelle `tiktok_clip_media`)
- `supabase/functions/admin-api/index.ts` (`tiktok_clip_drafts` und `tiktok_clip_media` zu ALLOWED_TABLES hinzugefuegt, neu deployed)
- `src/admin/pages/social/TikTokClipStudio.tsx` (Upload-Zentrale, Medien-Galerie, Bibliotheks-Dialog, Auto-Save, Drag-and-drop)

### Datenbank-Aenderungen
- Neue Tabelle `tiktok_clip_media`:
  - Spalten: id, clip_id (FK zu tiktok_clip_drafts ON DELETE CASCADE), file_name, storage_path, public_url, media_type, file_size, mime_type, sort_order, created_by (FK auth.users), created_at
  - RLS aktiviert, 3 Policies (SELECT, INSERT, DELETE) fuer authenticated mit owner-scoped Zugriff (auth.uid() = created_by)
  - Indizes auf clip_id und created_by
- `tiktok_clip_drafts` und `tiktok_clip_media` zur admin-api Edge Function ALLOWED_TABLES hinzugefuegt

### Verifikation
- `npx vite build` erfolgreich: 0 Fehler
- TypeScript: Keine Fehler in TikTokClipStudio.tsx
- Edge Function admin-api erfolgreich neu deployed

---

## 2026-08-23 — TikTok Clip Studio im Memberbereich

### Neue Funktionen
- **TikTok Clip Studio** (`/admin/social/clip-studio`): Vollständiges Werkzeug zur Erstellung kurzer Social-Media-Clips für TikTok, Instagram Reels und Stories
  - Drei Vorlagen: Hook/Statement, Zitat, Ankündigung
  - Texteingabe für Überschrift, Untertext und Button-Text mit Zeichenempfehlungen
  - 6 vordefinierte AVB-Hintergrund-Vorlagen (Markenverlauf, Bühnenlook, Seminar & Wissen, Sicherheit & Krise, Dunkel, Warm Akzent)
  - Eigene Bilder oder Videos als Hintergrund hochladbar
  - Video-Bearbeitung: Start-/Endpunkt-Zuschnitt mit visuellen Reglern, Vorschau-Abspielen
  - Clip-Längen: 5, 7 oder 10 Sekunden (bei Video-Hintergrund durch Zuschnitt bestimmt)
  - Lesbarkeits-Overlay (0-80% einstellbar) für gute Lesbarkeit auf allen Hintergründen
  - Textpositionierung: Oben, Mitte, Unten
  - AVB-Logo automatisch in jedem Clip (immer sichtbar, fest positioniert)
  - Live-Vorschau im 9:16-Format mit Animations-Effekten (fade-in, slide-up, scale-in)
  - Export als WebM-Videodatei (1080×1920, 30fps) via Canvas + MediaRecorder API
  - Download der exportierten Datei direkt aus dem Studio
  - Begleittext (Caption) und Hashtags für das Posting erfassbar
  - Entwürfe speichern, laden, duplizieren und löschen
  - Publer-Integration: Fertigen Clip direkt an Publer senden mit Plattform-Auswahl und Veröffentlichungszeitpunkt
  - Status-Tracking: Entwurf → Geplant → An Publer → Online (oder Fehler)

### Geänderte Dateien
- `supabase/migrations/20260823120000_create_tiktok_clip_studio_table.sql` (neue Migration — Tabelle `tiktok_clip_drafts`)
- `src/admin/pages/social/TikTokClipStudio.tsx` (neu erstellt — vollständige Clip-Studio-Komponente)
- `src/admin/components/AdminSidebar.tsx` (neuer Sidebar-Eintrag: TikTok Clip Studio unter Social Media)
- `src/App.tsx` (neue Routen: `/admin/social/clip-studio` und `/admin/social/clip-studio/:id`)

### Datenbank-Änderungen
- Neue Tabelle `tiktok_clip_drafts`:
  - Spalten: id, title, template, headline, subtext, button_text, background_type, background_value, background_storage_path, clip_duration, overlay_opacity, text_position, caption, hashtags (text[]), scheduled_at, status, publer_job_id, publer_error, export_url, export_storage_path, created_by (FK auth.users), created_at, updated_at
  - RLS aktiviert, 4 Policies (SELECT/INSERT/UPDATE/DELETE) für authenticated mit owner-scoped Zugriff (auth.uid() = created_by)
  - created_by defaults to auth.uid() für automatische Owner-Zuweisung
  - Indizes auf created_by und status

### Technische Entscheidungen
- Export erfolgt client-side via Canvas + MediaRecorder API (keine Server-Edge-Function nötig)
- Export-Format: WebM (VP9, 8Mbps) — breit unterstützt in modernen Browsern
- Publer-Versand nutzt bestehende `publish-to-publer` Edge Function: Clip wird als `social_posts`-Eintrag mit Video-Attachment angelegt und über die vorhandene Publishing-Pipeline gesendet
- AVB-Logo wird fest im Clip gerendert (nicht optional, gemäß Anforderung)
- Video-Zuschnitt über HTML5 Video-Element mit range-Input-Reglern

### Verifikation
- `npm run build` erfolgreich: 111 prerendered URLs, 0 Fehler
- TypeScript: Keine Fehler in TikTokClipStudio.tsx

---

## 2026-08-23 — Redaktionsplan: Zeitzonen-Bug behoben

### Behobene Fehler
- **SocialKalender zeigte falschen Tag:** `toISOString().slice(0,10)` konvertierte nach UTC, wodurch in deutscher Sommerzeit (UTC+2) der "Heute"-Marker und die Beitrags-Zuordnung um einen Tag verschoben waren
- Lösung: `localDateKey()`-Funktion formatiert Datum in lokaler Zeit, alle Stellen im Kalender (Monats-, Wochen-, Tagesansicht) umgestellt

### Geänderte Dateien
- `src/admin/pages/social/SocialKalender.tsx` — `localDateKey()` und `parseLocalDate()` hinzugefügt, alle `toISOString().slice(0,10)`-Verwendungen ersetzt

---

## 2026-08-17 — Telegram Bot Reparatur: Freigabe, Admin-UI, Publer-Versand

### Behobene Fehler
- **Freigabe ohne Reply:** `/freigabe`, `/ablehnen` und `/aenderung` funktionieren jetzt auch ohne Antwort auf die Vorschau-Nachricht — der Bot nutzt den aktuellen Entwurf aus dem Konversations-Status
- **Doppelte Command-Logs:** Die doppelte Protokollierung jedes Befehls in der `telegram_commands`-Tabelle wurde entfernt
- **Admin-Seite komplett ueberarbeitet:**
  - Beitraege zur Freigabe zeigen jetzt den vollstaendigen Text, Bilder/Videos und Metadaten
  - Freigeben, Ablehnen und Text bearbeiten direkt im Admin-Bereich moeglich
  - Neuer "An Publer senden"-Button sendet freigegebene Beitraege direkt an Publer
  - Verbessertes Layout mit 4-Spalten-Statistiken und aufklappbaren Beitrags-Karten

### Geaenderte Dateien
- `supabase/functions/telegram-webhook/index.ts` — Fallback auf Konversations-Status bei fehlender Reply-Referenz, doppeltes Logging entfernt
- `src/admin/pages/social/TelegramBotAdmin.tsx` — Komplette Ueberarbeitung mit Freigabe-Workflow, Textbearbeitung und Publer-Integration

---

## 2026-08-17 — Kontakte- und Aufgabenverwaltung im Admin-Bereich

### Geänderte Dateien
- `supabase/migrations/20260817120000_create_contacts_and_tasks_tables.sql` (neue Migration)
- `supabase/functions/admin-api/index.ts` (contacts + tasks zu ALLOWED_TABLES hinzugefügt, neu deployed)
- `src/admin/pages/KontakteAdmin.tsx` (neu erstellt — Kontakte-Verwaltung mit Auto-Import)
- `src/admin/pages/AufgabenAdmin.tsx` (neu erstellt — Aufgaben-Verwaltung mit Kanban-Ansicht)
- `src/admin/components/AdminSidebar.tsx` (neue Sidebar-Einträge: Kontakte, Aufgaben)
- `src/admin/pages/Dashboard.tsx` (Vorschau-Kacheln für Kontakte/Aufgaben, Überfällig-Warnung)
- `src/App.tsx` (neue Routen: /admin/kontakte, /admin/aufgaben)

### Hinzugefügte Funktionen
- **Kontakte-Verwaltung** (`/admin/kontakte`):
  - Automatischer Import aller Kontakte aus 5 Quellen beim Öffnen der Seite: Kontaktformular, Inhouse-Anfragen, Checklisten-Leads, Affiliate-Bewerbungen, Telegram-Abonnenten
  - Duplikatserkennung anhand E-Mail-Adresse oder Telegram-ID
  - Bei Duplikaten wird die Quelle zum bestehenden Kontakt hinzugefügt (Mehrfach-Quellen)
  - Manuelle Neuanlage von Kontakten über Modal
  - Bearbeitung und Löschung bestehender Kontakte
  - Tag-Zuweisung pro Kontakt: Kunde, Interessent, Partner, Dozent, Sonstiges
  - Filter nach Quelle und Tag, Volltextsuche
  - CSV-Export der gefilterten Kontaktliste
  - Quelle wird pro Kontakt gespeichert (Nachvollziehbarkeit der Herkunft)
  - "Import aktualisieren"-Button für manuellen Re-Import
- **Aufgaben-Verwaltung** (`/admin/aufgaben`):
  - Kanban-Ansicht mit 3 Spalten: Offen, In Bearbeitung, Erledigt
  - Alternative Listenansicht mit Sortierung nach Priorität
  - Aufgaben anlegen mit Titel, Beschreibung, Fälligkeitsdatum, Priorität (hoch/mittel/niedrig)
  - Optionale Verknüpfung mit einem Kontakt (Auswahlliste aller Kontakte)
  - Status-Wechsel per Button (Kanban) oder Dropdown (Liste)
  - Filter nach Priorität und Fälligkeit (überfällig/heute/diese Woche)
  - 4 Kennzahlen-Kacheln: Offen, In Bearbeitung, Erledigt, Überfällig
  - Überfällige Aufgaben werden rot hervorgehoben
- **Dashboard-Erweiterung**:
  - Neue Kachel "Kontakte" mit Gesamtanzahl
  - Neue Kachel "Offene Aufgaben" mit Anzahl
  - Warn-Banner bei überfälligen Aufgaben mit Direkt-Link

### Datenbank-Änderungen
- Neue Tabelle `contacts` (zentrale Kontaktverwaltung)
  - Spalten: id, name, email, phone, company, notes, source, source_ref, tags (text[]), status, telegram_id, created_at, updated_at
  - RLS aktiviert, 4 Policies (SELECT/INSERT/UPDATE/DELETE) für authenticated
  - Indizes auf email, telegram_id, source
- Neue Tabelle `tasks` (Aufgabenverwaltung)
  - Spalten: id, title, description, due_date, priority, status, contact_id (FK zu contacts), created_at, updated_at
  - RLS aktiviert, 4 Policies (SELECT/INSERT/UPDATE/DELETE) für authenticated
  - Indizes auf status, due_date, contact_id, priority
  - Foreign Key: contact_id → contacts(id) ON DELETE SET NULL

### Verifikation
- `npm run build` erfolgreich: 111 prerendered URLs, 0 Fehler
- Edge Function admin-api erfolgreich neu deployed
- TypeScript: Keine neuen Fehler durch die neuen Dateien

### Entscheidungen
- Auto-Import erfolgt automatisch beim Öffnen der Kontakte-Seite (kein manueller Klick nötig)
- Duplikatserkennung per E-Mail (case-insensitive) und Telegram-ID
- Bei Duplikaten wird die Quelle ergänzt, nicht doppelt angelegt
- Aufgaben sind nicht benutzerbezogen — alle Admin-Benutzer sehen alle Aufgaben (zentrale Verwaltung)
- Tags als text[]-Spalte in der contacts-Tabelle (keine separate Tag-Tabelle nötig)
- 5 vordefinierte Tag-Kategorien mit Farbkennzeichnung

---

## 2026-08-16 — Publer Analytics-Sync mit Reichweiten-Diagramm und Top-Beiträgen

### Geänderte Dateien
- `supabase/functions/sync-publer-analytics/index.ts` (neu erstellt, neue Edge Function)
- `supabase/functions/admin-api/index.ts` (social_analytics_charts zu ALLOWED_TABLES hinzugefügt, neu deployed)
- `src/admin/pages/social/SocialAnalytics.tsx` (vollständig neu geschrieben mit Sync-Button, Reichweiten-Diagramm, Plattform-Vergleich, Top-Beiträgen)

### Hinzugefügte Funktionen
- **sync-publer-analytics Edge Function**: Ruft automatisch Post-Insights und Chart-Daten von der Publer Analytics API ab
  - Lädt pro Plattform: Reichweite, Likes, Kommentare, Shares, Klicks, Engagement-Rate (Post Insights API)
  - Lädt pro Plattform: Zeitreihen-Daten für Reichweiten-Diagramm (Charts API, 90 Tage Rückblick)
  - Unterstützt manuellen Sync (POST mit Auth) und automatischen Cron-Sync (GET ohne Auth)
  - Nutzt vorhandenen Publer API-Token aus social_api_connections, ermittelt Workspace-ID automatisch
  - Upsert-Logik: Doppelte Datenpunkte werden aktualisiert statt dupliziert
- **Analytics-Seite komplett neu**:
  - "Daten aktualisieren"-Button für manuellen Sync (mit Lade-Animation und Erfolg-/Fehler-Meldung)
  - Zeitraum-Selector (7/30/90 Tage)
  - Vier Kennkarten: Reichweite, Engagement, Klicks, Engagement-Rate
  - Reichweiten-über-Zeit Liniendiagramm (SVG-basiert, eine Linie pro Plattform mit Farb-Legende)
  - Plattform-Vergleich mit Balken und Detail-Kennzahlen
  - Top-Beiträge-Tabelle mit Ranking, Plattform-Kennzeichnung und Performance-Werten
  - Empty-State mit Hinweis und Sync-Button wenn keine Daten vorhanden

### Datenbank-Änderungen
- Neue Tabelle `social_analytics_charts` (Zeitreihen-Daten für Analytics-Diagramme)
  - Spalten: id, platform_id, chart_id, chart_title, date, value, previous_value, fetched_at
  - Unique Constraint: (platform_id, chart_id, date) für Upsert-Support
  - RLS aktiviert, Policies für authenticated (admin-only), anon-Zugriff revoked
  - Indizes auf (platform_id, chart_id, date DESC) und (date DESC)

### Verifikation
- `npm run build` erfolgreich: 111 prerendered URLs, 0 Fehler
- TypeScript: Keine Fehler in SocialAnalytics.tsx (vorbestehende Fehler in anderen Admin-Pages unverändert)
- Edge Functions erfolgreich deployed: sync-publer-analytics + admin-api

### Offen
- Cron-Schedule (alle 6 Stunden) muss in Supabase Dashboard eingerichtet werden (pg_cron oder Supabase Scheduler)
- Daten erscheinen erst nach erstem Sync-Lauf, sobald Beiträge über Publer veröffentlicht wurden

---

## 2026-08-16 — Google Drive Bildbibliothek für Telegram Bot konfiguriert

### Geänderte Dateien
- `supabase/functions/admin-api/index.ts` (app_secrets zu ALLOWED_TABLES hinzugefügt, neu deployed)
- `src/admin/pages/social/TelegramBotAdmin.tsx` (Google-Drive-Konfigurationssektion hinzugefügt)

### Hinzugefügte Funktionen
- Google-Drive-Bildbibliothek-Konfigurationsmaske im Telegram-Bot-Admin-Bereich
  - Service-Account-JSON-Eingabe mit Anzeige/Verbergen-Toggle
  - Google-Drive-Ordner-ID-Eingabe
  - Validierung der JSON-Struktur vor dem Speichern
  - Status-Anzeige (konfiguriert/unvollständig)
  - Schritt-für-Schritt-Einrichtungsanleitung inline
- `app_secrets`-Tabelle im admin-api Edge Function freigeschaltet (ALLOWED_TABLES)

### Datenbank-Änderungen
- `GOOGLE_SERVICE_ACCOUNT_JSON` in `app_secrets`-Tabelle eingetragen (2357 Zeichen)

### Verifikation
- `npm run build` erfolgreich (vite build in 15.15s)
- TypeScript: keine neuen Fehler in geänderten Dateien

### Offen
- `GOOGLE_DRIVE_FOLDER_ID` muss noch im Admin-Bereich eingegeben werden

---

## 2026-07-23 — Sicherheitslücken geschlossen und ErrorBoundary hinzugefügt

### Geänderte Dateien
- `src/lib/sanitizeHtml.ts` (neu erstellt)
- `src/components/ErrorBoundary.tsx` (neu erstellt)
- `src/components/BlogTemplate.tsx` (sanitizeHtml import + Anwendung)
- `src/pages/BlogArticle.tsx` (sanitizeHtml import + Anwendung)
- `src/pages/ThemeArea.tsx` (DB-First-Pattern, Bug-Fix, ungenutzter Import entfernt)
- `src/pages/Seminare.tsx` (console.log entfernt)
- `src/App.tsx` (ErrorBoundary um Suspense-Wrapper gelegt)
- `package.json` (dompurify + @types/dompurify hinzugefügt)
- `KNOWN_ISSUES.md` (K-001, B-003, B-004, T-002 aktualisiert; G-007, G-008, G-009 hinzugefügt)
- `TASKS.md` (Erledigte Aufgaben aktualisiert)
- `NEXT_SESSION.md` (aktualisiert)

### Behobene Fehler
- 🔴 **K-001 gelöst:** XSS-Sicherheitslücke in Blog-Komponenten geschlossen — DOMPurify sanitizes alle HTML-Inhalte vor dem Rendern
- 🟠 **B-003 teilweise gelöst:** `console.log` in `Seminare.tsx` entfernt (`webVitals.ts` verbleibt bewusst)
- 🟠 **B-004 gelöst:** ErrorBoundary-Komponente erstellt und um beide Suspense-Wrapper (öffentlich + Admin) gelegt — Render-Fehler zeigen nun eine Fehlerseite statt weißer Seite
- 🟠 **ThemeArea.tsx Bug-Fix:** Unreachable `theme === 'brandschutz'`-Vergleich innerhalb `theme === 'veranstaltungssicherheit'`-Block korrigiert
- 🟠 **ThemeArea.tsx:** Ungenutzter `Heart`-Import entfernt

### Hinzugefügte Funktionen
- `src/lib/sanitizeHtml.ts`: Zentrale HTML-Sanitization mit DOMPurify (erlaubt nur sichere Tags: h2-h6, p, ul/ol/li, a, img, blockquote, pre, code, table, div, span, figure)
- `src/components/ErrorBoundary.tsx`: React ErrorBoundary mit benutzerfreundlicher Fehlerseite und „Zurück zur Startseite"-Button, Varianten für Website und Admin

### Verbesserungen
- `ThemeArea.tsx` nutzt nun DB-First-Pattern mit statischem Fallback (analog zu `Seminare.tsx`) — lädt Seminardaten primär aus Supabase
- `Seminare.tsx`: `handleSuccess` ruft nun `closeModal()` auf statt nur zu loggen

### Verifikation
- `npm run build` erfolgreich: 110 prerendered URLs, 0 Fehler
- TypeScript: Keine Fehler in geänderten Dateien (vorbestehende Fehler in Admin-Pages unverändert)

### Entscheidungen
- DOMPurify als Sanitization-Bibliothek gewählt (Industry-Standard, klein, gut gewartet)
- ErrorBoundary zeigt Fehlermeldung und Zurück-Button — keine automatische Wiederherstellung
- `allSeminars.ts` und `inhouseSeminarCatalog.ts` bleiben statisch (unterschiedliche Datenstruktur, bewusste Trennung)
- `webVitals.ts` console.log verbleibt als bewusste Performance-Monitoring-Maßnahme

### Zusammenfassung der Sitzung
- 3 der wichtigsten Aufgaben aus KNOWN_ISSUES.md bearbeitet: XSS-Sicherheitslücke, ErrorBoundary, Seminardaten-Konsolidierung
- 2 neue Dateien, 6 geänderte Dateien, 1 neue Abhängigkeit (dompurify)
- Build fehlerfrei, alle 110 Seiten prerendered

---

## 2026-07-23 — KNOWN_ISSUES.md erstellt und Codebase-Audit durchgeführt

### Geänderte Dateien
- `KNOWN_ISSUES.md` (neu erstellt, 21KB)
- `AI_RULES.md` (aktualisiert)
- `TASKS.md` (aktualisiert)
- `CHANGELOG.md` (aktualisiert)

### Hinzugefügte Funktionen
- `KNOWN_ISSUES.md` als zentrale Fehler- und Verbesserungsdokumentation mit 7 Bereichen: Kritische Fehler, Bekannte Bugs, Technische Schulden, Verbesserungsideen, Wiederkehrende Fehler, Risiken, Erledigte Probleme
- `AI_RULES.md`: `KNOWN_ISSUES.md` in Pflichtlese- und Aktualisierungsworkflow aufgenommen
- Codebase-Audit durchgeführt: TODO/FIXME-Suche, Dateigrößen-Analyse, Sicherheits-Pattern-Scan, Accessibility-Check, Test-Infrastruktur-Check

### Identifizierte Probleme
- 🔴 K-001: `dangerouslySetInnerHTML` ohne Sanitization in BlogTemplate.tsx und BlogArticle.tsx (XSS-Risiko)
- 🟠 B-001: `as any`-Type-Casts in Analyse.tsx (6 Stellen)
- 🟠 B-002: Index-basierte React-Keys in ~10 Komponenten (Rendering-Risiko)
- 🟠 B-003: `console.log` in Produktionscode (2 Stellen)
- 🟠 B-004: Kein ErrorBoundary für lazy-loaded Seiten (Absturzrisiko)
- ⚙️ T-001: 10 Dateien über 30KB (größte: blogPosts.ts 108KB, ContentAssistent.tsx 74KB)
- ⚙️ T-002: Seminardaten parallel statisch und in DB
- ⚙️ T-003: Keine Test-Infrastruktur
- ⚙️ T-004: Direkte fetch-Aufrufe statt zentraler API-Client
- ⚙️ T-005: Custom Routing statt react-router (öffentliche Website)
- ⚙️ T-006: Fehlende Barrierefreiheit
- 💡 10 Verbesserungsideen (V-001 bis V-010)
- 6 Risiken (R-001 bis R-006)
- 6 gelöste Probleme dokumentiert (G-001 bis G-006)

### Entscheidungen
- `KNOWN_ISSUES.md` wird ab sofort am Anfang jeder Sitzung gelesen und am Ende aktualisiert
- Gelöste Probleme werden nicht gelöscht, sondern mit Datum in "Gelöst"-Abschnitt verschoben
- Vor jeder Codeänderung wird geprüft, ob bekannte Probleme den betroffenen Bereich betreffen

### Zusammenfassung der Sitzung
- Vollständiger Codebase-Audit: Sicherheits-Patterns, Dateigrößen, TypeScript-Schwächen, Accessibility, Test-Abdeckung
- 1 kritisches Sicherheitsproblem, 4 Bugs, 6 technische Schulden, 10 Verbesserungsideen, 6 Risiken und 6 gelöste Probleme dokumentiert
- Dokumentationssystem nun vollständig: 6 Dateien (PROJECT_CONTEXT, TASKS, CHANGELOG, NEXT_SESSION, KNOWN_ISSUES, AI_RULES)

---

## 2026-07-23 — NEXT_SESSION.md erstellt und Dokumentationsworkflow erweitert

### Geänderte Dateien
- `NEXT_SESSION.md` (neu erstellt, 12KB)
- `AI_RULES.md` (aktualisiert)
- `TASKS.md` (aktualisiert)
- `CHANGELOG.md` (aktualisiert)

### Hinzugefügte Funktionen
- `NEXT_SESSION.md` als automatische Sitzungsübergabe mit 8 Bereichen: Projektstatus, Heute erledigt, Offene Aufgaben (mit Prioritäten), Nächster Entwicklungsschritt, Betroffene Dateien, Bekannte Probleme, Architekturänderungen, Hinweise für nächste KI-Sitzung
- `AI_RULES.md`: `NEXT_SESSION.md` in Pflichtlese- und Aktualisierungsworkflow aufgenommen

### Behobene Fehler
- (keine Code-Fehler)

### Entscheidungen
- `NEXT_SESSION.md` wird ab sofort am Ende jeder Sitzung aktualisiert
- Lesereihenfolge zu Sitzungsbeginn: PROJECT_CONTEXT → TASKS → CHANGELOG → NEXT_SESSION → AI_RULES → Codeanalyse
- Offene Aufgaben werden mit Prioritäten gekennzeichnet: 🔴 Kritisch, 🟠 Wichtig, 🟢 Optional

### Zusammenfassung der Sitzung
- `NEXT_SESSION.md` mit allen 8 geforderten Bereichen erstellt
- `AI_RULES.md` um `NEXT_SESSION.md` im Pflichtworkflow ergänzt
- Dokumentationssystem nun vollständig: 5 Dateien (PROJECT_CONTEXT, TASKS, CHANGELOG, NEXT_SESSION, AI_RULES)

---

## 2026-07-23 — Dokumentation gegen Codebasis verifiziert und korrigiert

### Geänderte Dateien
- `PROJECT_CONTEXT.md` (umfassend aktualisiert)
- `TASKS.md` (aktualisiert)
- `CHANGELOG.md` (aktualisiert)

### Hinzugefügte Funktionen
- (keine Code-Änderungen)

### Behobene Fehler
- PROJECT_CONTEXT.md: Routing-Architektur korrigiert — öffentliche Website verwendet custom switch-basiertes Routing, nicht react-router (nur Admin nutzt react-router)
- Seitenzahl von ~50 auf ~60 korrigiert (63 Page-Dateien gezählt)
- Blog-System: Klarstellung dass BlogArticle/BlogArticleNew/BlogExamples KEINE Duplikate sind, sondern unterschiedliche Use-Cases abdecken
- Daten-Dateien: Von 3 auf 10 korrigiert (analyseData, blogArticles, inhouseSeminarCatalog, kooperationFelsteadSeminare, kooperationGruenbergSeminare, kooperationSeminare, regionalData ergänzt)
- Komponenten: Von 5 auf 36 korrigiert (geo/, kooperation/, kooperation-felstead/, kooperation-gruenberg/ ergänzt)
- Tabellenzahl von ~20 auf ~24 korrigiert
- Tailwind-Theme (primary/accent-Ramps, Inter-Font, Custom-Animationen) dokumentiert
- Environment Variables (VITE_SUPABASE_URL, VITE_SUPABASE_ANON_KEY) dokumentiert
- Auth-Berechtigungs-Matrix (canEdit/canPublish/canApprove pro Rolle) hinzugefügt
- SEO-Infrastruktur (Sitemap-Generierung, Puppeteer-Prerendering, robots.txt) dokumentiert
- adminDb Custom Query-Builder Pattern (thenable, SelectBuilder/InsertBuilder/etc.) beschrieben
- Vite-Konfiguration (optimizeDeps.exclude lucide-react) dokumentiert
- Web-Vitals-Monitoring und IntersectionObserver-Scroll-Animationen dokumentiert

### Entscheidungen
- Blog-Seiten (BlogArticle/BlogArticleNew/BlogExamples) als bewusst unterschiedliche Use-Cases dokumentiert, nicht als Duplikate
- Custom Routing als historische Entscheidung dokumentiert (vermutlich für Prerendering-Kompatibilität)

### Zusammenfassung der Sitzung
- Vollständige Verifikation aller vier Dokumentationsdateien gegen den tatsächlichen Quellcode
- 10 inhaltliche Lücken und Korrekturen identifiziert und behoben
- PROJECT_CONTEXT.md von 12KB auf 21KB erweitert mit vollständiger Architektur-, Komponenten- und Datenbeschreibung

---

## 2026-07-22 — Projekt-Dokumentation initialisiert

### Geänderte Dateien
- `PROJECT_CONTEXT.md` (neu erstellt)
- `TASKS.md` (neu erstellt)
- `CHANGELOG.md` (neu erstellt)
- `AI_RULES.md` (neu erstellt)

### Hinzugefügte Funktionen
- Zentrale Projektdokumentation (`PROJECT_CONTEXT.md`) als alleinige Wahrheitsquelle
- Aufgabenverwaltung (`TASKS.md`) mit Status-Tracking
- Änderungsprotokoll (`CHANGELOG.md`)
- Entwicklungsregeln (`AI_RULES.md`)

### Behobene Fehler
- (keine)

### Entscheidungen
- Vier Dokumentationsdateien werden ab sofort verpflichtend bei jeder Sitzung gelesen und aktualisiert
- Bei Abweichungen zwischen Chatverlauf und Dokumentation/Code hat die Dokumentation Vorrang
- Dokumentation ist dauerhafter Teil des Projekts

### Zusammenfassung der Sitzung
- Vollständige Codebase-Analyse durchgeführt (React/Supabase-Architektur, ~50 öffentliche Seiten, Admin-CMS, Social-Media-Modul, 11 Edge Functions, ~20 DB-Tabellen, 37 Migrationen)
- Alle Erkenntnisse in `PROJECT_CONTEXT.md` strukturiert dokumentiert
- Projekt-Dokumentationssystem etabliert (PROJECT_CONTEXT, TASKS, CHANGELOG, AI_RULES)
