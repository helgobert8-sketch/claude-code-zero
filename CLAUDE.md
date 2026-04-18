# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projects

### Tic Tac Toe (tictactoe.html)
Single-file, vanilla HTML/CSS/JavaScript web app — no build step, no dependencies.
**To run:** open `tictactoe.html` directly in a browser.

All game logic lives in `tictactoe.html`:
- **UI** — 3x3 CSS Grid board; cells get `.x`/`.o`/`.taken`/`.win` classes for styling.
- **Game state** — `cells` (9-element array), `current`, `gameOver`, `mode` (`'2p'` or `'1p'`), `score`.
- **AI** — minimax for Hard; random for Easy; mix for Medium. CPU always plays `'O'`.
- **Flow** — `handleClick` -> `placeMove` -> win/draw check -> if CPU's turn, `setTimeout(cpuMove, 400)`.

### Macro Investor Dashboard (C:\Users\manyw\Dashboard\)
Modular intelligence system for tracking 6 macro investors. Three-layer architecture:

**Layer 1 — Profiles** (`Dashboard/Profiles/`)
Structured investor profiles with Kernthesen, Frameworks, Current Positioning, Historical Calls.
- 6 investors: Raoul Pal, Jordi Visser, Tom Lee, Arthur Hayes, Michael Howell, Matt Hougan
- Config: `Profiles/investors.json`
- Each investor folder may contain `kernthesen.json` — canonical list of stable thesis IDs (e.g. `JV-01`..`JV-08` for Jordi) used for thesis tracking in Layer 3 + 5. Currently defined for: Jordi Visser.

**Layer 2 — YouTube Monitor** (`Dashboard/YouTube/`)
Python script scanning YouTube channels + search for new investor content.
- `youtube_monitor.py` — main script (scrapetube for discovery, yt-dlp for transcripts)
- `youtube_config.json` — investor channels, search terms, settings
- `cookies.txt` — YouTube auth cookies for transcript download (in .gitignore)
- `scan_history.json` — tracks seen videos across runs
- `reports/` — markdown scan reports
- `transcripts/` — downloaded video transcripts by investor
- Dependencies: `scrapetube`, `yt-dlp`, `feedparser` (Python 3.14, `py` launcher on Windows)
- Deno runtime at `AppData/Local/Microsoft/WinGet/Packages/DenoLand.Deno_.../deno.exe`

**Layer 3 — Profile Analysis** (`Dashboard/YouTube/profile_analyzer.py` + `Dashboard/Analyses/`)
Compares new video content against investor profiles. Classifies findings as:
- **Kontinuitaet** — reaffirms existing thesis
- **Diskontinuitaet** — revises or contradicts a position
- **Neues** — new aspects not previously in the profile

Analysis workflow: `--pending` -> read profile + transcript in session -> produce analysis -> `save_analysis_direct()` to log.

Analysis schema v2: each finding carries `type` + `thesis_id` (from `kernthesen.json`) + `finding` text. `save_analysis_direct(findings=[...], publish_date=...)` auto-fetches `publish_date` from YouTube if not passed. Legacy `classifications={}` dict still accepted for entries without thesis_id mapping. `load_kernthesen(investor_id)` loads the canonical thesis list for validation.

**Layer 4 — Council of Macro Investors** (`Dashboard/Council/`, `dashboard.html`)
Interactive Q&A against all profiles simultaneously. 3 modes: council / consensus / devils_advocate.

**Layer 5 — Thesenverlauf** (`Dashboard/Thesenverlauf/`)
Per-investor SVG timeline showing continuity/discontinuity of Kernthesen over time. Swim-lane chart: one lane per thesis, markers per video date (K = green circle, D = orange diamond, N = blue triangle), hover tooltips with finding details. Health-bar in lane label shows K/D/N ratio. Data source: `Analyses/<investor>/retrofit.json` (historic, one-shot mapping of pre-schema-v2 findings) + future new-schema entries. Currently built for: Jordi Visser (`jordi_visser.html`).

**Additional Modules** (planned): Market data, earnings calendar, Fed/ECB watch, news aggregator.
