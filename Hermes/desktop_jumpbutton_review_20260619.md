# Desktop-Review: „Jump to latest prompt"-Button — Review + 2 Fixes (2026-06-19)

**Wer/Was:** Claude Code (Opus 4.8) hat den von Hermes mit Chris erarbeiteten lokalen
Diff im Hermes-Desktop-UI reviewt und zwei kosmetische Punkte direkt gefixt. Nichts
committet/gepusht — alles liegt im Working-Tree von
`C:\Users\manyw\AppData\Local\hermes\hermes-agent`.

## Worum ging es
Zweiter runder Floating-Button neben dem bestehenden „Scroll to bottom" (Pfeil runter):
ein **Pfeil-hoch**, der zum Anfang des **letzten echten Human-Prompts** (Start des
aktuellen User+Assistant-Turns) springt. Frontend-only, keine Architektur-Umbauten.

## Verdikt
**Safe.** Frontend-only, fasst **nichts** an Sessions/Config/`state.db`/Profile/Auth/
Memories/Vault. Funktional korrekt; zwei rein optische Schwächen gefunden und behoben.

## Geprüft und für sauber befunden
- **Kein Sprung auf synthetische/Hintergrund-Messages (Kern-Requirement):** Background-
  Process-Notifications haben zwar `role:'user'`, werden aber in `UserMessage` über
  `PROCESS_NOTIFICATION_RE` abgefangen und **ohne** `StickyHumanMessageContainer`
  gerendert → kein `data-human-message="true"`. Steer-/Slash-Notes sind `role:'system'`.
  Die Sprunglogik `turns.reverse().find(… data-human-message …)` überspringt nachlaufende
  Notifications korrekt und landet auf dem echten letzten Human-Prompt. Sauber gebaut.
- **Lange Sessions / „Show earlier messages":** robust — der jüngste Turn ist immer
  gerendert, Ziel immer im DOM; degenerierter Fall (nur Notification-Turn) ist ein
  harmloser No-op.
- **i18n:** vollständig (en/ja/zh/zh-hant + types.ts). `defineLocale` merged über `en`,
  daher ist das Nachreichen von `scrollToBottom` in ja/zh-hant eine Verbesserung.
- **State/A11y/Stick-to-bottom:** `resetThreadScroll` resettet das neue Atom mit;
  `stopScroll()` vor dem Smooth-Scroll verhindert das Zurückreißen ans Live-Tail; beide
  Buttons haben `aria-label` + `tabIndex`-Toggle.

## Gefundene Punkte + angewandte Fixes

### Punkt 1 — `hidden` killte die Exit-Animation (kosmetisch)
`display:none` (Tailwind `hidden`) wurde im selben Render wie `data-state='out'` gesetzt →
das `@keyframes thread-jump-out` lief nie. Die gesamte Exit-Maschinerie
(`*HasShownRef`, `'out'`-Zweig, Out-Keyframe) war damit toter Code, und der bestehende
Pfeil-runter-Button verschwand schlagartig statt herauszuanimieren.

**Fix (Option A — Totholz entfernt, Eingang bleibt animiert):**
- `apps/desktop/src/app/chat/scroll-to-bottom-button.tsx`: beide `hasShownRef`-Refs +
  ungenutzten `useRef`-Import entfernt; State ist jetzt `visible ? 'in' : 'idle'`;
  Doc-Kommentar korrigiert (Hide ist bewusst sofort, kein Exit-Anim — ehrlich
  dokumentiert).
- `apps/desktop/src/styles.css`: tote `.thread-jump-button[data-state='out']`-Regel und
  `@keyframes thread-jump-out` entfernt; `prefers-reduced-motion`-Block bereinigt;
  Kommentar angepasst.

*Entrance-Animation bleibt erhalten; Verschwinden ist instant (war es de facto schon).*

### Punkt 2 — Sprung-Offset falsch berechnet (kosmetisch, v. a. Neben-Fenster)
`parseFloat(getComputedStyle(viewport).getPropertyValue('--sticky-human-top'))` konnte den
Wert nicht lesen: im Hauptfenster `0.23rem` (Einheit fällt weg → ~3 px daneben,
unsichtbar), in Secondary-Windows ein `calc(...)` → `NaN → 0`, wodurch die oberste
Prompt-Zeile nach dem Sprung hinter die Titelleisten-Maske rutschte.

**Fix:**
- `apps/desktop/src/components/assistant-ui/thread-list.tsx`: Offset jetzt aus der
  px-aufgelösten `top` des Prompt-Elements gelesen:
  ```ts
  const stickyEl = target.querySelector<HTMLElement>('[data-human-message="true"]')
  const stickyGap = stickyEl ? parseFloat(getComputedStyle(stickyEl).top) || 0 : 0
  ```
  Korrekt im Haupt- **und** in Neben-Fenstern.

## Verifikation (nach den Fixes, im aktuellen Tree)
- `npm run test:ui -- src/app/chat/scroll-to-bottom-button.test.tsx` → **8/8 passed**
- `npm run typecheck` (`tsc -p . --noEmit`) → **sauber, keine Fehler**
- `npx eslint src/app/chat/scroll-to-bottom-button.tsx src/components/assistant-ui/thread-list.tsx` → **OK (exit 0)**

## Build-Sicherheit
`hermes desktop --force-build` baut laut `apps/desktop/electron/main.cjs` (resolveRendererIndex)
nur das Renderer-Bundle nach `dist/` neu — reiner Frontend-Build, fasst keine Nutzerdaten an.
**Unbedenklich.**

## Status / offene Schritte
- Änderungen liegen **nur im Working-Tree**, nicht committet (bewusst). Branch `main`
  ist 185 Commits hinter `origin/main` — vor einem Commit ggf. Rebase/Pull bedenken.
- Geänderte Dateien (insgesamt aus Feature + Review): scroll-to-bottom-button.tsx (+test),
  thread-list.tsx, thread.tsx, store/thread-scroll.ts, styles.css, i18n (en/ja/zh/zh-hant/types).
- Nächster Schritt nach Freigabe: `hermes desktop --force-build` + Sichtprüfung, dann
  Commit.
