# Pending Prompt — Hermes-Setup Phase 5 (Autonomie: Morning-Brief-Cron + Scout-Rolle)

**Kern:** Hermes-Agent-Setup fortsetzen mit **Phase 5 — Autonomie**. Phase 1–4 (Basis / Gedächtnis / Erreichbarkeit / Werkzeuge) sind DONE — siehe Memory `project_hermes_agent` + `Hermes/Hermes-Setup Doku 15062026.md` (Abschnitt 6 = Phasen, Anhang A.5 = Phase-4-Befehle).

**Aufgabe Phase 5:**
1. **Morning-Brief-Cron** via Reverse-Prompting einrichten (`hermes cron` UI; täglicher Brief). Vorher mit Hermes durchsprechen, was rein soll (Kalender/Mail/Trends/Projekt-Status) — schlank halten, Token-Hygiene.
2. **Scout-Rolle** etablieren: personalisierte Ideen-/Prototyp-Vorschläge aus Personenkenntnis, **Nordstern-gebunden + ruthless Triage**; graduierte Vorschläge → Linear/PRD → Claude Code baut (Brücke Assistenz↔Coding). Pattern aus `project_hermes_agent` (Flächen-Ordnung) + [[feedback_synthesize_focus_anti_sprawl]].
3. Live-Setup-Disziplin: Chris führt interaktive `hermes`-Schritte/Cron-Anlage aus, Claude begleitet + verifiziert.

**Optionaler Auftakt zu Phase 5 — Desktop-App (nur wenn konkret gebraucht):** Profiles-/Cron-UI ist für Multi-Profil/Autonomie nützlich. **NICHT** den Source-Build (`hermes desktop`) — der scheitert an node-24-Drift + Electron-Download-SSL. Stattdessen **prebuilt `.exe`-Installer** probieren (Docs `…/windows-native`). Falls doch `hermes update` nötig: vorher `hermes backup` **und** den Auto-TTS-Self-Patch beachten (sonst geht er verloren — `Hermes/hermes_autotts_selffix_20260616.patch` re-applien).

**WICHTIG — bewusst NICHT in Hermes:** **Linear bleibt draußen** (Claude-Code-Build-Achse → `project_linear_build_axis`).

**Separat geparkt (NICHT Teil von Phase 5):**
- **Opus-4.8-Orchestrator-Profil** → erst nach SDK-Credit-Claim + Verifikation (Anthropic-OAuth = raue Ecke).
- **Buttcoin-Content-System „rund machen"** → eigener Design-Pass, startet mit Failure-Diagnose der Vorversionen → `project_buttcoin_content_system`.

Auto-Pickup-Regel: Kern kurz nennen + anbieten, **nicht ungefragt starten** ([[feedback_no_unsolicited_actions]]).

---
*Selbst-Reset: Nach Ausführung von Phase 5 diese Datei auf `# Pending Prompt — (leer)` zurücksetzen.*
