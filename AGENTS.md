# Project notes for agents

## What this repo is
A single static page: `splash.html`, plus two PNG images in the repo root. There is
no build step, no package manager, no backend and no database. `splash.html` is the
whole app; `README.md` is notes only.

`docker-compose.base44.yml` serves the repo straight from the cloned source with
nginx (repo bind-mounted read-only into the web root, `.base44/nginx.conf` mounted
over `/etc/nginx/nginx.conf`). Editing any file is visible on the next page load —
nothing needs rebuilding or restarting.

## Non-obvious setup findings
- The sandbox clones the repo root as `0700 root-owned`. nginx's stock `user nginx;`
  therefore fails every `stat()` on the mounted source with `13: Permission denied`
  (surfaces as a 403 / rewrite cycle). `.base44/nginx.conf` sets `user root;` so the
  workers can read the mount. Re-apply that if the config is ever replaced.
- The app's only page is `splash.html`, so the nginx config uses `index splash.html;`
  plus a fallback — without it, `/` returns 403 because there is no `index.html`.
- No credentials, no external services and no third-party integrations are needed.
  `/run/base44/app.env` exists but this project requires nothing from it.

## How to run and verify
- Start: `docker compose -f docker-compose.base44.yml up -d` (web entry point on
  host port 3000).
- Verify: `curl -s -o /dev/null -w '%{http_code}' http://localhost:3000/` should be
  200, and the returned HTML should contain `WELCOME TO THE KABOOMIVERSE`. The
  cartoon `<img>` should resolve from the repo root (it is a relative path; it used
  to point at a GitHub `blob` URL, which served an HTML page and rendered broken).
- No tests, linters or migrations exist in this repo.
