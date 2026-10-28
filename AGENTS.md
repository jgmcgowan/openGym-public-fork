# AGENTS.md

Handoff for any coding assistant working in this checkout (Codex, Copilot, Claude Code, Gemini, Cursor, and others). Read this file, then [HANDOFF.md](HANDOFF.md) for where the fork stands. [CLAUDE.md](CLAUDE.md) is the longer upstream code map; keep it, and extend it when the architecture changes. People-facing diagrams live in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## What this is

openGym is a self-hosted gym and body-weight tracker. Two containers (`api` and `web`) plus a `./data` folder the operator owns: no third-party account, no telemetry. Sign-in is passkeys (WebAuthn), with an optional password. The web app is an installable PWA. Capacitor apps for Android and iPhone (Apple Watch app and widgets included) work on their own or synced to a server.

This repository is James McGowan's fork: <https://github.com/jgmcgowan/openGym>, forked from <https://github.com/DuarteSantos8/openGym>. License is AGPL-3.0-or-later. The exercise stills and animations are licensed separately ([NOTICE.md](NOTICE.md)).

## Stack

- Frontend: React 19, React Router 7, Zustand 5, Vite 8. Tests use Vitest. No TypeScript.
- API: plain `node:http` in `api/server.js`. Dependencies are `@simplewebauthn/server`, `web-push`, and `undici`. The Claude Agent SDK is an optional dependency for the coach image target only.
- Web: multi-stage Docker build, nginx in front of the static app, `/api` proxied to the API so passkeys see one origin.
- Phone shells: Capacitor 8 (`frontend/android`, `frontend/ios`). The watch app, widgets, and animation-pack downloader live under `frontend/ios`.
- Optional MCP server in `mcp/` (`@modelcontextprotocol/sdk`, Zod). Stdio only. It is not part of the Docker image.
- Node 22. CI sets `NODE_VERSION: 22`. The Dockerfiles pin `node:22.23.4-alpine`.
- There is no ESLint, Prettier, or other formatter config.

## Repo layout

```
frontend/   React + Vite app (src/views, src/components, src/store, src/lib, src/sheets.jsx).
            android/ and ios/ are the Capacitor shells (docs/MOBILE.md).
api/        server.js plus small modules beside it. coach/ is the optional AI coach;
            coach/core/ is shared with the phone apps. openapi.yaml documents every route.
web/        Dockerfile and nginx.conf.template (serves the app, proxies /api).
catalogue/  Exercise database: exercises/<id>.json, i18n/, licensed 180 px media/.
mcp/        Optional stdio MCP server. Reads DATA_DIR. Its one write tool saves through the API.
scripts/    Build, CI, and release helpers (scripts/README.md).
website/    opengym.ch, plain HTML/CSS/JS, deployed separately from this fork.
helm/, kubernetes/, contrib/nix/   Other deploy shapes. Examples, not the live deployment.
docs/       Guides indexed in docs/README.md. docs/dev/ holds feature design notes.
```

Generated and committed, so do not edit them by hand: `frontend/src/lib/exercises-data.js`, `frontend/src/instr/`, `frontend/src/exercise-names/`, `frontend/src/exercise-desc/` (from `scripts/catalogue/build.mjs`). The API's coach catalogue assets and `website/api.html` are generated too (`scripts/build-coach-assets.mjs`, `scripts/build-api-docs.mjs`).

## Run locally

Docker with Compose is the supported stack. From the repo root:

```bash
cp .env.example .env
docker compose pull          # prebuilt images (skip this to build from source)
docker compose up -d         # http://localhost:8080
# or, to build this tree:
docker compose up -d --build
```

Frontend hot reload, proxying `/api` to `http://127.0.0.1:3000` (override with `API_TARGET`):

```bash
cd frontend && npm install && npm run dev
```

The dev server expects the API to be up (the compose stack, or `cd api && npm start`). Default API port is 3000. `DATA_DIR` defaults to `/data` inside the API process; compose mounts `./data` there. Passkeys on localhost work over HTTP. Any other hostname needs HTTPS and matching `RP_ID` and `ORIGIN`. Read [docs/SELF_HOSTING.md](docs/SELF_HOSTING.md) before changing auth, sessions, or notifications.

Exercise media in dev is served from `catalogue/media`. Set `MEDIA_TARGET` to proxy it elsewhere.

## Test, lint, and build

There is no linter. Match the surrounding code by hand. These are the checks CI runs (`.github/workflows/test.yml`). A docs-only change skips the test jobs via the path filter; `tests-ok` still reports.

```bash
# Frontend (Vitest)
cd frontend && npm test
cd frontend && npx vitest run src/lib/progression.test.js
cd frontend && npx vitest run -t "some test name"

# API (node:test) and MCP (Vitest)
cd api && npm test
cd mcp && npm test

# Checks besides the suites. Root `npm ci` once, for js-yaml.
cd frontend && node scripts/check-locales.mjs
cd frontend && node scripts/check-source-strings.mjs          # report only, exit 0
cd frontend && node scripts/check-source-strings.mjs --strict # exit 1 if a t() key is missing
node scripts/catalogue/build.mjs --check
node scripts/catalogue/validate.mjs
node scripts/catalogue/check-media.mjs
node scripts/build-coach-assets.mjs --check
node scripts/build-api-docs.mjs --check

# Builds
cd frontend && npm run build
cd frontend && npm run build:mobile   # VITE_MOBILE build, 180 px media, cap sync
```

CI also runs `npm run test:fatigue-probe` in `frontend/`, builds and boots both API image targets (`default` and `coach`) plus the web image, and checks that the published API image has no AI runtime. `cd mcp && npm run check:node-loadable` and `node api/scripts/check-core-loadable.mjs` confirm the servers load in plain Node. iOS compile is `.github/workflows/ios.yml` on a Mac runner. Signed APKs, the Play bundle, and the App Store IPA are built on the upstream release machine, never in CI ([docs/APP_STORES.md](docs/APP_STORES.md)).

## Conventions

- Dependency-light is a hard constraint. Frontend: React, React Router, Zustand, plus Capacitor plugins. API: three dependencies. A new dependency is a hard sell.
- Training logic (what you lift next, or reading a logged session back) is a pure helper in `frontend/src/lib` with a unit test beside it.
- Some rules exist twice on purpose, because the API image cannot import the frontend. Change both sides in the same PR: `frontend/src/lib/sync-merge.js` and `api/sync-stamps.js`, `frontend/src/lib/glyphs.js` and `api/coach/core/glyphs.js`, `frontend/src/lib/nudge.js` and `api/nudge.js`, `frontend/src/lib/queue.js` and `api/queue.js`. The phones import `api/coach/core/` from `frontend/src/lib/coach-local.js`.
- New UI strings go through `t()` and into every pack in `frontend/src/locales/`. English is the source language and has no file. pt-BR inherits from pt (`frontend/scripts/pt-br-inheritance-fingerprint.mjs`).
- User-facing text is plain and human, with no em dashes. Leave [CHANGELOG.md](CHANGELOG.md) alone; release notes are written at release time.
- Add a route by adding a key to the `routes` object in `api/server.js` and the same entry in `api/openapi.yaml`.
- State is one JSON document per profile with a server revision. A conflicting PUT returns 409; the client merges in `frontend/src/lib/sync-merge.js` and retries. Newest stamp per field wins.
- Click through UI changes in a browser before opening a PR. See [CONTRIBUTING.md](CONTRIBUTING.md) and `.github/pull_request_template.md`.
- First-time contributor workflows on the upstream repo wait for a maintainer to approve them. This fork has issues disabled. Track work in pull requests and in [HANDOFF.md](HANDOFF.md).

## Deployment

James runs a self-hosted copy of this fork. How and where it is deployed is kept outside this public repo, along with its proxy config and live `.env`. Do not invent those files or the values in them, and do not add server details here.

What the repo does specify:

- `docker-compose.yml` names `ghcr.io/duartesantos8/opengym-api:latest` and `ghcr.io/duartesantos8/opengym-web:latest` (upstream images, amd64 and arm64). The same tags are also published to `registry.gitlab.com/duartesantos8/opengym/{api,web}`. `docker compose up -d --build` builds from the local tree instead. The coach image is opt-in: `API_TARGET=coach docker compose up --build api`.
- `.github/workflows/docker-publish.yml` on this fork would push `ghcr.io/jgmcgowan/opengym-api` and `ghcr.io/jgmcgowan/opengym-web` (`edge` on `main`, and `latest` plus semver tags when a GitHub release is published). The compose file is not pointed at those names. Nothing in the repo confirms which tag the live deployment runs. This fork has no GitHub releases of its own.
- Updating a normal install, from [SECURITY.md](SECURITY.md), is `git pull && docker compose pull && docker compose up -d`. Only the latest upstream release is supported. There is no LTS branch.
- Passkeys bind to the exact `RP_ID` and need `ORIGIN` to be the HTTPS URL the browser uses. `.env.example` documents every variable. The generic Caddy snippet in [docs/SELF_HOSTING.md](docs/SELF_HOSTING.md) is a reverse proxy to `localhost:8080` (`WEB_PORT`). The live site's proxy config is not checked in.
- The API stores users, credentials, and sessions in `./data` (`DATA_DIR=/data` in the container): `db.json`, `state-<uid>.json`, `uploads/`, the session secret, and `vapid.json` (Web Push keys, generated on first run). Back up `data/`. `./coach-auth` is a provider credential cache kept outside `data/` so a `data/` tarball does not include a live refresh token.
- `helm/` and `kubernetes/` pin `ghcr.io/duartesantos8/opengym-{api,web}:1.4.3`. They are examples, not the live layout.
- `.github/workflows/mirror.yml` pushes `main` and `v*` tags to GitLab only when `github.repository` is `DuarteSantos8/openGym`. It does not run for this fork. Do not push or merge on the GitLab mirror.

App version in this tree is 1.4.3 (`frontend/package.json`, `api/package.json`, `api/openapi.yaml`, Helm `appVersion`). Upstream tagged `v1.4.3` on 2026-10-26. See [HANDOFF.md](HANDOFF.md) for how far this fork is from that upstream `main`.

## Safety

- Never commit secrets. That includes `.env`, anything under `data/` (session secret, passkeys, user state, `vapid.json`), `coach-auth/`, tokens, and provider API keys. `data/` is gitignored. Do not commit it.
- Keep changes on a branch and open a pull request. Do not push straight to `main`.
- Do not add or replace anything in `catalogue/media/`. Those files are licensed from Gym visual, 180 px at most, and `scripts/catalogue/check-media.mjs` enforces that in CI, the web image, and a self-hosted server. Never commit, upload, or serve a larger size, and never feed the media to an AI tool.
- Do not put passwords, tokens, or other secrets in commit messages, [HANDOFF.md](HANDOFF.md), or the AI Work Log.
- This repo is public. Do not add personal details about James, server details, or links to private documents.

## Shared AI log

After you finish work in this repo:

1. Append one row to James's AI Work Log sheet (he shares the link privately; never write it here). Columns: date (ET), assistant, project (`openGym`), repo / URL, what changed, status (`Done`, `In progress`, or `Blocked`), next step, PR / link, notes. Only append. Do not delete or rewrite other rows, and do not drop the Projects or Lists tabs. A Drive write that replaces the whole file wipes those tabs. If that is the only write you have, do not use it. Put the row in [HANDOFF.md](HANDOFF.md) for a person to paste.
2. Update [HANDOFF.md](HANDOFF.md) so the next assistant can see the current state, what just changed, what is still open, and the next step.
3. If you do not have access to the sheet, say so in the pull request and in HANDOFF.md. Do not invent a log entry.

Read James's `README - Start Here` handoff doc before the first edit. The GitHub repo stays the source of truth for code. The sheet is the shared journal.
