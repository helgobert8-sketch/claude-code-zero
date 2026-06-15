# Hermes — Schnellstart-Checkliste (Aufsetz-Tag)

> Begleitblatt zu `HERMES_SETUP.md`. Prinzip: **erst eine saubere Basis, dann Schicht für Schicht** — nicht alles auf einmal. Nach jeder Phase ein **GATE**: nicht weiter, bevor es läuft. Haken setzen beim Erledigen.

## Vorab-Entscheidungen (vor dem Start klären)
- [ ] **Provider:** Anthropic/Claude **Opus 4.8** (Empfehlung — du hast Claude; Opus ist Alex' Wahl für die Hermes-Harness). Login/API-Key griffbereit?
- [ ] **Vault-Pfad** festlegen, z. B. `C:\Users\manyw\HermesBrain`
- [ ] **Messenger:** Telegram (Bot über @BotFather) — kommt in Phase 3

## Phase 1 — Basis (MUSS zuerst sauber laufen)
- [ ] Desktop-Installer von `hermes-agent.nousresearch.com/desktop` ausführen
- [ ] `hermes doctor` → alles grün
- [ ] `hermes model` → Anthropic / **Opus 4.8** wählen
- [ ] Erster Test-Chat mit etwas Überprüfbarem (z. B. „Schau in dieses Verzeichnis…") → Antwort + Tool-Nutzung ok
- [ ] `hermes -c` (Session fortsetzen) funktioniert
- [ ] **GATE:** Läuft ein sauberer Chat über mehrere Runden? → erst dann weiter.

## Phase 2 — Gedächtnis (Obsidian + 4-Schichten-System)
- [ ] **Obsidian** installieren + Vault-Ordner anlegen
- [ ] `hermes config set OBSIDIAN_VAULT_PATH "C:\Users\manyw\HermesBrain"`
- [ ] **Memory-Setup-Prompt aus Anhang A.1** an Hermes geben → er baut `Agent-Shared/` + `Agent-Hermes/` + `daily/`
- [ ] (optional) Wissensbasis-Prompt **A.2** mit deinen Domänen (Theologie/Philosophie/Makro/AI/Buttcoin)
- [ ] Agent-Ordner in ein **privates GitHub-Repo** sichern (Backup)
- [ ] **GATE:** Neue Session → liest Hermes den Vault beim Start? Schreibt er ins Tageslog?

## Phase 3 — Erreichbarkeit (von überall)
- [ ] `hermes gateway setup` → Telegram-Bot-Token eintragen
- [ ] `hermes gateway run` (Test) → vom Handy eine Nachricht schicken, Antwort kommt
- [ ] (später) `hermes gateway install` für Dauerbetrieb

## Phase 4 — Werkzeuge (sparsam dosieren!)
- [ ] **Linear:** `hermes mcp install linear` (OAuth im Browser) → nur nötige Tools aktivieren (`find/get/create/update_issue`)
- [ ] **Skills ausmisten:** in „Skills & Tools" alles Unbenutzte **abschalten** (jeder aktive Skill kostet Tokens; Hermes legt auch im Hintergrund welche an)
- [ ] (Content, optional) **Grok-OAuth** via `hermes tools` für Bild/Video + Trend-Scan

## Phase 5 — Autonomie
- [ ] **1 Morning-Brief-Cron** (per **Reverse-Prompting** erstellen, 9:00) → in der **Cron-UI** prüfen, dass er existiert
- [ ] (optional) 1 Aufgaben-Cron: Opportunity-Scan (Reddit/X) oder Quellen-Monitoring

## Goldene Regeln (immer)
- [ ] Erst **eine** saubere Basis, dann Schichten.
- [ ] Unklar, wie? → **Reverse-Prompting**: „Schreib mir den besten Prompt dafür."
- [ ] Skills regelmäßig **ausmisten** (Tempo + Kosten).
- [ ] **YOLO-Mode AUS**; bei Riskantem `terminal.backend docker`.
- [ ] Agent „kaputt"? → Claude Code im Hermes-Ordner öffnen, Problem beschreiben, fixen lassen.

---
*Volle Erklärungen, Befehle & Setup-Prompts: siehe `HERMES_SETUP.md`. Wir gehen das live gemeinsam durch.*
