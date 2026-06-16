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

---

## 6. Status & offene Phasen

- **Phase 1 — Basis (Install, Modell, Test-Chat):** ✅ DONE.
- **Phase 2 — Gedächtnis: DONE (Kern 06-15/16; Agent-Shared + Auto-Load-GATE 06-16).** Obsidian + Vault `C:\Users\manyw\HermesBrain` (`OBSIDIAN_VAULT_PATH` gesetzt; Vault war anfangs doppelt verschachtelt `HermesBrain\HermesBrain` → eine Ebene hochgezogen). 4-Schichten-Struktur gebaut (`INDEX.md` + `Agent-Shared/` + `Agent-Hermes/` + `daily/`; leere Ordner via `.gitkeep`). Brain-Dump (physisch vorbereitet, in Hermes eingetippt) + Reverse-Prompting durch. **Geschrieben:** `SOUL.md` (`…\hermes\SOUL.md`, Persona/Ton) + `AGENTS.md` (`C:\Users\manyw\AGENTS.md` — Workdir-geladen, operatives Destillat **ohne Sensibles**, Projekte nur Name+1-Zeile). Persistenz-Sicherung: `Agent-Shared/_braindump-raw.md` + `_entscheidungen.md`. **Routing-Regel:** SOUL=Persona · AGENTS=operatives Destillat (nichts Sensibles, geht bei jeder Nachricht an OpenAI) · Agent-Shared=voller Wissensschatz on-demand (sensibel ok, **keine** Seed-Phrases/Passwörter/Kontonummern). **Agent-Shared gefüllt (06-16, durch Claude Code, Freigabe-Schleife pro Entwurf eingehalten):** 10 thematische Notizen aus `_braindump-raw.md` (`01-person` … `10-referenzen`) + in `INDEX.md` verlinkt. Redaktionsregeln: Code/Build **präferentiell** (nicht exklusiv) bei Claude Code — Prinzip „passendes Modell je Aufgabe" (Hermes/GPT-5.5+Codex kann abgegrenztes Coden selbst; dynamisches Modell-Feld; API-Kostenfall); keine echten Geheimnisse; Buttcoin-Detail dünn → von Chris zu ergänzen. **Quick-Check bestanden:** frisch `hermes` aus Home-Terminal → „Was ist mein Nordstern?" korrekt aus AGENTS.md (Auto-Load bestätigt).
- **Phase 3 — Erreichbarkeit:** Telegram via @BotFather. (ffmpeg für Voice-Messages nachinstallieren.)
- **Phase 4 — Werkzeuge:** Grok als Tool (X-Trends/Bild) + Skills/Tools-Ausmist-Pass. (**Linear raus aus Hermes** — gehört zur Claude-Code-Build-Achse, nicht zu Hermes; Entscheid 2026-06-16.)
- **Phase 5 — Autonomie:** Morning-Brief-Cron via Reverse-Prompting + **Scout-Rolle** (personalisierte Ideen-/Prototyp-Vorschläge aus Personenkenntnis, Nordstern-gebunden + Triage; graduierte Vorschläge → Linear/PRD → Claude Code). **Mission Control** nicht jetzt — Kandidat fürs Buttcoin-Content-System („Content-System rund machen"-Track, startet mit Failure-Diagnose der Vorversionen).

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

*(Weitere Prompts aus Phase 3–5 werden hier ergänzt, sobald wir sie nutzen.)*

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
