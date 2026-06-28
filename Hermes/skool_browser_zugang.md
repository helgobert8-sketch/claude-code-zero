# Hermes ↔ private Skool-Community: authentifizierter Browser-Zugang

**Stand:** 2026-06-27 · **Ziel:** Hermes soll Text-Posts aus einer privaten Skool-Community
(`https://www.skool.com/vibe-coding-academy`) lesen/zusammenfassen, in die nur *du* eingeloggt bist.

---

## Warum es nicht „einfach so" geht

Hermes' Standard-Browser (`config.yaml → browser.cloud_provider: local`) startet einen **eigenen,
headless Chromium mit Wegwerf-Profil pro Task**. Der teilt **keine Cookies** mit deinem echten Chrome
(Sicherheits-Isolation) → er landet auf der öffentlichen `LOG IN / JOIN $97/month`-Seite.
Die Desktop-GUI zeigt dieses Browser-Fenster nicht an (unsichtbarer Hintergrund-Prozess).

**Lösung:** Hermes per **CDP (Chrome DevTools Protocol)** an einen Chrome anhängen, in den **du dich
selbst** eingeloggt hast. Wichtig: Der Schutzkreis ist das **Browser-Profil**, nicht die Seite —
Hermes kann jede Seite ansteuern, in die dieses Profil eingeloggt ist. Darum gilt überall:
**im Debug-Profil nur das einloggen, was Hermes sehen darf (nur Skool).**

Wichtige Pfade auf diesem Rechner:
- Chrome:        `C:\Program Files\Google\Chrome\Application\chrome.exe`
- Debug-Profil:  `C:\Users\manyw\AppData\Local\hermes\chrome-debug`  (separates Profil, NICHT dein normales!)
- Hermes-Config: `C:\Users\manyw\AppData\Local\hermes\config.yaml`
- Vault:         `C:\Users\manyw\HermesBrain`
- Shell:         git-bash / MSYS (nicht PowerShell)

---

## Weg A — pro Sitzung, aus dem Terminal (empfohlen)

Attended, temporäre Verbindung; endet beim Trennen. **Funktioniert in der Desktop-GUI UND in der
Terminal-CLI** — Voraussetzung: lokales Gateway (dein Setup). NICHT verfügbar in WebUI/Telegram/Discord-
Chats (dort wird der Slash-Befehl nur als Text an den Agenten geschickt) und nicht bei einem Remote-
Gateway. (Verifiziert in `apps/desktop/src/app/session/hooks/use-prompt-actions.ts:1183-1252`, Gating auf
`mode === 'remote'`. Frühere „nur Terminal"-Aussage war zu pauschal — korrigiert dank Hermes' Source-Check.)

1. In **Hermes Desktop** (oder alternativ im Terminal via `hermes chat`) den Status prüfen:
   ```
   /browser status
   ```
2. Verbinden:
   ```
   /browser connect
   ```
   → Hermes startet ein **sichtbares** Chrome mit `--remote-debugging-port=9222` und Profil
   `…\hermes\chrome-debug` und hängt sich an.
3. **In diesem Fenster** zu `https://www.skool.com/vibe-coding-academy` gehen und dich **selbst einloggen**.
   (Cookie bleibt im chrome-debug-Profil erhalten — Login überlebt Neustarts.)
4. Verbindung prüfen:
   ```
   /browser status
   ```
5. Hermes den Skool-Prompt geben (siehe unten).
6. Fertig:
   ```
   /browser disconnect
   ```
   und/oder das Debug-Chrome schließen.

---

## Weg B — dauerhaft, auch in der GUI / für Crons

Stehende Verbindung in der Config verankert → gilt für **alle** Hermes-Oberflächen (GUI, Gateway
**und Crons** wie Morning-Brief/Scout), solange das Debug-Chrome läuft. Mehr Komfort, größerer Risiko-
Radius (unbeaufsichtigte Crons könnten den Browser mitbenutzen).

1. In `config.yaml` unter `browser:` ändern:
   ```yaml
   cdp_url: 'http://127.0.0.1:9222'     # vorher: ''
   ```
2. Debug-Chrome manuell starten (git-bash) und in diesem Fenster Skool **einmal** einloggen:
   ```
   "/c/Program Files/Google/Chrome/Application/chrome.exe" \
     --remote-debugging-port=9222 \
     --user-data-dir="/c/Users/manyw/AppData/Local/hermes/chrome-debug" \
     --no-first-run --no-default-browser-check &
   ```
3. Hermes / Gateway **neu starten** (Config wird beim Start gelesen).
4. Verifizieren: Hermes Skool öffnen lassen → Erfolg = Mitglieder-Feed, nicht `/about`-Login.
5. Zurücksetzen, wenn nicht mehr gewünscht: `cdp_url` wieder auf `''` + Debug-Chrome schließen.

> Hinweis: env-Var `BROWSER_CDP_URL` hat Vorrang vor `config.yaml → browser.cdp_url`, falls beides gesetzt ist.

---

## A vs B — der Unterschied

| | A — `/browser connect` | B — `cdp_url` in config.yaml |
|---|---|---|
| Erlaubnis-Umfang | Jede Seite, in die das Debug-Profil eingeloggt ist | identisch |
| Lebensdauer | Nur diese Terminal-Session; endet bei disconnect/Schließen | Dauerhaft, solange Debug-Chrome läuft |
| Oberflächen | Desktop-GUI + Terminal-CLI (lokales Gateway); nicht WebUI/Telegram/Discord, nicht Remote-Gateway | Alle: GUI, Gateway, **Crons** |
| Auslösung der Navigation | Immer durch dich (attended) | Durch dich — aber Crons könnten es auch nutzen |

In **beiden** Fällen geht Hermes nur zu Skool, wenn du es beauftragst. Eine technische Pro-Seiten-
Sperre gibt es über CDP nicht — wer wirklich inhalts-/seiten-scharf begrenzen will, nimmt den
manuellen Datei-Export (Skool-Posts kopieren → `.md` in `HermesBrain\`).

---

## Sicherheits-Disziplin (für A und B)

- **Niemals** Passwort, 2FA-Code, Cookies oder Session-Token an den Agenten geben — du loggst dich
  selbst im sichtbaren Fenster ein.
- Debug-Profil **minimal** halten: nur Skool einloggen. (Scope = Profil, nicht Seite.)
- `--user-data-dir` **nie** auf dein normales Chrome-Profil zeigen (sonst Zugriff auf alle Logins/Passwörter).
- Port `9222` ist localhost-gebunden, aber alles auf dem Rechner, das ihn erreicht, kann den Browser
  steuern, solange er läuft → nach Gebrauch schließen.
- Skool ist eine bezahlte private Community: sparsam lesen, nicht aggressiv crawlen (ToS).

---

## Prompt für Hermes (kopierfertig)

Erst Weg A oder B aufsetzen (Verbindung steht, du bist in Skool eingeloggt), dann diesen Prompt geben.
`<URL>` und Dateiname anpassen.

```
Hermes, wir lesen private Skool-Inhalte über einen Browser, in den ICH mich selbst eingeloggt habe
und den du per CDP fernsteuerst. Regeln:

- Du steuerst meinen bereits eingeloggten Chrome. Frag mich NIE nach Passwort, 2FA-Code, Cookies
  oder Session-Token und versuche NICHT, dich selbst einzuloggen.
- Wenn du auf einer Login- oder "JOIN $97/month"-Seite landest, ist die Verbindung nicht aktiv oder
  ich bin nicht eingeloggt — sag mir das sofort, statt einen Login zu versuchen.
- Besuche ausschließlich die Seiten, um die ich dich bitte. Kein Crawlen über die genannte
  Community hinaus, keine Massen-Abrufe; lies die Posts ruhig und sequenziell.

Aufgabe: Öffne <URL>, lies die Text-Posts und gib mir eine strukturierte Zusammenfassung
(je Post: Titel · Autor/Datum falls sichtbar · Kernaussage in 2-3 Sätzen). Wenn Inhalt hinter
"mehr anzeigen" / abgeschnitten ist, expandiere ihn vor dem Zusammenfassen. Speichere das Ergebnis
anschließend als Markdown unter C:\Users\manyw\HermesBrain\ (frag mich nach dem Dateinamen, falls unklar).
```

---

## Empfehlung in einem Satz

Gelegentlich / Null-Risiko → manueller Export. · Wiederholbar, attended → **Weg A**. ·
Bequem in der GUI / für Routinen → Weg B (mit Profil-Hygiene & Cron-Bewusstsein). · Meiden:
Real-Profil-CDP, Cookie-Import.
