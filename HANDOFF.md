# HANDOFF.md

Current state of the `jgmcgowan/openGym` fork. Facts below were taken from this checkout and from GitHub on 2026-10-28. The OVH VPS was not inspected.

How to work in the repo: [AGENTS.md](AGENTS.md). The longer code map is [CLAUDE.md](CLAUDE.md).

## Current state

- Fork of [DuarteSantos8/openGym](https://github.com/DuarteSantos8/openGym). This remote is <https://github.com/jgmcgowan/openGym>. Default branch is `main`.
- `main` is `e6a95c28a` (2026-10-27), "Roadmap: in step with the milestones (v1.4.4 programmes/tour/fixes, v1.4.5 Google Play/F-Droid/integrations, v1.4.6 Wear OS/Health Connect, v1.4.7 engine/circuits, then v1.4.8 to v1.4.12)".
- Compared with `DuarteSantos8/openGym` `main` on 2026-10-28: this fork is **behind by 2 commits** and ahead by 0. Both missing commits are README-only:
  - `5fdb229fe` (2026-10-27) README: openGym-styled Buy me a coffee and Ko-fi badges, the Trendshift daily and weekly ranks back at the top
  - `bb60bbe5e` (2026-10-27) README: a new top (hero with the app on three phones, light and dark, one row of badges, App Store, Android and Self-host buttons), Trendshift badges moved to Community
- App version in the tree is **1.4.3** (`frontend/package.json`, `api/package.json`, `api/openapi.yaml`, `helm/Chart.yaml`). Upstream release `v1.4.3` ("the Halloween update") was published 2026-10-26. This fork has no GitHub releases. [CHANGELOG.md](CHANGELOG.md) has no `## Unreleased` section; the top section is `v1.4.3 (2026-10-26)`.
- Issues are disabled on this fork. Handoff docs are in pull request [#1](https://github.com/jgmcgowan/openGym/pull/1).
- Live site, from the operator: <https://opengym.devop.work>, James's OVH VPS, prebuilt GHCR images behind Caddy, directory `/opt/apps/opengym`. `docker-compose.yml` still pulls `ghcr.io/duartesantos8/opengym-api:latest` and `ghcr.io/duartesantos8/opengym-web:latest`. The fork's publish workflow would tag `ghcr.io/jgmcgowan/opengym-{api,web}` instead. Which image the VPS is running is not recorded in the repo.
- Assistant entry points added in this change: [AGENTS.md](AGENTS.md), this file, and a pointer at the top of [CLAUDE.md](CLAUDE.md). The rest of `CLAUDE.md` is the upstream project map and was left in place.

## Recent changes

Upstream history on this `main`, newest first. Nothing here is a fork-only commit.

- **2026-10-27.** Roadmap rewritten to match the milestones through v1.4.12 (`e6a95c28a`, `22d7ef8a6`). Website short links `/go/ios`, `/go/android`, `/go/survey` for printed cards (`e1697a465`).
- **2026-10-26.** Release v1.4.3 marked shipped in the changelog, roadmap, and README (`5d61a94ca`). Screenshots updated (`922b586a6`). Website APK sizes for 1.4.3 (`7c8290ca2`). FAQ: the iPhone app is in the App Store, Google Play is next (`08eabf4f4`).
- **2026-10-25.** Version bump to 1.4.3 across API, Android `versionCode` 28, Helm chart 0.2.3, and the website (`00d009361`). Halloween theme, iPhone alternate app icons, Health Connect history reads, iOS 15.5 to 16.1 color fallbacks, and a real "I own no dumbbells" switch landed in the same release window. Pull request and bug report templates now name the iPhone, Watch, and Android apps.

v1.4.3 headlines, from [CHANGELOG.md](CHANGELOG.md) and [ROADMAP.md](ROADMAP.md): gym reminder, tiny muscle map, CSV export and GymMane import, routine-slot backups, machine weight stacks, Apple Health and Health Connect read-back, Apple Watch RPE / countdown / Water Lock, Coach questions, Swedish and Spanish (Spain), MCP `log_workout`, and the Halloween theme (it removes itself on 1 November 2026).

## Known open items

- The two upstream README commits above are not in this fork yet.
- [ROADMAP.md](ROADMAP.md) says **v1.4.4** (programmes, tour, and fixes) is in TestFlight upstream, aimed at about 2026-11-08. It lists work still going in before that release: rest after the last set (#853), a discarded workout that still reached Apple Health, the Coach shortlist (#829), music pausing on the rest sound, and two security fixes. None of that is a branch in this checkout.
- Later milestones in the same file, not started here: v1.4.5 Google Play, F-Droid, and integrations; v1.4.6 Wear OS and Health Connect; v1.4.7 progression engine and circuits.
- `.github/workflows/test.yml` runs `check-source-strings.mjs` as report-only because of a known backlog of demo strings. `--strict` fails the process when a `t()` key is missing from every pack.
- Live `.env`, Caddy config, and `/opt/apps/opengym` were not verified from this environment. Do not treat [AGENTS.md](AGENTS.md) deployment notes as a copy of the server.
- The AI Handoff folder has a leftover spreadsheet named "AI Work Log (append test, delete me)": <https://docs.google.com/spreadsheets/d/1lOXP3_VGUojiXegAIqhmvKdrbQEl5NNpRxY8O8HVDPs/edit>. It is a copy made to see whether a Drive write could append one row. That write replaced the copy and left a single tab. The live AI Work Log was not edited. Trash the copy in Drive.
- `check-source-strings` and the catalogue generators are easy to forget after a UI or exercise change. CI on a code PR will catch a stale generated file.

## AI Work Log row to append

The live sheet was not edited. Paste this as a new row on the AI Work Log tab only. Columns, in order:

| Date (ET) | Assistant | Project | Repo / URL | What changed | Status | Next step | PR / link | Notes |
|---|---|---|---|---|---|---|---|---|
| 2026-10-27 | Cursor | openGym | https://github.com/jgmcgowan/openGym | Added AGENTS.md and HANDOFF.md, and pointed CLAUDE.md at AGENTS.md. Docs only. | Done | Fast-forward the two upstream README commits if this fork should match parent main. | https://github.com/jgmcgowan/openGym/pull/1 | Sheet not edited from the agent. A full-file Drive replace drops the other tabs. |

```text
2026-10-27	Cursor	openGym	https://github.com/jgmcgowan/openGym	Added AGENTS.md and HANDOFF.md, and pointed CLAUDE.md at AGENTS.md. Docs only.	Done	Fast-forward the two upstream README commits if this fork should match parent main.	https://github.com/jgmcgowan/openGym/pull/1	Sheet not edited from the agent. A full-file Drive replace drops the other tabs.
```

## Next steps

1. Paste the row above into the [AI Work Log](https://docs.google.com/spreadsheets/d/1GTAvxzWC9WTI0OlMeLOr5r9XMrOpVxD9L9zvkg9A-R8/edit) tab. Leave the Projects and Lists tabs as they are.
2. If this fork should match upstream `main`, fast-forward the two README commits. They do not change the app. Confirm the diff before merging.
3. Leave v1.4.4 feature work on upstream unless someone asks for it here. The working plan is [ROADMAP.md](ROADMAP.md) and the upstream milestones.
4. A change to the live site means the VPS at `/opt/apps/opengym` (Caddy in front, prebuilt GHCR images). Get the real `.env` and image tags from the machine before changing them. Back up `data/` first.
5. After later work, update this file. Append the AI Work Log only when the write is a real row append. A replace of the whole file drops the other tabs, so put the row here instead.
