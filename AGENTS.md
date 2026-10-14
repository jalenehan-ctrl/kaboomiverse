# kaboomiverse — working notes

Dependency-free static site (plain HTML/CSS/JS). No package manager, no build
step, no database: `index.html` (Crybaby Kaboom canvas game) is the app home,
`splash.html` is the standalone 18+ access portal, and the two PNGs are artwork.

## Running it here

```
docker compose -f docker-compose.base44.yml up -d --build
```

One `web` service (`python:3.12-slim`) serves the bind-mounted repo root with
`python3 -m http.server 3000`. Nothing is baked into an image, so edits are live
on the next request.

Verification: `docker compose -f docker-compose.base44.yml ps` (service should be
`healthy`) and `curl -s -o /dev/null -w '%{http_code}\n' http://localhost:3000/`.

## Quirks

- **No hot reload.** `http.server` has no watcher, so after editing a file call
  `reload_preview` to refresh the preview; do not expect edits to appear on their own.
- **The game is canvas-only.** `index.html` renders everything to a 1280x720
  canvas scaled to the viewport, so there is no DOM to query for gameplay state.
  The game's top-level bindings (`gameState`, `score`, `sanity`, `tantrum`,
  `player`, `enemies`, `highScore`) are classic-script globals and are readable
  from devtools/eval by bare name (e.g. `gameState`) for checks.
- **Input.** `mousedown`/`touchstart` on the canvas drives the game (left 45% of
  the canvas = jump, right = whine, top-right = KABOOM once the tantrum meter is
  full); a `keydown` on `window` starts the game with Space/Enter and maps
  Space/W/ArrowUp = jump, Z/J/Enter = whine, X/K = kaboom.
- **Audio.** Web Audio is created lazily on first user gesture, so a preview
  browser with no interaction stays silent.
- `canvas.roundRect` is used throughout — it needs a current browser (Chrome 99+).
