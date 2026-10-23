# TMX in the Tofu gallery

TMX (CourtHive's eXtensible Tournament Manager) is the desk software a racket-sports tournament runs on:
entries and participants, events and draws (elimination, round robin, tiered structures), the order of
play and the court grid, score entry, and printable draws, schedules and sign-in sheets. This copy runs
entirely in the visitor's browser. Tournaments are kept in the browser's own storage (IndexedDB, with
preferences in localStorage), so nothing is sent anywhere and no account is needed.

## How Tofu runs it

| | |
|---|---|
| Framework preset | `vite` |
| Package manager | pnpm 12.14.0 (the `packageManager` field), Node 24 (`engines.node` is `>=24`) |
| Install | `pnpm install --frozen-lockfile` (the committed `pnpm-lock.yaml`) |
| Build | `pnpm build` (runs `rimraf dist && tsc && vite build`) |
| Output directory | `dist` (entry page `dist/index.html`; routes are hash routes, so no rewrites are needed) |
| Environment variables | none required |
| Database | none: data stays in the visitor's browser |

Optional settings, not needed for the gallery copy:

- `SERVER`: the address of a CourtHive competition server. Sign-in, shared tournaments and publishing talk to it.
  Left empty, the app asks its own address, where no such server runs, and works locally.
- `VITE_SERVICE_WORKER`: set it to register the offline service worker. Left unset, the built `sw.js` is never
  registered.

## What works, and what needs a CourtHive server

Works in the browser alone, the way upstream's own online demo runs: creating tournaments (or generating demo
ones), participants (including import from a delimited file or a Google Sheet), events and draws, scheduling,
scoring and PDFs. A tournament survives a reload, but it lives in that one browser: clearing the site's data
removes it.

Needs CourtHive's own server, which Tofu does not run: signing in and registering, several people working on
one tournament, publishing to the public viewer, delegated or live scoring, chat, and languages other than
English (English is bundled; the others are fetched from the server). The account menu still offers
**Log in**; without a server it cannot succeed.

Requests to other sites happen only when a person asks for them: Google Fonts for some themes, map tiles
from OpenStreetMap or Esri when a venue map is shown, and Google Sheets when importing a sheet. None needs a
key. There are no analytics or advertising scripts.

## What we changed, and why

The upstream code, name, logo and favicon are unchanged. Every change is a commit on the `tofu` branch:

1. **`pnpm-workspace.yaml` and `pnpm-lock.yaml`.** Upstream overrides five CourtHive packages
   (`tods-competition-factory`, `courthive-components`, `pdf-factory`, `@courthive/provider-config`,
   `@courthive/scoring-visualizations`) with `link:../` paths, because its maintainers build TMX next to
   checkouts of those repositories. A copy of this repository alone has none of those folders, so it cannot
   install. We removed the five overrides, so the published versions `package.json` already names are
   installed, and regenerated the lockfile with pnpm 12.14.0. Upstream's own CI does the same at build time.
2. **`package.json`.** The `postinstall` script ran `playwright install chromium`, which downloads a test
   browser (about 186 MB) on every install, including a deploy that never runs the browser tests. It is now
   `pnpm test:install-browser`; run it before `pnpm test:e2e`.
3. **`TOFU.md`.** This file.

## Upstream and licence

- Upstream: <https://github.com/CourtHive/TMX>, branch `main`, commit
  `d1e5ffab641a4219a7f8e050f85e564266bd4ed8` (TMX 8.40.2).
- Licence: MIT, Copyright (c) 2024 Charles Allen. See `LICENSE.txt`, kept exactly as upstream ships it.
