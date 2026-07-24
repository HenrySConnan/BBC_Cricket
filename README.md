# BBC Cricket Scorer

A single-page, offline-first Progressive Web App for scoring backyard **BBC Cricket**
matches (6 overs a side, 5 legal balls an over, every player bowls exactly one over).
No backend, no login, no build step — just static files.

## Files

- `index.html` — the entire app (structure, styles, and logic in one file)
- `manifest.json` — PWA metadata so it can be "installed" to a home screen
- `sw.js` — service worker that caches the app shell for offline use
- `icon.svg` — app icon

## Run it locally

Just open `index.html` in a browser. For the "install to home screen" / offline
service-worker features to work, serve it over HTTP rather than `file://`:

```bash
npx serve .
# or
python3 -m http.server 8000
```

## Host it free on GitHub Pages

1. Push this folder to a GitHub repository.
2. In the repo, go to **Settings → Pages**.
3. Under "Build and deployment", choose **Deploy from a branch**, pick your
   default branch and the `/ (root)` folder.
4. Save. GitHub will give you a URL like
   `https://<your-username>.github.io/<repo-name>/`.
5. Open that URL on your phone and use **Share → Add to Home Screen**
   (iOS Safari) or the install icon in the address bar (Android Chrome) to
   get a full-screen app icon.

Once it's loaded once with a connection, it keeps working with no signal at all —
handy for a park with no reception.

## How it works (for future changes)

Everything lives inside the `<script>` tag in `index.html`, organised into
numbered sections:

1. **Constants & helpers** — over/ball math, formatting.
2. **State model & persistence** — the single `state` object that describes
   the whole match; auto-saved to `localStorage` after every ball.
3. **Navigation** — simple show/hide between `.screen` sections.
4. **Setup wizard** — the 6-step flow: names → squads → batting/bowling
   order → toss → confirm.
5. **Scoring engine** — pure functions that mutate `state.innings[n]`:
   `scoreRuns`, `scoreExtra`, `scoreWicket`, plus the shared
   `afterBallProgress` that checks for over/innings completion.
6. **Live screen rendering** — redraws the scoreboard, ball ticker, player
   cards, and button grids from `state` after every action.
7. **Innings break / match completion** — hand-off between innings 1 and 2,
   and final result detection.
8. **Match summary + awards** — scorecards, extras, over timeline, and the
   simple heuristics used for Player of the Match / top scorer / best
   bowler / most sixes.
9. **Export** — PDF via the browser's print dialog, native Share sheet
   (falls back to clipboard), and a hand-drawn PNG scorecard via Canvas.
10. **PWA bootstrap** — registers `sw.js`.

There's a full "Undo Last Ball" implemented via state snapshots
(`pushUndo` / `popUndo`), and tapping any ball chip in the ticker opens an
edit sheet that recalculates the innings from its ball-by-ball history.

## Notes on the BBC rules implemented

- 6 overs per innings, 5 legal balls per over, 30 legal balls total.
- Exactly 6 bowlers per team, each bowling one over, in the order set during
  match setup.
- Extra dismissal types beyond a normal "Wicket": **One Hand One Bounce**,
  **Over the Fence**, and **Electric Fence** — all recorded as separate
  dismissal reasons on the scorecard.
- Wides / no balls add one extra run and don't count as a legal ball; byes
  and leg byes add one run and **do** count as a legal ball. If more runs
  were actually run off an extra, tap the ball chip afterwards to edit it.
- Round-robin points (win = 1, loss = 0) are shown on the final result line.
