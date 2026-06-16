# Hermes-Setup Doku — 15.06.2026

Protokoll des tatsächlichen Aufsetz-Verlaufs (Windows 11, Start bei null), inkl. Stolpersteine und Lösungen. Begleitend zur idealtypischen Anleitung `HERMES_SETUP.md` + Quickstart-Checkliste im selben Ordner.

---

## 0. Vorentscheidung: Provider GPT-5.5 / Codex (statt Opus 4.8)

Ursprünglich war Opus 4.8 als Harness-Modell geplant. Verworfen für den Start, weil:
- „Opus via Claude Max + Agent-SDK-Credit (ab 15.06.2026)" ist eine **Deckelung** (Max 5x = $100/Mo, separater Topf), kein „unbegrenzt" — danach Pay-as-you-go.
- Die **Anthropic-OAuth-Anbindung in Hermes ist die fehleranfälligste Ecke** (offene GitHub-Issues #12905 / #21107 / #25267; Detail siehe Auth-Warnblock in `HERMES_SETUP.md` Abschnitt 4).

Entscheid: heute mit **GPT-5.5 über ChatGPT/Codex-OAuth** aufsetzen (nutzt das vorhandene ChatGPT-Plus-Abo, kein API-Key). **Opus 4.8 später als zweites Orchestrator-Profil** via SDK-Credit — erst nach Claim + Verifikation, dass die Nutzung wirklich den Credit zieht.

**Env-Foot-Gun vorab geprüft:** weder `OPENAI_API_KEY` noch `ANTHROPIC_API_KEY` gesetzt (würde sonst den OAuth-Pfad überschreiben → ungewolltes Pay-as-you-go).

---

## 1. Installation (mit Stolpersteinen)

1. **Desktop-Installer** von `https://hermes-agent.nousresearch.com/` geladen und gestartet.
2. **Norton blockierte `install-main.ps1`** (Meldung „IDP.Generic"). → Das ist ein **False Positive** der Verhaltens-Heuristik auf Installer-PowerShell-Skripte. **„Ausnahme erstellen"** gewählt (nicht Quarantäne).
   - Installer lief danach nicht von selbst weiter → **abgebrochen + neu gestartet** → lief mit gesetzter Ausnahme durch.
3. **Desktop-GUI-Build schlug fehl:** „apps/desktop build failed (exit 1)".
   - Ursache (aus dem Log): Vite/Rolldown kann `./react-shim` aus dem npm-Paket `@assistant-ui/tap` nicht auflösen — **Versions-Drift im `main`-Branch**, kein System-/Norton-/Netzwerkfehler. (`NODE_ENV` war leer → anderer bekannter Build-Brecher ausgeschlossen.)
   - Der eigentliche **Agent-Core war bereits installiert** — nur die optionale Electron-GUI scheiterte.
4. **Pivot auf den CLI-Installer** (überspringt den Desktop-Build):
   - In einem normalen PowerShell-Fenster: `iex (irm https://hermes-agent.nousresearch.com/install.ps1)`
   - (Hinweis: beim ersten Versuch waren versehentlich `---` aus einem Copy-Block mitkopiert → entfernt; der Befehl muss mit `iex` beginnen.)
5. Der CLI-Installer **aktualisierte das Repo** (git fast-forward, 4 Commits) und zog dabei **den Fix für `@assistant-ui/tap` mit** (`tests/test_assistant_ui_tap_compat.py` + `package.json`/`-lock`).
   - Auf die Frage **„Restore local changes now? [Y/n]"** → **`n`** geantwortet (die gestashten „local changes" waren nur stale Build-Artefakte des fehlgeschlagenen GUI-Builds; nicht zurückspielen).

---

## 2. Setup-Wizard — getroffene Auswahl

- **„How would you like to set up Hermes?"** → **2. Full setup** (nicht 1 „Quick Setup / Nous Portal" — das wäre ein anderer Provider gewesen).
- **Provider** → **6. OpenAI ▸ (Codex CLI or direct OpenAI API)** → Unterauswahl **Codex CLI** (= OAuth übers ChatGPT-Abo, kein API-Key).
- **Codex-OAuth-Hürde:** roter Hinweis „Gerätecode-Autorisierung für Codex aktivieren".
  - Fix: **ChatGPT → Settings → Security → „Autorisierung per Gerätecode für Codex aktivieren" einschalten** (Default ist aus, da Device-Code-Flows phishing-anfälliger sind). Persönliches Plus-Konto = selbst aktivierbar, kein Admin nötig.
  - Danach Login wiederholt → erfolgreich. **„Default model set to: gpt-5.5 (via OpenAI Codex)".**
- **Terminal Backend** → **Local** (Default „Keep current"). Sicherheit via YOLO-Mode AUS, nicht via Sandbox; bei riskanten Jobs später optional `terminal.backend docker`.
- **Messaging-Plattformen** → **übersprungen** (Telegram bewusst erst in Phase 3 via @BotFather).
- **Tools (Token-Hygiene — nicht „alles an"):**
  - **AUS:** Computer Use (macOS — irrelevant auf Windows), Yuanbao, Spotify, Home Assistant, X (Twitter) Search (Grok-Auth fehlt → Phase 4). Video/Mixture-of-Agents/Context-Engine ebenfalls aus.
  - **AN (Kern):** Memory (kritisch!), Web Search, Browser Automation, Terminal, File Operations, Code Execution, Vision, Image Generation, Text-to-Speech, Skills, Task Planning, Session Search, Clarifying Questions, Task Delegation, Cron Jobs, Cross-Platform Messaging.
- **Tool-Provider (jeweils gratis/kein-Extra-Key):**
  - Browser Automation → **Local Browser** (Headless Chromium, schon installiert).
  - Image Generation → **OpenAI (Codex auth) — `gpt-image-2-medium`** (gratis übers ChatGPT-Abo, kein Key).
  - Text-to-Speech → **Microsoft Edge TTS** (gratis, kein Key). *Caveat:* `ffmpeg` ließ sich per winget nicht installieren → Sprach-Nachrichten-Encoding limitiert (später ffmpeg nachinstallieren).
  - Web Search → **DuckDuckGo (ddgs)** (gratis, kein Key); Seiten-Extraktion über den Local Browser.
  - Vision → automatisch über GPT-5.5 (vision-fähig), kein eigener Provider nötig.

---

## 3. Abschluss & Phase-1-GATE

- **„[OK] Installation Complete!"** + Hinweis **„Restart your terminal for PATH changes to take effect"**.
- **Neues** Terminal → `hermes` → Test-Chat: „Welches Modell nutzt du? + liste Ordner C:\Users\manyw\AppData\Local\hermes".
- Ergebnis: **Modell `gpt-5.5`, Provider `openai-codex`** + korrekte Tool-Nutzung (Ordner sauber gelistet). **Phase-1-GATE bestanden.**

---

## 4. End-Konfiguration (Stand 15.06.2026)

- **Modell:** `gpt-5.5` via `openai-codex` (ChatGPT-Abo-OAuth, kein API-Key); Kontextfenster 272K.
- **Terminal-Backend:** Local.
- **Image Gen:** `gpt-image-2-medium` (Codex-auth, gratis).
- **TTS:** Microsoft Edge TTS (gratis; ffmpeg fehlt).
- **Web Search:** DuckDuckGo (ddgs); Extraktion via Local Browser.
- **Vision:** über GPT-5.5.

**Wichtige Datei-/Ordner-Pfade:**
- Config: `C:\Users\manyw\AppData\Local\hermes\config.yaml`
- API Keys/.env: `…\hermes\.env`  ·  OAuth-Credentials: `…\hermes\auth.json`
- Persona: `…\hermes\SOUL.md`  ·  Memory: `…\hermes\memories\`
- Daten: `…\hermes\cron\, sessions\, logs\, skills\`
- Code (Repo): `…\hermes\hermes-agent\`
- Anleitung/Doku: `ClaudeCodeZero\Hermes\` (HERMES_SETUP, Quickstart, diese Doku)

---

## 5. Lessons / Gotchas (für künftige Installs)

- **Norton „IDP.Generic" auf Installer-PS-Skripte = False Positive** → Ausnahme erstellen (nicht Quarantäne); bei Hängern Installer abbrechen + neu starten.
- **Die Electron-Desktop-GUI ist die fragile Ecke** (main-Branch-Dependency-Drift). Der **CLI-Installer `install.ps1` überspringt den GUI-Build** und ist der robustere Weg. Die GUI ist für Hermes' Funktion nicht nötig (CLI/TUI + Gateway sind der Workhorse).
- **Codex CLI braucht „Gerätecode-Autorisierung"** in ChatGPT → Settings → Security (Default aus).
- **Kein `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` in der Umgebung** lassen — überschreibt den OAuth-Pfad → ungewolltes Pay-as-you-go.
- **Token-Hygiene:** Tools/Skills ausmisten; und der **Brain-Dump gehört in den Vault (on-demand)**, nicht in SOUL.md/AGENTS.md (die werden bei *jeder* Nachricht geladen).
- **Opus-4.8-Orchestrator-Profil** kommt später via Agent-SDK-Credit — erst nach Claim + Verifikation (Anthropic-OAuth = raue Ecke, siehe Warnblock in HERMES_SETUP.md).
- **Hermes patcht auf „Kannst du das fixen?" seinen EIGENEN Quellcode** (06-16-Erfahrung mit dem Auto-TTS-Bug): methodisch (Code + Regressionstests + `pytest`/`py_compile`), aber es entsteht **lokale Divergenz vom Nous-Upstream** im `…\hermes-agent\`-Repo. **Gut zu wissen:** (a) Patches werden erst beim nächsten Gateway-Restart scharf — vorher mit `git -C …\hermes-agent diff` reviewen; (b) Hermes kann den „Refusing to restart"-Schutz via *verzögertem* Restart umgehen und sich selbst neu starten; (c) jeden Selbst-Fix als `.patch` sichern (`git diff --output=…`) + in `ClaudeCodeZero\Hermes\` ablegen, sonst geht er bei `hermes update` („Restore local changes? n") verloren. Verwerfen: `git -C …\hermes-agent checkout -- .`.
- **Desktop-GUI bleibt die fragile Ecke** (2. Anlauf 06-16): jetzt nicht mehr `@assistant-ui/tap`, sondern **Electron-Download `fetch failed`** (AV/EDR-SSL-Interception) **+ node-22-vs-24-Drift**. Die App ist für Hermes' Funktion nicht nötig (CLI + Telegram + Gateway sind der Workhorse) — sauberer Bau-Versuch erst nach `hermes update` auf node-24-Basis.

---

## 6. Status & offene Phasen

- **Phase 1 — Basis (Install, Modell, Test-Chat):** ✅ DONE.
- **Phase 2 — Gedächtnis: DONE (Kern 06-15/16; Agent-Shared + Auto-Load-GATE 06-16).** Obsidian + Vault `C:\Users\manyw\HermesBrain` (`OBSIDIAN_VAULT_PATH` gesetzt; Vault war anfangs doppelt verschachtelt `HermesBrain\HermesBrain` → eine Ebene hochgezogen). 4-Schichten-Struktur gebaut (`INDEX.md` + `Agent-Shared/` + `Agent-Hermes/` + `daily/`; leere Ordner via `.gitkeep`). Brain-Dump (physisch vorbereitet, in Hermes eingetippt) + Reverse-Prompting durch. **Geschrieben:** `SOUL.md` (`…\hermes\SOUL.md`, Persona/Ton) + `AGENTS.md` (`C:\Users\manyw\AGENTS.md` — Workdir-geladen, operatives Destillat **ohne Sensibles**, Projekte nur Name+1-Zeile). Persistenz-Sicherung: `Agent-Shared/_braindump-raw.md` + `_entscheidungen.md`. **Routing-Regel:** SOUL=Persona · AGENTS=operatives Destillat (nichts Sensibles, geht bei jeder Nachricht an OpenAI) · Agent-Shared=voller Wissensschatz on-demand (sensibel ok, **keine** Seed-Phrases/Passwörter/Kontonummern). **Agent-Shared gefüllt (06-16, durch Claude Code, Freigabe-Schleife pro Entwurf eingehalten):** 10 thematische Notizen aus `_braindump-raw.md` (`01-person` … `10-referenzen`) + in `INDEX.md` verlinkt. Redaktionsregeln: Code/Build **präferentiell** (nicht exklusiv) bei Claude Code — Prinzip „passendes Modell je Aufgabe" (Hermes/GPT-5.5+Codex kann abgegrenztes Coden selbst; dynamisches Modell-Feld; API-Kostenfall); keine echten Geheimnisse; Buttcoin-Detail dünn → von Chris zu ergänzen. **Quick-Check bestanden:** frisch `hermes` aus Home-Terminal → „Was ist mein Nordstern?" korrekt aus AGENTS.md (Auto-Load bestätigt).
- **Phase 3 — Erreichbarkeit: ✅ DONE (06-16).** Telegram-Bot `@paronthes_hermes_bot` via @BotFather (`/newbot` → Name → Username auf `bot` endend → Token). `hermes gateway setup` → Telegram + Token + **Allowed-User = eigene numerische Telegram-ID** (geholt via **@userinfobot**, NICHT über den eigenen Bot!) + **Home-Channel = eigene DM**. `hermes gateway install` registriert **Scheduled Task `Hermes_Gateway`** (Auto-Start bei Windows-Login; UAC nur für die Task-Anlage). **Text-GATE** (hin/zurück) bestanden. **ffmpeg** für Voice via winget mit **expliziter Paket-ID** `Gyan.FFmpeg.Essentials -e --accept-source-agreements --accept-package-agreements` (das generische `winget install ffmpeg` scheiterte zuvor an Mehrdeutigkeit) → ffmpeg 8.1.1 full-build/gyan.dev mit `libopus`; danach `hermes gateway restart`. **Voice-GATE** beide Richtungen bestanden (Opus→Text Transkription + TTS→Opus Encoding). **Deutsche Stimme:** Default war englisch (`en-US-AriaNeural`) → in `C:\Users\manyw\AppData\Local\hermes\config.yaml` (NICHT `~/.hermes/`!) unter `tts.edge.voice` auf `de-DE-FlorianMultilingualNeural` (mehrsprachig, m) geändert + Gateway-Restart.
- **Phase 4 — Werkzeuge: ✅ DONE (06-16).**
  - **(A) Grok als Tool:** `hermes auth add xai-oauth` (PKCE-Loopback-OAuth, setzt **SuperGrok / X-Premium+** voraus; **kein** Modellwechsel — GPT-5.5 bleibt Orchestrator, daher `auth add` statt `hermes model`) → `hermes tools enable x_search`. **Live verifiziert:** X-Trend-Scan über Telegram liefert echte X-Status-Links. `video_gen` testweise aktiviert, mangels Provider + akutem Bedarf wieder `disable`d; Bild-Gen bleibt unverändert bei `openai-codex`/`gpt-image-2-medium`. (Hinweis: `hermes login` ist in v0.16.0 entfernt → `hermes auth add` nutzen.)
  - **(B) Desktop-App: ⏸️ GEPARKT** — `hermes desktop --build-only` scheitert an (1) Electron-Binary-Download (`@electron/get` → `fetch failed`, vermutlich AV/EDR-SSL-Interception wie bei RSS/YT) **und** (2) node-22-vs-24-Drift (Desktop-Code verlangt node≥24, Hermes bündelt v22). Für „wenn Platz"-Bonus nicht den Eingriff wert. **Wichtig:** Der Source-Build (`hermes desktop`) ist NICHT der einzige Weg — laut früherer Recherche gibt es einen **prebuilt `.exe`-Installer** (Docs `…/windows-native`), der den node/Electron-Source-Build umgeht = der noch-nicht-probierte, vielversprechendere Pfad. Alternativ vorher `hermes update` (node-24 + aktueller Code, `hermes backup` davor) + SSL-Bypass + Gateway-Stop gegen EBUSY-Locks. Als **optionaler Auftakt von Phase 5** vorgemerkt.
  - **(C) Skills-Ausmist:** 63 → **37 enabled / 26 disabled** via `skills.disabled`-Liste in config.yaml (Batch über Hermes' eigene `save_disabled_skills`-API; Backup `config.yaml.bak.ausmist_*`, 1 Zeile reversibel). Deaktiviert: Musik/Audio + Bild/CV/VJ + ML-Training/Ops + Fremd-Ökosysteme (yuanbao/openhue/himalaya/airtable/teams/polymarket/maps) + 4 Design (excalidraw/sketch/notion/powerpoint). Skills-Index im System-Prompt 7,1 → **4,0 KB** (−42 %); Tool-Schemas +4 KB durch `x_search` → netto sinnvoller belegt statt totes Rauschen.
  - **Nebenereignis 06-16 — Hermes hat seinen eigenen Auto-TTS-Bug selbst gefixt:** Auf „Kannst du das fixen?" hin **7 Dateien im `hermes-agent`-Source gepatcht** (`gateway/platforms/base.py`+`run.py`+`slash_commands.py` + 4 Test-Dateien, 125+/18−), `/voice on` (Voice→Voice) vs `/voice tts` (auch Text→Sprache) vs `/voice off` sauber getrennt, **`177 passed / 21 skipped`** + `py_compile`/`git diff --check` sauber, dann **Gateway selbst neu gestartet** (verzögerter Restart, umging den „Refusing to restart"-Schutz) → Auto-TTS-**Mechanik** greift live (Text-Antwort kam mit Sprachnachricht). **Caveat 2026-06-16:** die Test-Antwort war zufällig eine **englische Rate-Limit-Meldung** → Sprachnachricht englisch; **deutsche Stimme (de-DE-Florian) + echte inhaltliche Antwort sind noch zu verifizieren** (offen bis ChatGPT-Plus-Limit-Reset). **Patch gesichert** als `Hermes/hermes_autotts_selffix_20260616.patch`. ⚠️ **Upstream-Divergenz** — diese 7 Dateien weichen jetzt vom Nous-Code ab und erschweren künftige `hermes update`s (genau den Desktop-/node-24-Pfad). Vor einem Update: Patch via `git apply` re-anwenden ODER auf offiziellen Nous-Fix warten + lokal zurückrollen (`git -C …\hermes-agent checkout -- .`).
- **Phase 5 — Autonomie:** Morning-Brief-Cron via Reverse-Prompting + **Scout-Rolle** (personalisierte Ideen-/Prototyp-Vorschläge aus Personenkenntnis, Nordstern-gebunden + Triage; graduierte Vorschläge → Linear/PRD → Claude Code). **Optionaler Auftakt: Desktop-App** (Befund Phase 4 B) — nur koppeln, wenn die Profiles-/Cron-UI für Multi-Profil/Autonomie konkret gebraucht wird. **Mission Control** nicht jetzt — Kandidat fürs Buttcoin-Content-System („Content-System rund machen"-Track, startet mit Failure-Diagnose der Vorversionen).

---

## Anhang A — Verwendete Prompts (zum Wiederverwenden)

### A.1 Test-Chat (Phase-1-GATE)

```text
Welches Modell nutzt du gerade? Und liste mir bitte die Top-Level-Einträge im Ordner C:\Users\manyw\AppData\Local\hermes auf.
```

### A.2 Memory-Setup-Prompt (4-Schichten-Struktur im Vault anlegen)

```text
Richte unser Gedächtnis-System im Obsidian-Vault ein. Geh so vor:
1. Löse zuerst den Vault-Pfad auf (Umgebungsvariable OBSIDIAN_VAULT_PATH) und nenne mir den konkreten absoluten Pfad, den du verwendest.
2. Lege im Vault diese Struktur an (mit den Dateitools, nicht über die Shell):
   - Ordner "Agent-Shared/" — geteiltes, dauerhaftes Wissen über mich, meine Projekte, Ziele, Referenzen (on-demand abgerufen).
   - Ordner "Agent-Hermes/" — deine eigenen Notizen/Erkenntnisse als Hermes.
   - Ordner "daily/" — Tageslogs (eine Notiz pro Tag, Format YYYY-MM-DD.md).
   - Eine "INDEX.md" im Vault-Root, die die Struktur erklärt (mit [[Wikilinks]]).
3. Halte in der INDEX.md auch unsere Routing-Konvention fest:
   - SOUL.md (in ~/.hermes) = nur schlanke Persona/Ton.
   - AGENTS.md = destillierte Essenz (wer ich bin kurz, aktive Projekte, harte Regeln, Arbeitsweise) — kurz halten, wird bei jeder Nachricht geladen.
   - Agent-Shared/ = der volle, ausführliche Wissensschatz — on-demand.
4. Lies künftig zu Session-Beginn die INDEX.md + relevante Agent-Shared-Notizen und führe ein Tageslog in daily/.
5. Erfinde JETZT noch keine Inhalte über mich — den großen Brain-Dump gebe ich dir im nächsten Schritt; jetzt nur Struktur + Konvention.
Antworte auf Deutsch und zeig mir am Ende den angelegten Baum.
```

### A.3 Brain-Dump-Rahmen-Prompt (Brain-Dump + Reverse-Prompting, geroutet)

```text
Gleich gebe ich dir einen ausführlichen Brain-Dump über mich, meine Arbeit, meine Ziele und unsere Zusammenarbeit. Geh GENAU so vor:
1. Ich tippe den Brain-Dump evtl. über mehrere Nachrichten. Nimm alles nur auf — fass nichts zusammen und schreib nichts in Dateien — bis ich "FERTIG" schreibe.
2. Wenn ich "FERTIG" sage: stell mir deine Rückfragen (Reverse-Prompting) in sinnvollen Bündeln (nicht 20 auf einmal), um Lücken zu schließen und unsere Zusammenarbeit zu schärfen.
3. Erst wenn wir mit den Rückfragen durch sind, verteilst du das Wissen geroutet — und zeigst mir JEDEN Entwurf zur Freigabe, BEVOR du schreibst:
   - Persona/Ton → SOUL.md (schlank halten)
   - destillierte Essenz (wer ich bin in Kurzform, aktive Projekte, harte Regeln, Arbeitsweise) → AGENTS.md (bewusst kurz, max. 1–2 Seiten; wird bei jeder Nachricht geladen). Nenn mir den Pfad, an dem du AGENTS.md lädst.
   - der volle, ausführliche Brain-Dump → Agent-Shared/, thematisch in mehrere Notizen aufgeteilt, mit [[Wikilinks]], und in der INDEX.md verlinkt (on-demand).
Bestätige jetzt mit "bereit" — dann tippe ich los. Antworte auf Deutsch.
```

### A.4 Phase 3 — Erreichbarkeit (Telegram + Voice), wiederverwendbare Befehle

**Telegram-Bot anlegen (in der Telegram-App):**
1. `@BotFather` öffnen → `/newbot` → Anzeigename → Username (muss auf `bot` enden) → **Token** notieren (geheim).
2. **Eigene numerische User-ID holen:** `@userinfobot` anschreiben (`/start`) → er antwortet `Id: 123456789`. (Wichtig: das ist ein **eigener** Bot — der eigene Hermes-Bot kann die ID nicht liefern.)

**Gateway einrichten + Dauerbetrieb (PowerShell, Home-Verzeichnis):**
```powershell
hermes gateway setup     # Telegram wählen → Token → Allowed-User = eigene ID → Home-Channel = eigene DM (Y)
hermes gateway run       # Vordergrund-Test (Logs sichtbar); Ctrl+C stoppt
hermes gateway install   # als Scheduled Task 'Hermes_Gateway' (Auto-Start bei Login; UAC bestätigen)
hermes gateway status    # 'running/active' prüfen
hermes gateway restart   # nach Config-Änderungen (liest config.yaml neu ein)
```
*Nicht gleichzeitig `run` (Vordergrund) und den installierten Dienst laufen lassen → Telegram-Conflict (ein Token, ein getUpdates-Consumer).*

**ffmpeg für Voice-Messages (winget, kein Admin nötig):**
```powershell
winget install --id Gyan.FFmpeg.Essentials -e --accept-source-agreements --accept-package-agreements
# danach NEUES Fenster: ffmpeg -version  →  dann: hermes gateway restart
```
*Explizite `--id` nutzen — `winget install ffmpeg` bricht an Mehrdeutigkeit ab.*

**Deutsche TTS-Stimme setzen** (Config: `C:\Users\manyw\AppData\Local\hermes\config.yaml`, **nicht** `~/.hermes/`):
```yaml
tts:
  provider: edge
  edge:
    voice: de-DE-FlorianMultilingualNeural   # m, mehrsprachig (DE + eingebettetes EN); Alt.: de-DE-SeraphinaMultilingualNeural (w), de-DE-ConradNeural (m), de-DE-KatjaNeural (w)
```
*Edit speichern → `hermes gateway restart`. Backup der Config liegt als `config.yaml.bak.*` daneben.*

### A.5 Phase 4 — Werkzeuge (Grok-Tool + Skills-Ausmist), wiederverwendbare Befehle

> Nicht-interaktive `hermes`-Checks lassen sich auch ohne Terminal-Neustart fahren, wenn man den Launcher absolut adressiert: `C:\Users\manyw\AppData\Local\hermes\hermes-agent\venv\Scripts\hermes.exe`. Interaktives (OAuth-Browser-Flow) im eigenen Terminal.

**(A) Grok als Tool anbinden (kein API-Key, OAuth):**
```powershell
hermes auth add xai-oauth        # PKCE-Loopback-OAuth; Browser → autorisieren. SuperGrok/Premium+ nötig. Aendert NICHT das Hauptmodell.
hermes auth status xai-oauth     # erwartet: "logged in"
hermes tools enable x_search     # X-Trend-Scan scharf schalten (nutzt die xai-oauth-Credential, model grok-4.20-reasoning)
hermes tools list                # Verifikation
```
*Funktionstest (in Telegram/Hermes):* „Nutze dein X-Search-Tool (Grok): die 3–4 meistdiskutierten Themen rund um KI-Agenten, je mit Link." → muss `🐦 x_search` feuern + echte x.com-Status-Links liefern. (`video_gen` braucht zusätzlich einen Provider-Block → erst bei konkretem Bedarf.)

**(B) Tool-/Skill-Status messen + ausmisten:**
```powershell
hermes prompt-size               # Byte-Breakdown System-Prompt + Tool-Schemas (Baseline/Erfolgsmessung)
hermes tools list                # Toolsets enabled/disabled
hermes skills list               # installierte Skills + Status (Summenzeile: x enabled / y disabled)
hermes skills config             # interaktive TUI zum Einzel-Toggle
```
**Skills non-interaktiv per Batch deaktivieren** (TUI schreibt nur die Liste `skills.disabled` in config.yaml → direkt setzbar):
```python
# venv-Python, cwd = hermes-agent
from hermes_cli.config import load_config
from hermes_cli.skills_config import get_disabled_skills, save_disabled_skills
cfg = load_config()
save_disabled_skills(cfg, set(get_disabled_skills(cfg)) | {"comfyui","yuanbao",...}, platform=None)
```
*Danach `hermes gateway restart`, damit der schlankere Prompt in der laufenden Session greift (oder warten bis zum nächsten Login-Auto-Start). Config-Backup vorher anlegen.*

**Gateway-Restart-Disziplin:** `hermes gateway restart` killt den Prozess (kein graceful-Flag; `restart_drain_timeout: 180` lässt laufende Arbeit nur bis 180 s abfließen). **Nicht** mitten in eine laufende Agent-Aufgabe restarten — abwarten oder den nächsten Login-Auto-Start nutzen (Config-Änderungen sind schon persistent).

---

## Anhang B — Datenschutz / Daten-Fluss (wer sieht die Memory-Inhalte?)

Wichtig, weil die Memory teils persönliche Infos enthält:

- **Speicherung = lokal.** Vault, `SOUL.md`, `AGENTS.md` liegen auf dem Rechner; Hermes läuft lokal und schickt nichts an NousResearch.
- **„Sehen" = nur, was tatsächlich in eine Anfrage geht.** Ein Modell ist zustandslos und kennt nur, was im jeweiligen Request steht.
  - **Aktives Hauptmodell (derzeit GPT-5.5 / OpenAI):** bekommt **`SOUL.md` + `AGENTS.md` bei jeder Nachricht** + **`Agent-Shared`-Notizen nur, wenn on-demand abgerufen.** → OpenAI ist der primäre Empfänger des geladenen Kontexts.
  - **Tools (Grok, Web-Suche, Bild-Gen, TTS):** bekommen **nur ihren engen Input** (Query, Bild-Prompt, zu sprechender Text) — nicht den ganzen Brain-Dump.
  - **Zweites Modell-Profil (z. B. Claude/Opus später):** sieht die Memory nur, wenn es das gerade aktive Modell einer Session ist (dann Anthropic).
- **Daten-Minimierung (Praxis):**
  - `AGENTS.md` geht bei *jeder* Nachricht raus → schlank + nichts Hochsensibles.
  - `Agent-Shared/` geht nur on-demand raus → besserer Ort für ausführliche persönliche Details.
  - Echte Geheimnisse (Passwörter, Keys, Konto-/Finanzdaten) gehören in **keine** Memory-Datei.
- **Training-Hebel:** Bei OpenAI/ChatGPT steuern die Data Controls deines Kontos, ob Eingaben fürs Training genutzt werden (ChatGPT → Settings → Data Controls → Modell-Training aus, falls gewünscht).
