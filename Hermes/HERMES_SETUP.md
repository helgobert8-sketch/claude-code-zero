# Hermes Agent — Einrichtung & Nutzung (Schritt-für-Schritt)

> **Version 2 (Stand 2026-06-13).** v1 war das technische Rückgrat aus der offiziellen NousResearch-Doku. **v2 integriert Alex Finns konkrete Praxis** aus der Vibe Coding Academy (Bootcamp-Live-Sessions #14–#17, „Loop"-Session, plus seine Posts zu Memory-System, Linear und Fixes). Alex-spezifische Punkte sind mit **[Alex]** markiert.
>
> Zielsystem: **Windows 11, Start bei null.** Schwerpunkt laut Auftrag: Hermes als (d) Alltags-„AI-Employee", (c) Content/Social, (a) Second Brain (Obsidian + Linear). Programmieren bleibt bei Claude Code.
>
> **Was v2 gegenüber v1 ändert (Kurz-Changelog):**
> - **Provider:** für die *Hermes-Harness* → **Opus 4.8** (nicht Fable — zu teuer in Hermes). Abschnitt 4.
> - **Gedächtnis:** Alex' **offizielles 4-Schichten-Memory-System** als primäre Variante (Abschnitt 6 + Anhang A).
> - **Neu:** Profiles vs. Sub-Agents (Abschnitt 7). Hermes-Desktop-Praxis (Abschnitt 3.3).
> - **Linear:** „zweites Gehirn", nicht Orchestrator; Hermes-Kanban für Orchestrierung (Abschnitt 8). Verbatim-Templates → `templates/`.
> - **Skills-Disziplin:** unbenutzte Skills abschalten (Token-Bloat). Abschnitt 12.
> - **Content/Social:** Multi-Profile-Pipeline + Grok-OAuth + Opportunity-Loop (Abschnitt 10).
> - **Neu (Interview Isenberg×Finn, 6/2026):** Reverse-Prompting/Brain-Dump als Kerntechnik (14.0), Opportunity-Scan-Details (10.3), Kosten-Hygiene mit `/new` (4), härtere OpenClaw-Wechselgründe (0).

---

## 0. Was ist Hermes — in einem Absatz

**Hermes Agent** ist ein quelloffener KI-Agent von **NousResearch** (`github.com/NousResearch/hermes-agent`, Doku: `hermes-agent.nousresearch.com/docs`). Anders als ein reines Chatfenster ist Hermes ein **Agent mit Werkzeugen, Gedächtnis und Autonomie**: Er führt Terminalbefehle aus, liest/schreibt Dateien, sucht im Web, nutzt 100+ mitgelieferte „Skills", bindet externe Dienste per MCP an, lässt sich über Messenger (Telegram/Discord/Slack) bedienen und erledigt Aufgaben zeitgesteuert (Cron). Er läuft als **CLI**, **TUI**, native **Desktop-App** (Windows/macOS/Linux) und **Web-Dashboard** — alle teilen denselben Agenten, dieselbe Konfiguration, dieselben Sessions.

**Hermes vs. OpenClaw — Alex' Stand (Juni 2026):** Beides sind „AI-Employee"-Frameworks und **funktional fast identisch** („from a pure execution perspective, they're basically the same"). **[Alex]** lehnt inzwischen **zu Hermes**: das Team ist fokussierter, Releases seltener aber zuverlässiger („du kannst updaten und weißt, es bricht nichts"), bessere Produktvision, und die neue Desktop-App liefert die beste UX. Zwei harte Gründe aus dem Isenberg-Interview (6/2026): OpenClaws ungetestete „Schrotflinten"-Updates haben Alex' Setup tatsächlich zerschossen, und **OpenClaw verdrahtet Modelle fest**, während **Hermes Modelle/Thinking-Settings dynamisch wechselt**, sobald ein neues erscheint (sein Apple-vs-Android-Vergleich: Hermes = fokussierte, getestete Wochen-Updates). Aber **anti-tribalistisch**: läuft OpenClaw bei dir gut, gibt es keinen Zwang zu wechseln — du kannst beide parallel betreiben (getrennte Ordner/Agenten, sie stören sich nicht). **Für dich (Start bei null): direkt mit Hermes anfangen.**

---

## 1. Das mentale Modell (bevor du etwas installierst)

Stell dir Hermes als **Betriebssystem mit Gedächtnis und Mitarbeitern** vor, nicht als Chatbot:

```
                       ┌─────────────────────────────┐
                       │        HERMES (Kern)         │
                       │  „Kontrollturm" — denkt,     │
                       │  ruft Werkzeuge, koordiniert │
                       └──────────────┬──────────────┘
            ┌─────────────────────────┼──────────────────────────┐
            │                         │                          │
   ┌────────▼────────┐      ┌─────────▼─────────┐      ┌──────────▼─────────┐
   │   GEDÄCHTNIS     │      │    FÄHIGKEITEN     │      │    ZUGÄNGE / I/O    │
   │  4-Schichten +   │      │  Skills + MCP      │      │  Desktop-App · CLI  │
   │  Obsidian-Vault  │      │  (Obsidian, X,     │      │  Telegram · Discord │
   │  (Abschnitt 6)   │      │  Linear, GitHub …) │      │  · Web-Dashboard    │
   └──────────────────┘      └────────────────────┘      └─────────────────────┘
            │                         │
   ┌────────▼────────┐       ┌────────▼────────┐
   │   PROFILES       │       │   AUTONOMIE      │
   │ mehrere Spezial- │       │  Cron · Kanban   │
   │ Agenten (Absch.7)│       │  · Loops (A. 11) │
   └──────────────────┘       └──────────────────┘
```

- **Kern**: das LLM (für Hermes → Opus, Abschnitt 4) + die Hermes-Logik.
- **Gedächtnis**: 4 Schichten von „immer präsent" bis „on-demand"; Herzstück ist ein **Obsidian-Vault** (Abschnitt 6). Das macht Hermes über Wochen *besser*.
- **Fähigkeiten**: **Skills** (Anleitungs-Dokumente) + **MCP-Server** (externe Dienste, z. B. Linear).
- **Zugänge**: wie du redest — **Desktop-App** (empfohlen), Terminal, oder Messenger von unterwegs.
- **Profiles**: mehrere *getrennte* Spezial-Agenten (Abschnitt 7).
- **Autonomie**: Cron-Jobs, der Kanban-Board, „Loops" (Abschnitt 11).

**Goldene Regel:** Erst *einen* sauberen Chat zum Laufen bringen. Erst *dann* Schicht für Schicht ergänzen (Gedächtnis → Messenger → Skills/MCP → Cron). Nicht alles auf einmal.

---

## 2. Voraussetzungen (Windows, von null)

| Was | Nötig? | Anmerkung |
|---|---|---|
| **Git** | empfohlen | `git --version`. Der Desktop-Installer holt den Rest selbst. |
| Python / Node.js / ripgrep / ffmpeg | **NEIN** (automatisch) | Installer zieht alle Abhängigkeiten. Nicht manuell. |
| **Ein LLM-Zugang** | **JA** | Abo (Anthropic / Nous Portal / ChatGPT) oder API-Key. Abschnitt 4. |
| Mind. **64.000 Token Kontext** | JA (Modell) | Claude/GPT/Gemini erfüllen das locker. |
| **Obsidian** (kostenlos) | für Gedächtnis | Abschnitt 6. |
| **Linear**-Account | für Aufgaben | Kostenloser Plan reicht; Abschnitt 8. |

> **Hosting [Alex]:** Du brauchst keinen Server. Windows-PC reicht. Ein VPS / Mac-Studio / DGX-Spark lohnt erst, wenn der Agent **24/7** laufen oder **lokale Modelle** hosten soll (Abschnitt 4).

---

## 3. Schritt 1 — Installation auf Windows

### 3.1 Weg A — Desktop-App-Installer (empfohlen) **[Alex]**
Alex nennt Hermes Desktop „the best experience for Hermes on the desktop" und nutzt auf dem Desktop kein Telegram mehr.
1. Öffne **`https://hermes-agent.nousresearch.com/desktop`**.
2. Lade den **Windows-Installer** herunter und führe ihn aus.
3. Er richtet automatisch **Kommandozeile + Desktop-App** ein (inkl. Python, Node.js, ripgrep, ffmpeg, venv, globaler `hermes`-Befehl).
4. Beim ersten Start führt dich das **Onboarding** durch Provider/Modell (du kannst „Choose provider later" wählen → Abschnitt 4 nachholen).

### 3.2 Weg B — nur Kommandozeile (PowerShell)
```powershell
iex (irm https://hermes-agent.nousresearch.com/install.ps1)
```
Desktop-App jederzeit nachinstallieren/-starten:
```powershell
hermes desktop
```

> **Datenablage (wichtig):** Auf Windows **`%LOCALAPPDATA%\hermes`** (in der Doku oft generisch `~/.hermes`). Dort liegen `.env` (Geheimnisse/API-Keys), `config.yaml` (Einstellungen), `skills/`, `sessions/`.

### 3.3 Was die Desktop-App dir bietet (und wie Alex sie nutzt) **[Alex]**
- **Sessions je Thema** statt eines „Uni-Threads" — z. B. *Content · Local-AI · Recherche · Coding · Strategie*. Hält Kontext sauber → bessere Antworten + weniger Token-Kosten.
- **Artifacts-Panel**: sammelt automatisch alle Links/Bilder/Dateien/Medien. Unterwegs Links reinwerfen, später geordnet abrufen.
- **Skills & Tools-Seite**: hier **unbenutzte Skills abschalten** (s. Abschnitt 12 — jeder aktive Skill wird an *jede* Nachricht angehängt).
- **Profiles** (unten): deine mehreren Spezial-Agenten (Abschnitt 7).
- **Cron-UI** (Abschnitt 11) — Cron-Jobs hier **manuell anlegen = 100 % sicher**, dass sie wirklich existieren.
- **Mission Control** + Memory-Bereich (kann dein Obsidian-Gedächtnis einblenden).
- **Mehrere Rechner**: Desktop-App auf jedem Hermes-Rechner installieren — „verbindet sich automatisch mit allen".

### 3.4 Funktionsprüfung
```powershell
hermes doctor
```
Sagt genau, was fehlt. `hermes: command not found` → PowerShell neu öffnen.

---

## 4. Schritt 2 — Provider & Modell (der wichtigste Schritt) **[Alex-Korrektur ggü. v1]**

Starte den Assistenten:
```powershell
hermes model        # nur Provider/Modell
hermes setup        # kompletter Assistent
```

### Alex' reale Empfehlung für die Hermes-Harness
- **Opus 4.8 für Hermes/OpenClaw** — *nicht* Fable. Begründung **[Alex]**: „Hermes uses outrageous amounts of tokens"; Fable würde deine Limits „absurd schnell" sprengen. **Opus ist der Sweet Spot** für die Agent-Harness. (Fable 5 gehört in **Claude Code / Claude Desktop** zum Programmieren — nicht in Hermes.)
- Opus **4.8 statt 4.7** (gleicher Preis). Auf dem **$200-Plan**: in Claude Code **Fast-Mode** = ⅓ des Preises.
- **Brain/Muscle [Alex]**: teures Modell (Opus) nur als „Gehirn" zum Planen/Reviewen; die „Muskel"-Arbeit an günstigere Modelle/CLIs delegieren (Codex 5.5, Sonnet, oder ein lokales Modell). *Wichtig:* als „Muskel" lieber **ein Modell/CLI direkt** rufen, **nicht** einen zweiten Hermes-Agenten — sonst wird dessen ganzer Skills-/Memory-Kontext mitgeschickt (Token-Bloat).
- Alternativen: **GPT-5.5** ist als Agent-Modell brauchbar; **ChatGPT-OAuth** und **xAI-Grok-OAuth** funktionieren nativ in `hermes model` / `hermes tools` (kein API-Key nötig).

### Provider-Wahl in der Praxis
1. **Anthropic / Claude (Opus 4.8) — Alex' Wahl für die Harness.** Per OAuth (Anthropic-Plan) oder API-Key. Du bleibst im Claude-Ökosystem, das du kennst. → **Die Auth-Wahl entscheidet über deine Kosten — siehe Warnblock unten.**
2. **Nous Portal — einfachster Rundum-Start.** Ein Abo deckt 300+ Modelle **plus Tool Gateway** (Web-Suche, Bildgenerierung, TTS, Cloud-Browser) ab — praktisch für Content/Recherche, wenn du keine Einzel-Keys jonglieren willst:
   ```powershell
   hermes setup --portal
   ```
3. **OpenRouter** — ein Key, viele Modelle, flexibles Routing.

> Wechsel jederzeit per `hermes model` — keine Bindung. **Empfehlung für dich (Stand 15.6.2026):** mit **ChatGPT/Codex-OAuth (GPT-5.5)** starten — glatter OAuth-Pfad, ChatGPT-Plus-Abo vorhanden; Anthropic-OAuth ist (noch) die raue Ecke (siehe Warnblock). **Opus 4.8 später als zweites Profil (Orchestrator/„Gehirn")**, sobald der Agent-SDK-Credit geclaimt + verifiziert ist — ein low-volume-Orchestrator passt gut in die $100/Mo-Deckelung (Max 5x). Nous Portal als Option für gebündelte Content-Tools.

### ⚠️ Anthropic-Auth: OAuth (Subscription-Credit) vs. API-Key — der Risiko-Schritt **[Recherche 15.6.2026]**

Für Anthropic/Claude gibt es **zwei** Auth-Wege, und die Wahl entscheidet, ob du den Anthropic **Agent-SDK-Monats-Credit** nutzt oder API-Geld verbrennst:

| Auth-Modus | Wie einrichten | Billing |
|---|---|---|
| **OAuth / Subscription** *(empfohlen)* | `hermes model` → **Anthropic OAuth** (Browser-Login) | Läuft über deine **Claude-Subscription** und zieht den **Agent-SDK-Credit** (Max 5x = **$100/Mo**, Max 20x = $200/Mo — separater Topf seit **15.6.2026**; *Deckelung, kein „unbegrenzt"* — danach Pay-as-you-go) |
| **API-Key** | `ANTHROPIC_API_KEY` setzen | **Pay-as-you-go** zu API-Raten — **kein** Credit |

**So funktioniert OAuth:** Hermes authentifiziert „als Claude Code" gegen deinen Anthropic-Account und nutzt **Claude Codes eigenen Credential-Store** (`%USERPROFILE%\.claude\.credentials.json`, generisch `~/.claude/.credentials.json`) — kopiert den Token also *nicht* nach `…\hermes\.env`, damit er refreshbar bleibt.

> **⚠️ Das ist Hermes' fehleranfälligste Ecke** (anders als das glatte ChatGPT-/Grok-OAuth). Offene Issues: Credential-Management + Subscription-Routing ([#12905](https://github.com/NousResearch/hermes-agent/issues/12905)), Token-Mismatch zwischen Stores ([#21107](https://github.com/NousResearch/hermes-agent/issues/21107)), sauberes Codex-artiges Subscription-OAuth erst als Feature-Request ([#25267](https://github.com/NousResearch/hermes-agent/issues/25267)). **Plane hier Puffer ein — trickigste Stelle des ganzen Setups.**

**Zwei Foot-Guns:**
1. **Kein `ANTHROPIC_API_KEY` in der Umgebung lassen** — eine gesetzte Env-Var **überschreibt** den OAuth-Pfad → stilles Pay-as-you-go, Credit umgangen. Vor dem OAuth-Login prüfen, dass sie leer ist: `echo $env:ANTHROPIC_API_KEY` (PowerShell).
2. **Geteilter Credential-Store:** Hermes und Claude Code teilen sich `…\.claude`. Halten beide unterschiedliche OAuth-Tokens, kann die Auth brechen (#21107-Muster; auf Windows ohne Keychain entschärft, der Routing-Konflikt bleibt aber möglich).

**Verifizieren (sobald der Credit geclaimt ist, ab 15.6.2026):** Nach dem ersten Anthropic-OAuth-Chat prüfen, dass die Nutzung wirklich den **SDK-Credit** zieht und **nicht** als API-Spend auf einem Key landet. Bis der OAuth-Pfad sauber steht, trägt ein **günstigeres/lokales Fallback-Modell** (siehe „Lokale Modelle" unten / Qwen) die Zwischenzeit.

### Lokale Modelle (optional, später) **[Alex]**
Auf **DGX Spark / Mac Studio** (ideal 128 GB; Sweet-Spot-Modellgröße **27–35 B**, z. B. **Qwen 3 27B**): für triviale/kostenlose Daueraufgaben (z. B. Web-Scraping-Loops alle 20 Min). Einrichtung per Zuruf: „Hermes, lade & starte ein passendes lokales Modell für diese Hardware." **Tailscale** nur nötig, wenn das Modell auf einem *separaten* Gerät läuft. **Mac-Mini-Warnung [Alex]:** Chip ist für lokale Modelle schwach — experimentell ok, aber kein starkes Modell erwartbar. Stand 6/2026: **DGX Spark ~$4.800** (128 GB, plug-and-play, kein Monitor nötig; NVIDIAs neue **Nemotron**-Open-Models sind Spark-first); **Mac Studio** wäre Alex' Präferenz (mehr Unified Memory → größere/„kühlere" Modelle), war aber außer in Kleinst-Configs ausverkauft. **Kosten als Investment [Alex]:** „$200 für Claude, $4.800 für den Spark — das sind Investitionen in dich, die einen ROI bringen sollen" (distinkt zu Konsum-Abos wie Netflix). Greg Isenbergs Gegenprobe: erst mit der kostenlosen App Wert beweisen, dann skalieren.

### Wie Einstellungen gespeichert werden
- Geheimnisse/Tokens → `…\hermes\.env`  · Normale Einstellungen → `…\hermes\config.yaml`
```powershell
hermes config set model anthropic/claude-opus-4.8
hermes config set OPENROUTER_API_KEY sk-or-...
```

### Kosten im Griff **[Alex]**
Die verbreitete „$1.000/Monat"-Klage ist fast immer ein **Kontext-Hygiene-Problem**, kein Hermes-Problem: „If you manage your contexts and sessions well, you are not paying anything close to it." Drei Hebel:
- **`/new`** im Chat leert ein vollgelaufenes Kontextfenster mitten in der Session.
- **Sessions je Thema** trennen (Abschnitt 5) — hält jede Nachricht schlank.
- **Pro Aufgabe Modell + Effort dosieren** — z. B. „Opus 4.8 mit minimalem Effort" oder „Haiku" für Aufgaben ohne viel Denkbedarf. Opus' großes Kontextfenster kann „deine Rechnung 3-4× erhöhen", wenn man es volllaufen lässt.

---

## 5. Schritt 3 — Erster Chat & Sessions

Starte die **Desktop-App** oder im Terminal:
```powershell
hermes            # klassische CLI
hermes --tui      # modernes Terminal-UI
```
Test mit etwas leicht Überprüfbarem („Schau in mein aktuelles Verzeichnis und sag mir, was wie die Hauptdatei aussieht."). **Erfolg:** Banner zeigt dein Modell, Hermes antwortet fehlerfrei, nutzt bei Bedarf ein Werkzeug, läuft über mehrere Runden.

**Sessions** (Verlauf — Basis fürs Gedächtnis):
```powershell
hermes --continue / hermes -c    # letzte Session fortsetzen
hermes sessions list             # alle Sessions
```
**[Alex]** Lege **je Thema eine eigene Session** an (Content/Coding/Recherche…) statt alles in einen Thread zu kippen. Slash-Befehle im Chat: `/help`, `/tools`, `/model`, `/save`.

> **Recovery-Reihenfolge (Doku):** `hermes doctor` → `hermes model` → `hermes setup` → `hermes sessions list` → `hermes --continue`.

---

## 6. Schritt 4 — Das Gehirn: Alex' 4-Schichten-Memory-System (Zweck a)

Das ist der Teil, der Hermes über die Zeit **besser** macht. Alex hat (eigene Worte) „die letzten Tage hart am Rebuild des Hermes-Gedächtnisses gearbeitet", weil das Standard-Gedächtnis schwächer war als OpenClaws und „jede Kompaktierung praktisch eine Lobotomie" war. Seine Lösung: eine **Obsidian-Vault-Schicht** obendrauf. **Funktioniert für Hermes UND OpenClaw identisch.**

### 6.1 Obsidian installieren & Vault anlegen
1. **Obsidian** (kostenlos) von `obsidian.md` installieren. (Obsidian ist nur eine grafische Oberfläche über Markdown-Dateien.)
2. Vault-Ordner anlegen, z. B. `C:\Users\manyw\HermesBrain`.
3. Pfad an Hermes geben (der mitgelieferte Skill `note-taking/obsidian` ist „filesystem-first" und liest/schreibt die `.md`-Dateien direkt):
   ```powershell
   hermes config set OBSIDIAN_VAULT_PATH "C:\Users\manyw\HermesBrain"
   ```
4. **[Alex]** **Sichere den ganzen Agent-Ordner in ein privates GitHub-Repo** („Erdbeben-/Laptop-tot-Versicherung").

### 6.2 Die 4 Schichten (Alex' „offizielles" System)
Von „immer präsent / klein" zu „on-demand / groß":

1. **Schicht 1 — Built-in Memory (~2.200 Zeichen).** Wird in *jeden* Prompt automatisch injiziert. Nur kompakte Fakten/Zeiger: dein Name, Vault-Pfad, häufige Befehle. „Wie Klebezettel am Monitor — immer sichtbar."
2. **Schicht 2 — `AGENTS.md` + `SOUL.md`.** Ebenfalls in jeden Prompt injiziert. Betriebsanweisungen, Persönlichkeit, harte Regeln, **verbindliche Logging-Regeln**. Die „Wie-verhalte-ich-mich"-Schicht.
3. **Schicht 3 — Obsidian-Vault (der Arbeitspferd-Layer).** *Nicht* auto-injiziert; der Agent **liest** ihn bei Session-Start und während der Arbeit. Alex' Ordnerstruktur (auf Mehr-Agenten ausgelegt):
   - **`Agent-Shared/`** (alle Agenten lesen/schreiben): `user-profile.md` (wer du bist, Präferenzen, Korrekturen), `project-state.md` (alle Projekte + Status), `decisions-log.md` (Entscheidungs-Historie).
   - **`Agent-Hermes/`** (privater Arbeitsbereich): `working-context.md` (woran gerade gearbeitet wird), `mistakes.md` (was schiefging), `daily/` (ein Logfile pro Tag).
   - **`Agent-OpenClaw/`** (falls du später OpenClaw parallel nutzt — Hermes fasst es nicht an).
   - **Regel:** geteilte Dateien dürfen **nie überschrieben** werden (nur ergänzen).
   - **LESEN bei**: Session-Start, nach Kompaktierung, wenn Details gebraucht. **SCHREIBEN bei**: Task-Start, alle 3–5 Tool-Calls, Task-Ende, Korrekturen.
4. **Schicht 4 — Session-Search.** Durchsuchbares Archiv aller bisherigen Gespräche (automatisch, du schreibst nicht hinein). Letzte Instanz: „Was haben wir letzte Woche zu X gemacht?"

**Der Fluss:** Neue Session → Vault lesen (user-profile, project-state, working-context, heutiges Log) → arbeiten, alle 3–5 Tool-Calls in den Vault checkpointen → Task fertig: an Tageslog anhängen + working-context aktualisieren → Kompaktierung? Todo-Liste überlebt, danach Vault neu lesen → Session-Ende: alles ins Tageslog flushen.

> **Einrichtung:** Den fertigen Setup-Prompt dafür findest du in **Anhang A** — einmal an Hermes geben, er baut die Struktur selbst.

### 6.3 Optional: eine Wissens-Schicht (raw / wiki / output)
Alex' System ist **agent-Gedächtnis** (Kontinuität über Sessions). Für *Recherche-Wissen* (passt zu deiner Bibliothek/Dashboard-Welt) kannst du im Vault zusätzlich eine **Wissensbasis** anlegen — das community-erprobte 3-Schichten-Muster: **`raw/`** (unveränderliche Quellen, append-only) → **`wiki/`** (vom Agenten synthetisiertes Wissen mit YAML-Frontmatter + ≥2 Querverweisen) → **`output/`** (erzeugte Reports/Briefings). Setup-Prompt dafür ebenfalls in Anhang A (Teil 2). Domänen z. B. `theologie`, `philosophie`, `makro`, `ai-dashboard`, `buttcoin`.

---

## 7. Profiles vs. Sub-Agents (wichtig zu verstehen) **[Alex]**

- **Profile = ein eigenständiger, voller Agent** mit *eigenem* Gedächtnis, eigenen Skills, eigenem Modell. Alex betreibt **7 Profiles**. Er organisiert sie **nach Modell-Stärke** (z. B. ein Opus-Profil, ein GPT-5.5-Profil, ein lokales „Qwen"-Profil), **nicht** nach menschlichen Rollen.
  > **[Alex] Anti-Pattern:** das „Paperclip"-Org-Modell (PM-/CTO-/CEO-/Designer-Agenten) sieht cool aus, ist aber ineffizient — „liked it for 10 minutes". Profile + Prompts wechseln ist effizienter, als Agenten in Firmen-Rollen zu pressen.
- **Sub-Agent = eine Kopie *eines* Agenten** (gleiche Skills/Memory, eigener Kontext) für Multitasking derselben Art Aufgabe. Sie koordinieren sich, um sich nicht in die Quere zu kommen.

**Faustregel:** Brauchen Aufgaben *unterschiedliche* Fähigkeiten (Recherche vs. Schreiben vs. Bild) → **Profiles**. Ist es dieselbe Art Arbeit, nur parallel → **Sub-Agents**.

Für deinen Start: **ein** Hermes-Profil reicht. Profiles kommen, wenn du Content-Pipelines (Abschnitt 10) oder lokale Modelle aufsetzt.

---

## 8. Schritt 5 — Aufgaben: Linear anbinden (Zweck a) **[Alex-Praxis]**

**Alex' Haltung:** Linear ist ein **„zweites Gehirn zum Organisieren", kein Orchestrator.** Drei Ebenen: **Teams** (eine pro App/Kunde) → **Projects** (Feature-Gruppe/Phase) → **Issues** (Einzelaufgaben). Er **fasst Linear kaum manuell an** — der (Coding-)Agent legt Projekte/Issues an, füllt Details, setzt Status, hängt Branches dran. „Manchmal öffne ich Linear tagelang nicht."

### 8.1 Anbinden — über den Hermes-MCP-Katalog (empfohlen)
Linear ist im kuratierten Hermes-MCP-Katalog:
```powershell
hermes mcp                 # interaktiver Picker
hermes mcp install linear  # Linear (remote OAuth)
```
- **OAuth-Flow**: Browser öffnet → in Linear einloggen & freigeben.
- **Tool-Auswahl** (Checkliste): aktiviere z. B. `find_issues`, `get_issue`, `create_issue`, `update_issue`; lass Gefährliches (`delete_workspace`) aus.
- Klappt OAuth erst später: `hermes mcp login linear` aus **frischem** Terminal.

Alternativ manuell in `config.yaml`:
```yaml
mcp_servers:
  linear:
    url: "https://mcp.linear.app/mcp"
    auth: oauth
```
Nach Config-Änderung im Chat: `/reload-mcp`.

### 8.2 Workflow **[Alex]**
1. Feature erst im Chat (Plan-Modus) durchdenken → dann: „Lege dafür ein Linear-Projekt + alle Issues mit Details an."
2. „Wo fangen wir an?" → ein Issue wählen → Stück für Stück bauen.
3. Wieder einsteigen: **„Wo waren wir?"** — der Agent liest Linear und weiß, wo es weitergeht. Oder: „Was ist das nächste hochwertige Issue?"
4. **Disziplin:** jede Code-Änderung = ein Linear-Issue; je Issue (bzw. je Feature) ein eigener **GitHub-Branch**; erst nach Test mergen. (Solo-Schnellbauten dürfen das überspringen.)
5. Alex referenziert aus seiner `AGENTS.md` ein Template `ops/templates/linear-task.md` (Projekt-Root, Ordner `ops/`). **Beide Verbatim-Vorlagen liegen für dich bereit:** `templates/AGENTS_example_AlexFinn.md` und `templates/linear-task_example_AlexFinn.md`.

> **Hinweis:** Dieser Linear-Workflow ist coding-zentriert — also primär *mein* (Claude-Code-)Revier. Für Hermes als Alltags-Assistent nutzt du Linear v. a., um Aufgaben zu erfassen/abzufragen.

### 8.3 Orchestrierung = Hermes-Kanban, nicht Linear **[Alex]**
Für Mehr-Agenten-Aufgabenverteilung nutzt Alex den **Hermes-Kanban-Board** (`hermes dashboard` → Kanban): Aufgabe in die **Triage**-Spalte werfen → Hermes zerlegt sie in Subtasks und weist sie Sub-Agenten zu (Triage → To-Do → Ready → In Progress → Blocked → Done). Seine Routine: morgens automatisierbare To-Dos in Triage schieben, weggehen, später fertig wiederkommen.

---

## 9. Schritt 6 — Immer erreichbar: Messaging-Gateway (Zweck d)

Damit Hermes dein „AI-Employee" wird, willst du ihm **von überall** schreiben — am einfachsten **Telegram** (auf Mobil; auf dem Desktop nutzt Alex die Desktop-App).
```powershell
hermes gateway setup     # Plattform + Token einrichten
```
Wählbar: **Telegram, Discord, Slack, WhatsApp, Signal, E-Mail, MS Teams, Home Assistant.** Telegram-Bot via `@BotFather` → Token in den Setup-Flow.
```powershell
hermes gateway run       # Vordergrund (Test)
hermes gateway install   # als Hintergrunddienst (immer an)
hermes gateway start / status
```
**[Alex]** Nutze **Telegram-Topics** (Threads je Domäne: Content/Programming/…) statt ständig neue Sessions. 24/7-Betrieb braucht einen durchlaufenden Gateway-Prozess (PC an, oder später VPS-Remote-Backend).

---

## 10. Schritt 7 — Content & Social (Zweck c) **[Alex-Praxis]**

### 10.1 Multi-Profile-Content-Pipeline **[Alex]**
Statt eines Allzweck-Agenten: getrennte **Profiles** je Schritt — **Research-Agent** (recherchiert) → **Writing-Agent** (Skript) → **Image-Agent** (Thumbnail) → **Social-Agent** (Tweets). (Profiles = Abschnitt 7.)

### 10.2 X/Twitter
- Skill **`social-media/xurl`** (Posten, Suchen, DMs, Medien über die X-v2-API). Braucht **X-API-Zugang** (jetzt pay-per-use, „ein paar Cent pro Call").
- **[Alex] Grok-OAuth** in Hermes ist **stark für KI-Bild-/Video-Generierung** und um **Trends zu scannen** — kein API-Key nötig, direkt X durchsuchen. Aktivieren: `hermes tools` → Grok hinzufügen. (Community-Beispiel: 2–3×/Tag Trends im eigenen Niche scannen → Reply-/Quote-Post-Vorschläge + 1 Content-Idee je Trend.)

### 10.3 Opportunity-/Signal-Finder **[Alex]** (sein Vorzeige-Use-Case)
Ein Cron-Job (bei Alex alle **20 Min** auf dem lokalen Qwen) liest **Reddit + X nach fremden Schmerzpunkten** und schreibt das Ergebnis in ein **eigenes HTML-Dashboard**. Jeder Eintrag enthält: das **Problem**, den **Quell-Link/Thread** (damit du die echten Kommentare liest), **warum gerade du** es lösen kannst (der Agent kennt deine Skills/Assets/Community), und den **ersten Schritt** — bei passenden Fällen baut er per Button sogar einen klickbaren **Micro-SaaS-Prototyp**. Die 20-Min-Taktung lohnt nur lokal (gratis); mit Cloud-Modell (Opus/ChatGPT) **1×/Morgen**, sonst „pay out the wazoo".
> **[Alex/Greg] Das ist der eigentliche Geld-Hebel:** Richte die Agenten auf **fremde** Probleme, nicht auf Spielereien — „find other people's challenges and then solve those challenges". Als Solopreneur mit niedrigen Kosten kannst du auch winzige Nischen profitabel bedienen.

### 10.4 Nützliche Content-Skills
- **`creative/humanizer`** — entfernt „KI-Sprech" (wichtig, damit Posts echt klingen).
- **`media/youtube-content`** — Transkript → Zusammenfassung/Thread/Blog.
- **`creative/claude-design`, `sketch`, `popular-web-designs`** — HTML-Artefakte/Mockups.

### 10.5 Alex' Distributions-Philosophie (übertragbar)
- **Distribution ist Moat #1.** Erst Publikum, dann Produkt.
- **Auf X anfangen → zu YouTube wachsen.** X baut „den Muskel, Interessantes zu sagen". Quote-Tweeting als Übung. **Cadence: 2×/Tag**, gerade am Anfang. Themen nicht mischen (off-Niche = ~0 Engagement) → ggf. zweiter Kanal.
- **Velocity vor Produktion.** YouTube-„Brain" in **Notion** (Ordner `Video Plans`, je Video ein Template mit Feldern: *title · other titles · the tweet · first 7 seconds · script · editing notes · description notes*). Aufnahme mit **Tella** (Face+Screen), ~80 % improvisiert aus High-Level-Stichpunkten; Schnitt in Tella: „Remove Silences" + Layout/Cut.

---

## 11. Schritt 8 — Autonomie: Cron, Kanban & „Loops"

### 11.1 Cron-Jobs
Zwei Sorten: **Gehirn-pflegend** (Vault aufräumen, Indizes, veraltete Seiten, Gespräche kategorisieren) und **Aufgaben-ausführend** (Trends/Releases, Tages-Digest, Content-Entwürfe).
```powershell
hermes cron list / create / edit / pause / resume / run / remove / status
```
**[Alex] Tipp:** Cron-Jobs **in der Desktop-Cron-UI manuell anlegen** = 100 % sicher, dass sie wirklich existieren (häufiger Fehler: „ich hab ihm gesagt, einen Cron zu planen, aber er tut's nicht").

**Empfohlener Start (genau 2 Jobs):**
1. *Täglicher Gehirn-Digest* — „Fasse morgens neue raw/-Quellen zusammen, aktualisiere die wiki-Seiten, schreibe einen kurzen Tagesüberblick nach output/reports/."
2. *Ein Aufgaben-Cron* — z. B. Opportunity-Scan (Abschnitt 10.3), Quellen-Monitoring, oder ein **Morning-Brief** (per Reverse-Prompt erstellt → Abschnitt 14.0).

### 11.2 Loops (fortgeschritten) **[Alex]**
Ein „Loop" lässt einen Agenten **dauerhaft ohne Aufforderung** arbeiten. Alex' Coding-Loop (primär *mein* Revier): **`/spec`** (du brain-dumpst Features morgens; der Skill interviewt dich, schreibt Spec, legt je ein Linear-Issue an) → **`/build`** (per `/loop /build`, z. B. alle 20 Min: greift unclaimed Specs, baut jede auf einem eigenen Preview-Branch) → **`/review`** (ein *anderes* Modell prüft gegen die Acceptance-Criteria). Blocker melden sich in einem Slack-Kanal `#linear-block`; **Merge-to-Production bleibt menschlich** („crazy to auto-merge to production"). Für dich relevant v. a. als Konzept — den Coding-Loop fahre ich.

---

## 12. Skills & MCP — Erweiterung (mit Disziplin) **[Alex]**

**Skills** sind „on-demand"-Anleitungs-Dokumente (`SKILL.md`). Wichtig: **Jeder *aktivierte* Skill wird an *jede* Nachricht angehängt** (Hermes prüft die Relevanz) → kostet Tokens, macht langsamer, verschlechtert Ergebnisse.
> **[Alex] Regel:** Geh in **Skills & Tools** (Desktop) bzw. `hermes skills config` und **schalte ab, was du gerade nicht brauchst**. Regelmäßig aufräumen. **Lade nicht alle Skills auf einen Agenten** — dafür sind Profiles da.
> **[Alex] Versteckter Kosten-Treiber:** Hermes **erzeugt im Hintergrund selbst Skills**, während du arbeitest (z. B. aus einem gebauten Mini-Game oder Skript) — die zählen ebenfalls zum Kontext. Also in der Skills-UI regelmäßig prüfen und Unnötiges deaktivieren. **„Tool Sets"** (neu, noch im Rollout) bündeln mehrere Skills/Tools für komplexe Jobs.

```powershell
hermes skills browse / search <wort> / install <source/path> / list / config
```
Jeder installierte Skill wird automatisch **Slash-Befehl** (`/obsidian`, `/xurl`). Oft musst du gar nichts explizit aufrufen — Hermes wählt selbst.

Relevante mitgelieferte Skills: `note-taking/obsidian`, `productivity/{notion,google-workspace,airtable,ocr-and-documents,powerpoint}`, `social-media/xurl`, `research/{arxiv,blogwatcher,polymarket}`, `creative/{humanizer,claude-design}`, `media/youtube-content`, `autonomous-ai-agents/{claude-code,codex}`.

**MCP** bindet externe Dienste an (Abschnitt 8 = Linear). Katalog: `hermes mcp` / `hermes mcp install <name>` / `hermes mcp configure <name>`. Manuell in `config.yaml` unter `mcp_servers:` (stdio: `command/args/env`; HTTP: `url/headers`; OAuth: `auth: oauth`). Migration von OpenClaw: Skill `official/migration/openclaw-migration`.

---

## 13. Sicherheit — bitte lesen

Ein Agent mit Terminal-, Datei- und Tool-Zugriff ist mächtig.
1. **YOLO-Mode = AUS lassen** (Standard). Umgeht Bestätigungs-Abfragen vor gefährlichen Befehlen. Nur bewusst/sessionweise an.
   > **[Alex]** fährt aus Tempo-Gründen „Full-Access/Auto-Mode" überall und übernimmt explizit *keine* Verantwortung für Schäden. **Meine Empfehlung für dich:** anfangs **mit** Bestätigungen arbeiten, bis du dem Setup vertraust — gerade weil Hermes auf deine echten Dateien zugreift.
2. **Terminal sandboxen** bei riskanteren Aufgaben:
   ```powershell
   hermes config set terminal.backend docker   # Container
   hermes config set terminal.backend ssh      # entfernter Server
   ```
3. **Geheimnisse/MCP-Vertrauen:** Keys in `.env` (nicht teilen/committen). MCP-Katalog-Manifest vor Install prüfen (`source:`, `bootstrap`). Sensible Server nur per **Whitelist** (`tools.include: [...]`).
4. **Remote-Dashboard** nie offen ins Internet — hinter VPN (Tailscale) oder OAuth.

---

## 14. Täglicher Workflow & Alex' Kern-Tipps **[Alex]**

### 14.0 Die mächtigste Einzeltechnik: Reverse Prompting / Brain Dump **[Alex]**
Alex' „Lieblings-Kombo": erst **roh brain-dumpen**, was du willst — dann **den Agenten den optimalen Prompt schreiben lassen**, statt selbst zu formulieren. Seine Antwort auf fast jedes „Wie mache ich X am besten?": *„The answer is always reverse prompting. Always."*
Beispiel (Morning-Brief): du sagst sinngemäß *„I want to set up a morning brief at 9am. My interests are AI, stock investing, tech, and the Boston Celtics. What would be the best prompt I can use to set this up with you?"* — der Agent liefert einen Prompt, der u. a. erzwingt: *„Pull fresh information from the last 24 hours. Do not rely on memory or stale data. Use web search and exact real headlines, prices, and scores"* (gegen veraltete Trainingsdaten) und *„Lead with a one-line vibe summary. Use bold letters and bullets"* (lesbares Briefing statt Textwüste). Diesen erzeugten Prompt dann als Cron-Job (9:00) speichern. **Nutze das für alles** — Vault-Aufbau, Linear-Setup, Content — gerade als Anfänger gegen „unknown unknowns".

- **In kleinen Schritten arbeiten.** Nach jedem Schritt: „Was hast du gerade getan?" — Kontrolle behalten, mitlernen.
- **„Use agents to build agents."** Nutze Claude Code/Codex zum Installieren/Konfigurieren von Hermes und zum Schreiben von Skills. (Bei dir: *ich* übernehme Coding/Setup-Feintuning.)
- **Wenn der Agent „kaputt" ist:** Terminal in den Hermes-Ordner, dort **Claude Code (oder Codex) öffnen**, Problem beschreiben, fixen lassen — bei Alex „100 % Erfolgsquote". (Alternativ: „update yourself" / `hermes doctor`.)
- **Ändert ein Modell sein Verhalten** (z. B. plant statt auszuführen): das Modell ist gleich → der Unterschied liegt in der Harness → **Regel-Datei + Persönlichkeits-Datei aktualisieren** statt nur mündlich zu korrigieren. Beispiel: „So hast du es früher gemacht, so machst du es jetzt — aktualisiere unsere Rules-/Personality-Datei entsprechend."
- **Globale Regeln:** Home-Verzeichnis-`.claude` / `.codex` (bzw. `.agents`) wird **vor** dem Projektordner gelesen — erspart Copy-Paste in jedes Projekt.
- **Lernpfad [Alex]:** „I'd actually start with the Claude Code Master Class" → dann OpenClaw-Masterclass; fast alles überträgt sich 1:1 auf Hermes.
- **Beobachtete Top-Nutzungen (Community):** (1) Recherche vor Calls, (2) Meeting-Notizen → Follow-up-Entwürfe, (3) Vault-/Wissenspflege.

**Dein realistischer Tagesrhythmus:** Morgens Telegram/Desktop an Hermes („Was steht an? Vault-Digest?") → tagsüber Quellen in `raw/` werfen, Hermes synthetisiert → auf Zuruf Content-Entwürfe → nachts/Cron: Vault-Pflege + Trend-Scan automatisch.

---

## 15. Schnell-Referenz (Befehle)

| Befehl | Zweck |
|---|---|
| `hermes` / `hermes --tui` / `hermes desktop` | Chat (CLI/TUI) / Desktop-App |
| `hermes setup` / `hermes setup --portal` | Voll-Assistent / Nous-Portal-Schnellstart |
| `hermes model` / `hermes config set <k> <v>` | Provider/Modell / Einstellung setzen |
| `hermes doctor` / `hermes update` | Diagnose / Aktualisieren |
| `hermes -c` / `hermes sessions list` | Letzte / alle Sessions |
| `hermes mcp` / `hermes mcp install linear` | MCP-Katalog / Linear |
| `hermes skills browse/install/list/config` | Skills verwalten (unbenutzte abschalten!) |
| `hermes gateway setup/run/install/status` | Messenger-Anbindung |
| `hermes cron list/create/edit/run` | Automationen |
| `hermes dashboard` | Web-Dashboard + Kanban-Board |
| `/reload-mcp`, `/tools`, `/model`, `/save` | Slash-Befehle im Chat |

---

## 16. Troubleshooting (Windows)

| Problem | Lösung |
|---|---|
| `hermes: command not found` | PowerShell neu öffnen (PATH) |
| `API key not set` | `hermes model` erneut, oder `hermes config set <PROVIDER>_API_KEY ...` |
| Leere/kaputte Antworten | Provider/Modell/Auth in `hermes model` prüfen |
| **Agent generell „kaputt" / nach Provider-Wechsel** | **[Alex]** Claude Code/Codex im Hermes-Ordner öffnen, Problem beschreiben, fixen lassen (100 %) |
| Agent plant statt auszuführen | **[Alex]** Rules-/Personality-Datei aktualisieren (Abschnitt 14) |
| Cron-Job „existiert nicht" | **[Alex]** in der Desktop-Cron-UI manuell anlegen |
| Zu langsam / teuer / schlechtere Antworten | Unbenutzte **Skills abschalten** (Abschnitt 12); Sessions je Thema trennen |
| Desktop-Build hängt bei „Electron download" | Installer nutzt automatisch `npmmirror.com`-Spiegel; sonst `ELECTRON_MIRROR` setzen |
| MCP-Tools fehlen | Connect/OAuth prüfen, `/reload-mcp`, ggf. `hermes mcp login <name>` frisches Terminal |
| Allgemein | `hermes doctor` |

**Alternative auf Windows:** Bei Zicken läuft Hermes sehr stabil unter **WSL2** (Linux-Installer `curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash`). Für den Anfang aber Desktop-App nativ.

---

## Anhang A — Setup-Prompts (zum Einfügen in Hermes)

### A.1 Alex' 4-Schichten-Memory-System (an Hermes geben)
```
Baue mein Langzeit-Gedächtnis als Obsidian-Vault-Schicht auf, nach folgendem 4-Schichten-System.
Mein Vault liegt unter OBSIDIAN_VAULT_PATH (siehe .env). Lies dieses System, erstelle die
Ordner/Dateien in Schicht 3 und halte dich ab sofort an die Lese-/Schreib-Regeln.

SCHICHT 1 — Built-in Memory (~2.200 Zeichen, wird in jeden Prompt injiziert):
  Nur kompakte Fakten/Zeiger (mein Name, Vault-Pfad, häufige Befehle/Pfade). Wie Klebezettel.

SCHICHT 2 — AGENTS.md + SOUL.md (in jeden Prompt injiziert):
  Betriebsanweisungen, Persönlichkeit, harte Regeln, verbindliche Logging-Regeln.

SCHICHT 3 — Obsidian-Vault (Arbeitspferd; NICHT auto-injiziert — bei Session-Start lesen):
  Ordner & Dateien anlegen:
  Agent-Shared/   (du darfst lesen+ergänzen, NIE überschreiben)
    user-profile.md     (wer ich bin, Präferenzen, Korrekturen)
    project-state.md    (alle Projekte + Status)
    decisions-log.md    (Entscheidungs-Historie)
  Agent-Hermes/   (dein privater Arbeitsbereich)
    working-context.md  (woran du gerade arbeitest)
    mistakes.md         (was schiefging + Lehre)
    daily/              (ein Logfile pro Tag, YYYY-MM-DD.md)
  LESEN bei: Session-Start, nach Kompaktierung, wenn du Details brauchst.
  SCHREIBEN bei: Task-Start, alle 3-5 Tool-Calls, Task-Ende, Korrekturen.

SCHICHT 4 — Session-Search:
  Durchsuchbares Archiv vergangener Gespräche (automatisch). Letzte Instanz für „was haben wir
  letzte Woche zu X gemacht?".

FLUSS: Neue Session -> Vault lesen (user-profile, project-state, working-context, heutiges Log)
-> arbeiten, alle 3-5 Tool-Calls checkpointen -> Task fertig: an Tageslog anhängen +
working-context aktualisieren -> nach Kompaktierung Vault neu lesen -> Session-Ende: alles ins
Tageslog flushen.

Lege jetzt die Ordnerstruktur in Schicht 3 an, erzeuge leere Startdateien mit kurzen Kopfzeilen,
und bestätige, dass du die Lese-/Schreib-Regeln ab jetzt befolgst.
```

### A.2 Optionale Wissensbasis (raw / wiki / output) — zusätzlich zum Memory-System
```
Lege im selben Vault zusätzlich eine Wissensbasis als 3-Schichten-Architektur an:
1. raw/    = unveränderliche Quellen (append-only; nie editieren) — Unterordner: articles/ papers/
             competitors/ repos/ tweets/ misc/
2. wiki/   = von dir synthetisiertes, strukturiertes Wissen; je Seite YAML-Frontmatter
             (title, type, domain, tags, sources, created, updated, confidence: high|medium|low)
             und mind. 2 [[Wikilinks]]. Domänen: theologie, philosophie, makro, ai-dashboard,
             buttcoin, concepts.
3. output/ = erzeugte Lieferobjekte (reports/ slides/ charts/) — aus wiki abgeleitet, nie Quelle.
Regeln: vor Anlegen neuer Seiten erst nach existierenden suchen (keine Duplikate); jede
bedeutsame Aktion an log.md anhängen; eine zentrale SCHEMA.md mit Konventionen/Tags/Namensregeln.
Lege SCHEMA.md, log.md, wiki/_index.md, templates/article.md und templates/raw-source.md an,
dann die volle Ordnerstruktur.
```

## Anhang B — Alex' verbatim Templates (Coding-Workflow)
- `templates/AGENTS_example_AlexFinn.md` — seine Rules-Datei (Default-Workflow, PR-Standard, PR-Review-Standard).
- `templates/linear-task_example_AlexFinn.md` — sein Start-Prompt je Linear-Issue + Issue-Struktur.

Diese sind coding-zentriert (npm/convex) und damit primär *mein* Revier — ich passe sie an deine Projekte an, wenn wir Hermes/Claude-Code für App-Bauten einsetzen.

---

### Quellen
- **Offizielle Hermes-Doku** (`hermes-agent.nousresearch.com/docs`): Installation, Quickstart, Desktop-App, Skills-Katalog, Obsidian-Skill, MCP, CLI-Befehle. NousResearch/hermes-agent (GitHub).
- **Alex Finn / Vibe Coding Academy** (von dir exportiert): Bootcamp #14–#17 + „Loop"-Session-Transkripte, Live-Call-PDF, Posts „My OFFICIAL agent memory system" (1.4.2026), „managing Linear" (30.5.2026), „Fix issues with Agents", „Grok OAuth", „GitHub". Plus öffentliche X-Posts `@AlexFinn`.
- **Interview Greg Isenberg × Alex Finn** „Hermes Agent Desktop: Full Setup + Real Use Cases" (YouTube `@GregIsenberg`, 6.6.2026, 43:48) — Reverse-Prompting, Opportunity-Scan, Kosten-Hygiene, Cost-as-Investment, OpenClaw-Wechselgründe.
- **Community-Praxis:** Max Mitcham „AI Agent Operating System" (raw/wiki/output + Cron-Strategie).
