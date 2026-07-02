# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Session-Start Auto-Pickup (gilt für ALLE Projekte, nicht nur Dashboard)

Bei Beginn **jeder** Session prüfen, ob im Wurzelverzeichnis eines aktiven Arbeitsverzeichnisses (Primär- **oder** Zusatzverzeichnis) eine `NEXT_SESSION.md` mit einem nicht-leeren pending Prompt liegt (alles ausser `# Pending Prompt — (leer)`). Wenn ja: dem User den Titel/Kern kurz nennen und anbieten, ihn auszuführen — **nicht ungefragt starten** (per [[feedback_no_unsolicited_actions]]). Die Datei ist pro Repo eine einzige, immer-ueberschriebene Pending-Prompt-Datei (kein Stapel); die ausfuehrende Session setzt sie per Selbst-Reset-Footer am Ende zurueck. Hintergrund + Konvention: `feedback_next_session_handoff`-Memory. Bekannte Dateien aktuell: `C:\Users\manyw\Dashboard\NEXT_SESSION.md` (AI-Dashboard-Analyse-Kadenz) · `C:\Users\manyw\Desktop\ClaudeCodeZero\NEXT_SESSION.md` (Linear-Build-Achse) · `C:\Users\manyw\AppFactory\NEXT_SESSION.md` (App-Factory-Einkommens-Track, ab 2026-06-27 — siehe `project_app_factory`-Memory). Weitere Repos analog, sobald dort eine angelegt wird.

## Session-Ende: GitHub-Backup-Disziplin (gilt für ALLE Projekte)

Am Ende **jeder** Session mit substantiellen Änderungen: `git status` in den berührten Repos prüfen, Commit-Message vorschlagen, nach OK committen + pushen — **proaktiv anbieten, nicht warten bis Chris fragt**. Das schließt das Memory-Repo ein (`.claude/projects/C--Users-manyw-Desktop-ClaudeCodeZero/memory/` → `helgobert8-sketch/claude-memory`, Sammel-Commit "Session YYYY-MM-DD: …"). Hintergrund + Detail-Konvention: `feedback_git_github`-Memory. Drift-Belege: 2026-05-15 (Dashboard, 350 Files) und 2026-05-16→07-02 (claude-memory, 6 Wochen / 25 Dateien). **Achtung:** `claude-code-zero` ist PUBLIC — private Inhalte dort nie ungefragt committen.

## Projects

### Dashboard — Macro + AI Domains (C:\Users\manyw\Dashboard\)
Modular intelligence system. Same 5-layer pipeline (Profiles, Monitor, Analysis, Council, Thesenverlauf) applied to two domains: **Macro** (6 investors, established) and **AI** (11 personas, build-out in progress).

**Session-Start Auto-Pickup:** Dashboard nutzt `C:\Users\manyw\Dashboard\NEXT_SESSION.md` — siehe die generelle Regel oben unter „Session-Start Auto-Pickup (gilt für ALLE Projekte)".

**Repository layout (post-Phase-1 refactor, 2026-05-02):**
```
Dashboard/
├── scripts/                 # shared Python tools, domain-aware via --domain {macro|ai}
│   ├── paths.py             # central path resolution (DASHBOARD_ROOT/Macro|AI/...)
│   ├── youtube_monitor.py
│   ├── profile_analyzer.py
│   ├── generate_dashboard.py
│   ├── realvision_monitor.py        # Pal-only, hardcoded macro
│   └── realvision_login_setup.py    # Pal-only, hardcoded macro
├── Macro/                   # established domain — 6 investors
│   ├── Profiles/  Analyses/  YouTube/  Council/  Thesenverlauf/
│   └── dashboard.html
└── AI/                      # new domain — 11 personas (Phase 2 DONE 2026-05-03: alle 11 Profile gebaut; Mostaque-Plan ersetzt durch Sutskever)
    ├── Profiles/  Analyses/  YouTube/  Council/  Thesenverlauf/
    ├── Feeds/               # non-human reports (Epoch AI, Stanford AI Index, SemiAnalysis)
    ├── AI_Commentary/       # 7 AI systems as meta-perspective
    └── dashboard.html       # generated via generate_dashboard.py --domain ai (Layer-1-4 funktional 2026-05-10)
```

**Domain selection:** Every script accepts `--domain {macro|ai}` (default `macro` for backwards-compat). `paths.py` resolves all data paths; scripts never hardcode `Dashboard/Macro/...`. `save_analysis_direct(..., domain="ai")` lets in-session calls target either domain.

**Layer 1 — Profiles** (`<Domain>/Profiles/`)
Structured profiles: Kernthesen, Frameworks, Current Positioning, Historical Calls.
- **Macro** (6): Raoul Pal, Jordi Visser, Tom Lee, Arthur Hayes, Michael Howell, Matt Hougan
- **AI** (11 Personas, alle gebaut): **Sam Altman** (SA-01..10), **Dario Amodei** (DA-01..10), **Demis Hassabis** (DH-01..10), **Jensen Huang** (JH-01..13), **Satya Nadella** (SN-01..11), **Elon Musk** (ELN-01..11), **Mark Zuckerberg** (MZ-01..10), **Yann LeCun** (YL-01..10), **Dylan Patel** (DP-01..11 — Taiwan-These stand-alone), **Liang Wenfeng** (LW-01..10 — DeepSeek, silent-persona), **Ilya Sutskever** (IS-01..10 — SSI, silent-persona; ersetzt urspruenglich-geplanten Mostaque). Bauprotokolle + Pass-Historie → `dashboard_build_log.md`.
- Config: `<Domain>/Profiles/investors.json`
- Each investor folder may contain `kernthesen.json` — canonical list of stable thesis IDs. **Macro IDs:** `JV-01..09` (Jordi), `MH-01..07` (Howell), `AH-01..08` (Hayes), `BW-01..08` (Hougan/Bitwise), `TL-01..09` (Tom Lee), `RP-01..10` (Pal/Bittel). **AI IDs:** `SA-01..10` (Altman), `DA-01..10` (Amodei), `DH-01..10` (Hassabis), `JH-01..13` (Huang), `SN-01..11` (Nadella), `ELN-01..11` (Musk), `MZ-01..10` (Zuckerberg), `YL-01..10` (LeCun), `DP-01..11` (Patel), `LW-01..10` (Liang), `IS-01..10` (Sutskever).

**Layer 2 — YouTube Monitor** (`<Domain>/YouTube/` + `scripts/youtube_monitor.py`)
- `youtube_config.json`, `scan_history.json`, `cookies.txt`, `reports/`, `transcripts/<investor_id>/` all live under `<Domain>/YouTube/`.
- Dependencies: `scrapetube`, `youtube-transcript-api`, `yt-dlp`, `feedparser` (Python 3.14, `py` launcher on Windows). Deno runtime at `AppData/Local/Microsoft/WinGet/Packages/DenoLand.Deno_.../deno.exe`.
- **Transcript fetch** (refactor 2026-05-09): `get_transcript()` is a wrapper — primary path is `youtube-transcript-api` (no cookies, no subprocess); fallback is `yt-dlp+deno+cookies` for caption gaps. Output is wrapped via `textwrap.fill(width=100)`. **IP-Block-Caveat:** YouTube blockt Datacenter/VPS-Provider-Ranges pauschal als `IpBlocked`. Vor Scan VPN auf Nicht-VPS-Endpunkt setzen (Australien/PacketHub bewaehrt 2026-05-09); Cookies an die API helfen bei IP-Block NICHT.
- **SSL-Bypass-Toggle YT_INSECURE_SSL=1** (eingefuehrt 2026-05-16 analog zum RSS_INSECURE_SSL-Pattern Block-A 2026-05-14): Bei Corporate-AV/EDR-SSL-Interception (Windows-Defender liefert "Basic Constraints of CA cert not marked critical") setzt `youtube_monitor.py` `ssl._create_default_https_context = ssl._create_unverified_context` (urllib) **plus** `requests.Session.merge_environment_settings`-Class-Level-Patch (verify=False fuer alle Session-Instanzen — noetig weil scrapetube + youtube-transcript-api intern `requests` nutzen, nicht urllib). Aktivierung via env-Var vor Scan: `$env:YT_INSECURE_SSL="1"; py scripts\youtube_monitor.py ...`. Banner-Output bestaetigt aktiven Toggle. Distinkt zum IP-Block (das ist YouTube-Server-side); SSL-Block ist Client-side (lokales AV/EDR).
- **Macro channels:** Raoul Pal `@RaoulPalTJM` (plus search for guest appearances), Jordi Visser `@JordiVisserlabs`, Tom Lee `@Fundstrat_Direct`; Hayes/Howell/Hougan search-only.
- **AI channels (Phase 3.1 DONE 2026-05-04, hybrid model):** Lab-CEOs hybrid (channel + persona-filter): Altman `@OpenAI`, Amodei `@anthropic-ai`, Hassabis `@Google_DeepMind`, Huang `@NVIDIA`, Nadella `@Microsoft`, Zuckerberg `@Meta`. Search-only (kein eigener YT-Channel oder bewusst silent): Musk, LeCun, Patel, Liang Wenfeng, Sutskever.
- **Pre-fetch persona filter** (Phase 3.1 — AI-only feature): per-investor `channel_persona_filter` field (Liste von Tokens, case-insensitive). `apply_persona_filter()` droppt Channel-Videos ohne Persona-Token in Title/Description, **bevor** Transkripte geladen werden. Verhindert Lab-Forschungs-Noise auf @OpenAI/@DeepMind/@NVIDIA etc. Backward-compatible: leeres/fehlendes Feld → kein Filter (Macro-Configs unveraendert).
- **Transcript prefilter** (`profile_analyzer.py --pending`, post-fetch): checks each pending transcript body (post-`===`-separator) for `name_tokens` from config. Displays `[!] Name nicht im Transkript` flag — hint only, never auto-skips. Tokens chosen empirically: YouTube auto-captions misrecognize names (e.g. "Raoul" → "ral pal") and solo-channel hosts rarely say their own name, so tokens include firm/show-name variants (Pal: `ral pal`, `journeyman`; Jordi: `22v`, `turbulence model`).

**Layer 2.5 — RSS Monitor** (`<Domain>/RSS/` + `scripts/rss_monitor.py` + `scripts/html_feed_scrapers.py`, AI-only initially, Phase 3.2 Validation+HTML-Scraper DONE 2026-05-10)
- `rss_config.json` (per-Feed: id/title/url/type {lab_blog|substack|news|html_scraper} / scraper_recipe (nur html_scraper) / paywall / person_attribution + person_attribution_kind {direct|sphere_voice} / persona_filter / enabled / notes), `scan_history.json` (seen_entries dict), `reports/scan_YYYY-MM-DD_HHMM.md` — alle unter `<Domain>/RSS/`.
- Dependencies: `feedparser 6.0.12`, `beautifulsoup4 4.14` (für html_scraper), `lxml 6.0` optional (Python 3.14). **Excerpt-Trigger-Modell**: Excerpt + Titel + URL landen direkt im Markdown-Report; Volltext-Fetch on-demand DEFER (`--fetch-full`-Stub raised NotImplementedError). Paywall-Quellen (Stratechery, SemiAnalysis) bleiben auf Trigger-Niveau.
- **Subkommandos:** `--scan` (alle aktivierten Feeds), `--list-feeds` (Status), `--feed <id>` (einzelner Feed-Test, ueberspringt enabled-Filter), `--days N` (Lookback, Default 7).
- **Persona-Attribution zweistufig:** (1) `person_attribution` direktes 1:1-Mapping (SemiAnalysis → `dylan_patel` direct; Situational Awareness → `ilya_sutskever` sphere_voice), (2) Token-Match-Fallback gegen `name_tokens` aller AI-Investoren (aus `youtube_config.json`) mit Word-Boundary-Schutz für kurze Tokens (<4 Zeichen) — verhindert False-Positives wie `ssi` matchend in `expre**ssi**ve`. Multi-Hit erlaubt (z.B. SemiAnalysis-Post über DeepSeek V4 → `dylan_patel(direct) + liang_wenfeng(token_match)`).
- **HTML-Scraper-Pfad** (`html_feed_scrapers.py`, Recipe-Registry): Sites ohne RSS bekommen `type=html_scraper` + `scraper_recipe=<name>`. Jede Recipe-Funktion `scrape_<name>(url)` returnt `(entries, http_status, error)` mit feedparser-Entry-kompatiblen Dicts. Statisches HTML only, BeautifulSoup `html.parser`, kein Playwright. CSS-Modules-Hash-Klassen via Substring-Match (`class*="PublicationList"`) targetiert, damit Hash-Rebuilds nicht sofort brechen. Erste Recipe: `anthropic_news` auf https://www.anthropic.com/news (Index liefert Date+Category+Title; kein Excerpt → `summary="[Category]"` als Mini-Hint).
- **Feed-Liste (9 Feeds, 7 enabled nach Validierungs-Welle 2026-05-10):** Enabled — `openai_news` (openai.com/news/rss.xml, 947 Items), `anthropic_news` (html_scraper recipe=anthropic_news, 10 Items pro Index-Page), `deepmind_blog` (deepmind.google/blog/rss.xml), `nvidia_blog` (blogs.nvidia.com/feed/, 18 Items), `semianalysis` (substack.com/feed direkt-URL, paywall, dylan_patel direct), `stratechery` (stratechery.com/feed/, 10 Items, paywall), `chinatalk` (chinatalk.media/feed, 20 Items, persona_filter Liang-Sphaere). Disabled mit `notes` — `meta_ai_blog` (alle 6 URLs probiert: ai.meta.com/blog/rss/+rss.xml+feed=404/parse-fail; research.facebook.com/feed/=stale 2023-Items; about.fb.com/news/category/ai/feed=404 — Folge-Session-Spezialfall fuer HTML-Scraper falls Tier-A-Bedarf), `situational_awareness` (situational-awareness.ai/rss.xml liefert nur 11 statische Manifest-Pages aus Jun 2024, keine Updates; aschenbrenner.substack.com + leopoldaschenbrenner.substack.com beide leer/404 — Aschenbrenner publiziert primaer via Twitter+Podcasts, nicht web-feed-greifbar; person_attribution-Mapping vorgehalten falls Substack spaeter aktiv wird).
- **Layer-3-Integration DONE 2026-05-11:** RSS-Items via `save_analysis_direct(..., source_type="rss", source_url=..., source_title=..., source_feed_id=..., publish_date=...)` analog zu YT-Transkripten. Pending-Workflow ueber `profile_analyzer.py --domain ai --pending-rss [--investor <id>]`; Detail siehe Layer-3-Block.
- **Cowork-Pilot-Variante LIVE 2026-05-16** (parallel zum lokalen rss_monitor.py, **distinkter Layer-Cut** Cowork=Trigger / Chris+local=Analyse): taegliche Tier-A-Triage-Routine `trig_016HNbJMLvgr4sXWtmJTVg75` auf Anthropic-Cowork-Cloud (Sonnet 4.6, Cron `0 5 * * *` UTC = 7:00 Berlin). **Firecrawl-MCP als Custom-Connector** wegen Cowork-IP-/Fingerprint-403-Wall auf direkten Origin-Servern (Anthropic-Bug #52479) — Firecrawl bypassed via eigener Fetch-Infrastruktur (URL `https://mcp.firecrawl.dev/{API_KEY}/v2/mcp`, Free-Tier 500 Scrapes/Monat ~ 300 fuer 10-Feeds-Daily). 10 Feeds (7 AI: openai_news/anthropic_news/deepmind_blog/nvidia_blog/semianalysis/stratechery/chinatalk + 3 Macro-Substacks: cryptohayes/visserlabs/capitalwars), 24h-Lookback, silent-unless-escalation, Schema-Disziplin AI=K/D/N/R/V/P/E + Macro=K/D/N. **Routine-Tool-Disziplin via Prompt-Whitelist** (Routinen ignorieren die Tool-Berechtigungen der Connector-Detail-Seite — Schutz vor teuren Crawl/Research/Browser-Tools steckt im Prompt selbst, expliziter VERBOTEN-Block). Test-Run 2026-05-16 ~19:30 sauber (10/10 Feeds gescraped, 1 Tier-A = Stratechery 2026.20 Roundup als Synthese bereits-abgearbeiteter Block-N3-Substanz). Defizit-Liste fuer Iteration-2 (Persona-ID-Block-Format-Haertung + Stratechery-Author-Lane-Disziplin + Wochen-Roundup-Erkennung) gesammelt fuer Production-Beobachtungs-Phase 1-2 Wochen. Detail siehe `project_cowork_pilots`-Memory.

**Layer 3 — Profile Analysis** (`scripts/profile_analyzer.py` + `<Domain>/Analyses/`)
Compares new content against profiles. Classifies findings as:
- **K** Kontinuitaet — reaffirms existing thesis
- **D** Diskontinuitaet — revises or contradicts a position
- **N** Neues — new aspects not previously in the profile

**AI domain only — extended schema (v3, design 2026-05-02, code-implementiert 2026-05-03):**
- **R** Realisiert — self-commitment / roadmap promise delivered (e.g. "Stargate Phase 1 online")
- **V** Verfehlt/Verschoben — self-commitment broken or postponed (e.g. "GPT-5 launch underwhelm")
- **P** Prediction — capability/world prediction came true or was falsified (e.g. "agents-in-workforce 2025 partially-met")
- **E** External-Pressure — third-party signal that loads/validates person/firm (e.g. "Sutskever departure", "Musk lawsuit")

Macro stays restricted to K/D/N. Rationale: AI-CEO material has roadmap-promises, capability-predictions, strategic-moves-as-implementation, and external-counterparty-pressure that K/D/N can't cleanly track. See `feedback_ai_finding_schema` memory.

Analysis workflow: `--pending` -> read profile + transcript in session -> produce analysis -> `save_analysis_direct()` to log.

Analysis schema v2: each finding carries `type` + `thesis_id` (from `kernthesen.json`) + `finding` text. `save_analysis_direct(findings=[...], publish_date=..., domain="macro")` auto-fetches `publish_date` from YouTube if not passed. Legacy `classifications={}` dict still accepted for entries without thesis_id mapping. `load_kernthesen(investor_id)` loads the canonical thesis list for validation.

**RSS-Pfad (Phase 3.2 Layer-3-Integration, DONE 2026-05-11):** `save_analysis_direct()` dispatcht ueber `source_type` ("yt"|"rss"), auto-inferiert aus Anwesenheit von `video_id` vs. `source_url`. RSS-Aufruf braucht `source_url` + `source_title` + `source_feed_id` + `publish_date` (explizit; kein Auto-Fetch). Dedup-Unique-Key: `(investor_id, source_url)` fuer RSS, `(investor_id, video_id)` fuer YT — beide gemeinsam in `<Domain>/Analyses/analysis_log.json` mit `source_type`-Marker (Legacy-YT-Eintraege ohne `source_type` werden implizit als "yt" behandelt). Backwards-kompatibel: bestehende positional YT-Aufrufer unveraendert; neue Parameter (`skip_reason`, `source_*`) sind keyword-only nach `*`-Separator. Analysis-Datei-Naming RSS: `<publish_date>_rss_<feed_id>_<slug>.md`.

**Pending-Workflow `--pending-rss`:** zeigt offene (Item, Persona)-Paare aus allen `RSS/reports/scan_*.md` Reports (gruppiert pro Investor; pro (Item, Persona) eine eigene Pending-Zeile per F1). Pro Investor-Sektion ein **Recent-5-Analysen-Hint** (YT+RSS gemischt, sortiert nach publish_date DESC) zur Echo-Detection vor dem Layer-3-Step (F6).

**Skip-Disziplin (RSS-Layer-3 ist primaer R/E/P-Trigger-Layer; K/D/N nur bei explizitem Persona-Statement im Excerpt):** Vier Skip-Klassen via `save_analysis_direct(findings=[], skip_reason=...)`:
- `customer_pr` — Kundenstory/Drittanbieter-PR ohne Persona-Statement (Parloa, Uber, Singular Bank).
- `consumer_pr` — Consumer/Vertical/CSR-PR ohne AI-Strategy-Bezug (Gaming-SSO, EMEA-Youth-Safety).
- `eng_blog` — Engineering-Implementations-Blog ohne strategischen Pivot (Voice-Latency, Codex-Safety).
- `needs_fulltext` — Excerpt-Substanz fehlt, aber Tier-A-Item; Volltext-Bedarf fuer Folge-Session vermerkt.

Plus `echo:<yt_id_or_rss_url>` fuer Cross-Reference-Skip wenn RSS-Item denselben Inhalt wie ein vorhandener Layer-3-Eintrag wiederkaut (F6). Drittpartei-Lesart und Komplement-Faelle werden analysiert, mit Cross-Ref-Praefix `[Cross-Ref <ref>]` im Finding-Text. `--fetch-full` bleibt DEFER per F2 — kein Volltext-Workflow in Phase 3.2.

**Macro-YT-Skip-Klassen (Layer-3-Skip-Disziplin parallel zur AI-RSS-Disziplin):** Bisher kodifiziert ueber `feedback_skip_clip_transcripts` (Echo-Clips als wortwoertliche Auszuege via `save_analysis_direct(findings=[])`). 2026-05-16 Pal-Layer-3-Pass-3 etabliert zusaetzlich:
- `affiliate_promo` — Drittpartei-Affiliate-Channel bewirbt Investor-Material ohne Investor-Eigenstatement (Beleg: FkUrhcexbjE DeCRYPTion Ep.009 — Drittchannel "James" bewirbt das GMI-Bibel-Report-PDF von Pal/Bittel auf Real Vision; Pal kommt selbst nicht zu Wort). Distinkt zu `echo:<id>` (kein woertlicher Auszug einer Vollversion, sondern Drittpartei-Werbung). Auch fuer AI-YT verwendbar.

**Layer 4 — Council** (`<Domain>/Council/`, generated `dashboard.html`)
Interactive Q&A against all profiles simultaneously. 3 modes: council / consensus / devils_advocate.

**Dashboard-HTML-Generator** (`scripts/generate_dashboard.py --domain {macro|ai}`): produziert die self-contained `<Domain>/dashboard.html`. Domain-aware:
- AI-Sidebar gruppiert thematisch (Lab-CEOs / Hyperscaler / Forscher / Analysten / China; investor→group via `AI_THEME_GROUPS` in script).
- AI-Healthbar/Card-Badges/Analysen-Findings rendern 7-Segment K/D/N/R/V/P/E (Macro bleibt 3-Segment K/D/N).
- Type-Normalisierung: retrofit.json haelt Macro=long ("kontinuitaet"…) und AI=short ("K"…); `normalize_event_type()` akzeptiert beide. analysis_log-Findings koennen Array-Form (AI) oder Dict-Form (Macro) sein; build_data konvertiert beides in Dict-Form fuer JS.

**Layer 5 — Thesenverlauf** (`<Domain>/Thesenverlauf/`)

**Build-Tooling (durable, committet 2026-06-01 — Dashboard-Commit 2da2f60):** `scripts/build_thesenverlauf/build_retrofit.py` + `build_html.py` (+ README) ersetzen die frueheren ephemeren `tmp_build_<persona>_{retrofit,html}.py`-Skripte (von Session-Cleanups wiederholt geloescht — Root-Cause-Fix). `build_retrofit.py` macht **additiven Merge** neuer `analysis_log`-Events in `retrofit.json` (retrofit = verified Single Source of Truth; **kein** Full-Rebuild → WELLE2-Injektionen / `reclassifications[]` / hand-tuned watch-notes bleiben erhalten; Dedup `source_url`/`video_id` via `identity_keys()`, Skip leere findings, long↔short-Typ-Norm, idempotent). `build_html.py` ersetzt nur den `<script id="data">`-Block (kernthesen numerisch-sortiert; watch_markers+videos aus retrofit; viewBox/H preserved weil hand-tuned). **Pipeline nach Layer-3-Pass:** `build_retrofit.py --all --dry-run` (git-diff-Gate) → `--all` → `build_html.py --all` → `generate_dashboard.py --domain ai`. Generisch fuer alle 11 Personas (`--investor <id>`); `--all` = die 5 RSS-aktiven (Hassabis/Altman/Amodei/Patel/Huang), Nadella separat via `--investor satya_nadella`. **Seit 2026-06-18 auch `--domain macro`** (Default `ai`; Macro behaelt Langform-Typen `type === 'kontinuitaet'`, `kernthesen.json` optional fuer Pal/Hougan, `notes`-as-string-Norm; `--all` = die 6 Macro-Investoren). kernthesen-Schema-Fallback (`build_html.lane_fields()`) ist Dauerloesung — die 6 old-schema kernthesen.json (Musk/Zuckerberg/LeCun/Liang/Sutskever/Nadella) werden bewusst NICHT normalisiert (Fallback liefert byte-identische Lane-Labels). Details: `scripts/build_thesenverlauf/README.md`. **NICHT loeschen.** Pass-Historie (Korea-Cascade, S-1/IPO-Prep, RSS-Voll-Scan/Catch-up …) → `dashboard_build_log.md`.

Per-investor SVG timeline showing continuity/discontinuity of Kernthesen over time. Swim-lane chart: one lane per thesis, markers per video date. Macro uses 3-marker schema (K = green circle, D = orange diamond, N = blue triangle). AI uses extended 7-marker schema (K/D/N + R = teal pentagon, V = red inverted-triangle, P = yellow star, E = purple square) per AI-v3 finding schema. Health-bar in lane label shows distribution; AI 7-segment, Macro 3-segment. Optional `watch_markers[]` block: badges at right lane edge mark forward-falsification anchors with `kind` (`p_falsification_watch`, `p_forecast_watch`, `r_realisation_watch`, `v_falsification_watch`, `e_external_pressure_watch`, `frame_diff_watch`, `standardisation_watch`, plus Sub-Kinds), `falsifikation_window`, `note`. Cross-Investor `frame_diff_watch` sind strikt bidirektional zu schliessen (`feedback_cross_ref_symmetry`). Data source: `Analyses/<investor>/retrofit.json` (kann `reclassifications[]`-Block fuer Audit enthalten; analysis_log.json untouched). **Charts gebaut: alle 6 Macro + alle 11 AI** (Watch-Persistence 11/11 Single-Source-of-Truth). Per-Persona-Bauprotokolle (Event-Counts, Watch-Decks, Pattern-Beitraege) → `dashboard_build_log.md`.

### Build-Historie (Record) → ausgelagert

Die vollstaendige Pass- und Build-Historie (Watch-Migration-Sweep, Persona-Pass-3-Layer-5-Watch-Builds, Block-8a..N3a, RSS-Layer-3-Pässe, Backref-Symmetrie-Audits etc.) liegt in `dashboard_build_log.md` (Repo-Root) — Record, nicht Steuerung. CLAUDE.md fuehrt nur durable Wissen + Konventionen.

**Konvention (Record vs. Wissen):** Neue Pass-/Build-Protokolle (`… DONE`-Eintraege, Per-Pass-Bilanzen, Event-Counts) ab jetzt an `dashboard_build_log.md` anhaengen, **NICHT** in CLAUDE.md. In CLAUDE.md kommen nur durable Aenderungen an Schema/Pipeline/Konventionen. Forward-State (offene Tasks) gehoert nach Linear (Job 2), nicht in CLAUDE.md.

**AI Profile-Build Conventions (Phase 2):**
- Each profile folder gets a `sources.md` annotated bibliography (URL + date + which Kernthese-ID it feeds + fetch-method). Wikipedia avoided (politicized). Podcast whitelist: Lex Fridman, Dwarkesh Patel, Hard Fork (NYT), Ezra Klein, Bg2 Pod, All-In, Bloomberg Open Interest — one explicit search-wave per podcast format per profile build. See `feedback_ai_sources_strategy` memory.
- Layer-3 finding schema for AI is v3 (K/D/N + R/V/P/E), see Layer-3 section above.

**Geplante Module / Additions → Linear:** Forward-State (AI/Feeds-Befüllung, `relevance_triage.py`, Additional Modules wie Market data / Earnings calendar / Fed/ECB watch / News aggregator) lebt als Backlog-Issues im Linear-Projekt „Dashboard" (Team Pan-Opticon), nicht mehr in CLAUDE.md — siehe Record-vs-Wissen-Konvention oben.

### Hermes Agent (Docs in diesem Repo: `Hermes/`)
Persönlicher Agent (NousResearch); Voll-Tracking im `project_hermes_agent`-Memory. Repo-Ordner `Hermes/` hält Setup-Doku, Cron-Prompts, Patches — kein Code.

**Browser-Zugang auf authentifizierte/private Seiten (z. B. Skool, 2026-06-27):** Hermes' Standard-Browser (`config.yaml → browser.cloud_provider: local`) ist ein isolierter Headless-Chromium mit Wegwerf-Profil — teilt **keine** Cookies mit dem normalen Chrome (landet auf Login-Seiten). Für Login-pflichtige Seiten: attended **`/browser connect`** in der Desktop-GUI (geht bei **lokalem** Gateway — verifiziert in `apps/desktop/src/app/session/hooks/use-prompt-actions.ts:1183`, Gating auf `mode === 'remote'`; NICHT in WebUI/Telegram/Discord-Chats). Das startet ein **sichtbares** Debug-Chrome (Port 9222, eigenes Profil `…\hermes\chrome-debug`), in das **der User sich selbst** einloggt — nie Passwort/Cookie/Token an den Agenten. Persistente `browser.cdp_url` in der config wird **abgeraten** (gilt für alle Oberflächen inkl. Crons + hebelt `/browser disconnect` aus, da der Override als Fallback weiter die config liest). Schutzkreis = Browser-**Profil**, nicht Seite → Debug-Profil minimal halten. Detail-Anleitung + kopierfertiger Hermes-Prompt: `Hermes/skool_browser_zugang.md`.
