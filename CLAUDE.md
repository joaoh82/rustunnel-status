# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This is the **public Upptime-powered status page** for rustunnel — `status.rustunnel.com`. It is not a service to build or run locally; it is a configuration repo whose state is driven by GitHub Actions and consumed by a Vercel-deployed Next.js status site.

## Parent project

**Parent project:** [rustunnel](/Users/joaoh82/projects/rustunnel/CLAUDE.md) — read for shared dev commands, env vars, and cross-service architecture.

### How I fit in

This repo monitors the live endpoints of every other rustunnel sub-project. Sibling services it pings:

| Sibling | What it is | Probed endpoint |
|---------|-----------|-----------------|
| `rustunnel-web/` (web) | Marketing site (Next.js, Vercel) | `https://rustunnel.com` |
| `rustunnel-web/` (platform-api) | Platform API (Rust/Axum, Hetzner) | `https://api.rustunnel.com/health` + `/health/db` |
| `rustunnel/` | Edge tunnel servers (Rust, EU/US/AP) | `https://{eu,us,ap}.edge.rustunnel.com:8443/api/status` |
| `rustunnel/` | Edge synthetic tunnels | `https://synthetic.{eu,us,ap}.edge.rustunnel.com` |

The bootstrap runbook (DNS, Vercel, Postgres token seed for the synthetic tunnels) lives in `rustunnel-private/docs/deployment/status-page-setup.md`.

## How this repo works

This is an **[Upptime](https://upptime.js.org/) instance** — there is no build/test/lint pipeline in the conventional sense. All work happens via GitHub Actions:

- **Source of truth** is `.upptimerc.yml`. Editing it triggers regeneration of the workflow files and downstream artifacts on the next CI run.
- `.github/workflows/*.yml` are **auto-generated** by Upptime's `update-template.yml` job (runs daily). Do not hand-edit them — your changes will be overwritten. Edit `.upptimerc.yml` instead.
- `api/<site>/*.json` and `graphs/<site>/*.png` are **generated artifacts** committed back by CI (response-time, uptime, badges). Never edit by hand.
- `history/<site>.yml` is the **per-check history** appended by the `Uptime CI` workflow every 5 minutes. Each commit on `master` from `upptime-bot` is one probe round.
- `README.md` has auto-generated regions delimited by `<!--start: status pages-->` / `<!--end: status pages-->` and `<!--live status-->` markers. Do not edit between those markers — the `Summary CI` job rewrites them.
- Incidents are tracked as **GitHub Issues** (see `.github/ISSUE_TEMPLATE/`); the status site renders open/closed issues as active/past incidents.
- Deployment: pushes to `master` → GitHub Actions builds the static Next.js site → Vercel serves `status.rustunnel.com` (CNAME set in `.upptimerc.yml` under `status-website.cname`).

### Common changes

- **Add/remove/rename a monitored endpoint** → edit the `sites:` list in `.upptimerc.yml` and commit. CI handles the rest (creates `api/<slug>/`, `history/<slug>.yml`, regenerates README).
- **Tune monitor cadence or workflow schedules** → edit `workflowSchedule:` in `.upptimerc.yml` (note the GitHub Actions free-tier minute budget — `uptime: */5 * * * *` is the floor).
- **Change status-page UI/branding** → edit the `status-website:` and `i18n:` blocks in `.upptimerc.yml`.
- **Force a probe round now** → trigger the `Uptime CI` workflow manually via `workflow_dispatch` (or `gh workflow run "Uptime CI"`).
- **Trigger a template update** → run the `Update Template CI` workflow; it pulls the latest Upptime template into the generated workflow files.

### Things to avoid

- Do not commit changes to `.github/workflows/*.yml`, `api/**`, `graphs/**`, or the auto-generated regions of `README.md` — they will be reverted by the next CI run.
- Do not bypass `[skip ci]` markers in upptime-bot commits — they prevent feedback loops.
- Synthetic tunnel hostnames are special: they need a real rustunnel client running with the seeded synthetic token (see `rustunnel-private/docs/deployment/status-page-setup.md`). A `0.00%` uptime on a synthetic-tunnel row usually means the seed didn't run, not that the edge is down.

## Knowledge Base

This project shares its knowledge base with its parent (rustunnel). Do **not** create a separate `Projects/rustunnel-status/` folder — entries about this child go in the parent's vault.

### Project-specific — `~/Documents/josh-obsidian-synced/Projects/rustunnel/`

- **Code (this child):** `/Users/joaoh82/projects/rustunnel/rustunnel-status`
- **Code (parent meta-repo):** `/Users/joaoh82/projects/rustunnel`
- **Context (read first):** `~/Documents/josh-obsidian-synced/Projects/rustunnel/context.md`
- **Notes (running journal):** `~/Documents/josh-obsidian-synced/Projects/rustunnel/notes.md`
- **Project wiki:** `~/Documents/josh-obsidian-synced/Projects/rustunnel/wiki/`

**How to use each:**

- `context.md` — stable background (product goals, stakeholders, domain). Read before starting non-trivial work. Update only when underlying facts change.
- `notes.md` — append-only dated journal. Add entries under `## YYYY-MM-DD` headings for decisions, blockers, TODOs, and incidents — anything worth preserving but not stable enough for `context.md`. Notes about *this child* still go here, in the parent's `notes.md`.
- `wiki/` — reference sub-docs (e.g. `Architecture.md`, `Local Dev Setup.md`, `Tech Services.md`). Create new files as topics emerge. Child-specific reference material can live here too — prefix the filename with the child name (e.g. `rustunnel-status — Setup.md`) when disambiguation helps.

**When to save:**

- New stable fact about the product/domain → update the parent's `context.md`.
- A decision, incident, or working note → append a dated entry to the parent's `notes.md`.
- Reusable reference material (setup steps, credential locations, architecture) → new/updated file in the parent's `wiki/`.

### Cross-project knowledge — `~/Documents/josh-obsidian-synced/vault/`

- **General wiki:** `~/Documents/josh-obsidian-synced/vault/wiki/` — start at `_master-index.md`, then drill into the relevant topic's `_index.md`.
- **Raw dumps:** `~/Documents/josh-obsidian-synced/vault/raw/` — drop unprocessed research here as `YYYY-MM-DD-{slug}.md`.

Read the general wiki when the question isn't specific to this project. Drop raw research or imported notes into `vault/raw/` so it's captured even before it's distilled.
