# Pending Prompt — Hermes-Setup Phase 4 (Werkzeuge: Grok-Tool + Skills-Ausmist)

**Kern:** Hermes-Agent-Setup fortsetzen mit **Phase 4 — Werkzeuge**. Phase 1–3 (Basis / Gedächtnis / Erreichbarkeit) sind DONE — siehe Memory `project_hermes_agent`.

**Aufgabe Phase 4:**
1. **Grok als Tool anbinden** (SuperGrok / X-OAuth, kein eigenes Profil): `hermes tools` → Grok hinzufügen. Zweck: KI-Bild-/Video-Generierung + X-Trend-Scan (kein API-Key nötig). Live-Setup-Disziplin: Chris führt OAuth aus, Claude begleitet + verifiziert (kurzer Funktionstest: 1 Bild + 1 Trend-Scan).
2. **Skills/Tools-Ausmist-Pass** (Token-Bloat-Hygiene, Alex-Finn-Warnung „Hermes uses outrageous amounts of tokens"): geladene Skills/Tools durchsehen, ungenutzte deaktivieren → schlanke, zweckgebundene Profile (Primär = Modell, Sekundär = Zweck-Scoping).

**WICHTIG — bewusst NICHT in Hermes:** **Linear bleibt draußen** (gehört zur Claude-Code-Build-Achse → `project_linear_build_axis`). In Phase 4 NICHT einrichten.

**Reihenfolge:** erst Grok-Tool (+ Funktionstest), dann Skills-Ausmist.

**Kontext:** Verlauf + wiederverwendbare Prompts/Befehle in `Hermes/Hermes-Setup Doku 15062026.md` (Abschnitt 6 = Phasen, Anhang A = Prompts; A.4 = Phase-3-Befehle). Danach folgt Phase 5 (Autonomie/Cron + Scout-Rolle). Auto-Pickup-Regel: Kern kurz nennen + anbieten, **nicht ungefragt starten**.

**Separat geparkt (NICHT Teil von Phase 4):**
- **Opus-4.8-Orchestrator-Profil** → erst nach SDK-Credit-Claim + Verifikation (Anthropic-OAuth = raue Ecke).
- **Buttcoin-Content-System „rund machen"** → eigener Design-Pass, startet mit Failure-Diagnose der Vorversionen → `project_buttcoin_content_system`.

---
*Selbst-Reset: Nach Ausführung von Phase 4 diese Datei auf `# Pending Prompt — (leer)` zurücksetzen.*
