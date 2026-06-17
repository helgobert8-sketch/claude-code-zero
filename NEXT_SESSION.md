# Pending Prompt — Hermes Desktop-App + Core-Update (geplante Migration)

**Kern:** Den am 2026-06-17 fehlgeschlagenen Desktop-Install als **bewusste, geplante Core-Update-Migration** nachholen. Phase 5 (Autonomie) ist sonst DONE — **Morning-Brief-Cron** (`6a0c0b7481fd`, tgl. 8:00) **und Scout-Cron** (`baa8c0508d27`, wöchentlich So 18:00) laufen + sind validiert. Hintergrund/Details: Memory `project_hermes_agent` (Bullets „Phase 5 Teil 2 — Scout" + „Parkplatz — Desktop-App-Retry UPDATE 2026-06-17" + „Codex-Rolle").

**Warum eigene Session (die Gotcha):** Der prebuilt `.exe`-Installer ist **KEIN benigner Install** — er fährt intern ein **volles `git pull --ff-only origin main`** (Core `45e2f4fdc` → `origin/main`, ~11,8k-Commit-Sprung) und stasht den Auto-TTS-Self-Patch; am 06-17 scheiterte das `git stash apply` (Konflikt in `gateway/platforms/base.py`) → Install brach ab. Sauber zurückgerollt auf `45e2f4fdc` + Patch byte-genau wiederhergestellt. Der Sprung muss **geplant + verifiziert** gefahren werden, nicht nebenbei.

**Aufgabe:**
1. **Vorgespräch + Entscheid:** Vorwärts (neuer Core + Desktop-GUI) ODER beim stabilen `45e2f4fdc` bleiben? Vorwärts nur, wenn Nutzen (Desktop-GUI / Cron-UI / Profiles) den Verifikations-Aufwand lohnt. Der Scout/Morning-Brief brauchen die Desktop-App NICHT.
2. **Falls vorwärts — Prep:** `hermes gateway stop` + `hermes backup` (sichert config/auth/cron/skills/sessions, **schließt `hermes-agent/` aus**) + frischen Core-Diff sichern (`git -C C:\Users\manyw\AppData\Local\hermes\hermes-agent diff > Hermes/tts_patch_backup_<datum>.patch`). Dann Installer (`Hermes-Setup.exe`) ODER `hermes update` fahren.
3. **TTS-Self-Patch-Schicksal klären:** Der Patch ist gegen den **alten** Gateway-Code (upstream hat `gateway/`-Dateien umgebaut, u.a. `send_message`-Tool entfernt). Prüfen, ob Auto-TTS auf dem neuen Core **schon ab Werk korrekt** ist (dann Patch fallen lassen) — sonst frisch neu ableiten. **NICHT den alten Patch blind erzwingen.**
4. **End-to-end neu verifizieren:** Voice (`/voice tts`, de-DE-Florian) · Morning-Brief- + Scout-Cron (`hermes cron list`) · Auth (`hermes auth list`: openai-codex / xai-oauth / copilot) · Gateway-Dienst läuft.
5. **NICHT** den Source-Build (`hermes desktop`) — scheitert an node-24/Electron-SSL. Nur prebuilt `.exe` bzw. `hermes update`.

**Recovery, falls's wieder klemmt:** `git -C C:\Users\manyw\AppData\Local\hermes\hermes-agent reset --hard 45e2f4fdc` + `git stash apply` bzw. `git apply Hermes/tts_patch_live_backup_20260617.patch`. `c2fa302e9` = `origin/main` (jederzeit wieder holbar; Vorwärtsweg bleibt offen).

**Separat geparkt (NICHT Teil dieser Session):**
- **Codex als Reviewer/Overflow wiren** — Auth ist bereits beidseitig bereit (Codex-App-Pro + Hermes `openai-codex`). Wiring = schlanke per-repo `AGENTS.md` → `CLAUDE.md`, erst bei konkretem Bau-Bedarf → `project_hermes_agent` (Codex-Bullet) + `project_linear_build_axis`.
- **Opus-4.8-Orchestrator-Profil** → via API-Key, nach Bedarf.
- **Scout-/Morning-Brief-Iteration** (v1.2-Härtungen) laufen nebenbei nach realen Läufen.

Auto-Pickup-Regel: Kern kurz nennen + anbieten, **nicht ungefragt starten** ([[feedback_no_unsolicited_actions]]).

---
*Selbst-Reset: Nach Ausführung diese Datei auf `# Pending Prompt — (leer)` zurücksetzen.*
