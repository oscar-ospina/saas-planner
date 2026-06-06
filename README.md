# saas-planner

Planning workspace for the SaaS product. **No application code lives here** — this repo only holds specs, plans, issues, and the Project v2 board (`Saas Planner Roadmap`, project number `1`). Product code is implemented in sibling repositories — [`saas-packages`](https://github.com/oscar-ospina/saas-packages) (the `@saas/ui` design system) and [`alta-vibracion-web`](https://github.com/oscar-ospina/alta-vibracion-web) (the consumer app), plus a future `api/` — and their stories are tracked here.

## Getting started (new machine)

### Prerequisites

- `git` ≥ 2.30
- [`gh` CLI](https://cli.github.com/) ≥ **2.40** (older versions cannot manage Project v2; the official apt repo or `brew install gh` gives the latest)
- [Claude Code](https://docs.claude.com/claude-code) (optional but recommended — this repo is meant to be operated from inside it)

### One-time setup

```bash
# 1. Clone alongside any future sibling repos
mkdir -p ~/code/saas && cd ~/code/saas
git clone https://github.com/oscar-ospina/saas-planner.git

# 2. Authenticate gh with the Project v2 scope from the start
#    (omitting `project` scopes will require a refresh later)
gh auth login -s project,read:project

# 3. Verify access
cd saas-planner
gh repo view oscar-ospina/saas-planner --json visibility,url
gh project view 1 --owner oscar-ospina
```

### Daily entry points

```bash
# What's open
gh issue list --repo oscar-ospina/saas-planner

# Read a specific story
gh issue view <number> --repo oscar-ospina/saas-planner

# Create a new epic or story (opens the choose template flow in the browser)
gh issue create --repo oscar-ospina/saas-planner --web
```

When working from Claude Code, also see `CLAUDE.md` in this repo for the conventions Claude follows (gh-only, AC format, sub-issue linking, cross-repo `Closes` syntax).

## Status

- ✅ **Design system foundation — complete** (epic #5, 2026-06-04): [`@saas/ui@0.2.0`](https://www.npmjs.com/package/@saas/ui) published to npm.
- 🚧 **Consumer app MVP — in progress** (epic #16): the [Alta Vibración](#the-product) app repo is **live** at [`alta-vibracion-web`](https://github.com/oscar-ospina/alta-vibracion-web); 3 of 11 sub-issues done. See **[Active work](#active-work--alta-vibración-consumer-app-epic-16)** below.

### Design system foundation (epic #5) — complete ✅

**[Epic #5 — Establish the design system foundation](https://github.com/oscar-ospina/saas-planner/issues/5)** is **done & closed** (2026-06-04). The sibling code repo **`@saas/ui`** is **published to npm — [`@saas/ui@0.2.0`](https://www.npmjs.com/package/@saas/ui)** (public) at [oscar-ospina/saas-packages](https://github.com/oscar-ospina/saas-packages); Storybook deployed, CI / Release / Pages green. All 9 sub-issues (#6–#15) are closed: the two ADR spikes, the 10 primitives, token + component↔Figma parity, a WCAG 2.2 AA keyboard + axe E2E gate, an example consumer app, Playwright VR, and semver/changesets with publish-on-merge.

The product is now named — **[Alta Vibración](#the-product)** — but the design system stays **brand-agnostic / reusable** (Alta Vibración is the reference brand, not baked in).

Both blocking spikes are closed with ADRs:

- **Stack** ([ADR](docs/superpowers/specs/2026-05-27-ds-stack-decision.md), [spike #6](https://github.com/oscar-ospina/saas-planner/issues/6)) — Tailwind v4 + Radix + shadcn sources, bundled as `@saas/ui`. Visual regression via Playwright (no Figma Dev seat, so Code Connect is not in scope).
- **Token pipeline** ([ADR](docs/superpowers/specs/2026-05-27-ds-tokens-pipeline.md), [spike #7](https://github.com/oscar-ospina/saas-planner/issues/7), [PoC](docs/superpowers/plans/2026-05-27-ds-tokens-poc/)) — custom transformer that walks the Figma file tree and emits a W3C DTCG `tokens.json` + Tailwind v4 `theme.css`. PoC ran end-to-end against `UI-Exercise` (88 of 93 local styles extracted).

## Active work — Alta Vibración consumer app (epic #16)

The DS foundation is shipped; the product surface is now being built. **[Epic #16 — Alta Vibración marketing landing (MVP)](https://github.com/oscar-ospina/saas-planner/issues/16)** imports `@saas/ui` and adds the brand layer (logo, copy, imagery) on top of the brand-agnostic DS, starting with the **Home** landing that converts via **WhatsApp/contact**. Planned **in this gh planner** (Epic → Story), *not* via `/gsd:new-project` — the Issues/Projects board stays the system of record. The **Claude Design bundle** (see [Design sources](#design-sources)) ships clickable UI kits + the full brand guide to build from.

**Progress — 3 of 11 sub-issues done:**

- ✅ **[Spike #17 — framework](https://github.com/oscar-ospina/saas-planner/issues/17)** → **Next.js App Router** ([ADR](docs/superpowers/specs/2026-06-06-av-app-framework.md)); a Vite/SPA was rejected for the SEO-critical surface.
- ✅ **[Spike #18 — `@saas/ui` brand layer](https://github.com/oscar-ospina/saas-planner/issues/18)** → **thin layer** (the app inherits the DS theme; no neutral refactor yet) ([ADR](docs/superpowers/specs/2026-06-06-av-brand-layer.md)). Validated with an invented second-brand theming proof, which confirmed the DS re-skins via semantic-role overrides — and exposed that only **Button** reaches raw palette tokens (follow-up [#27](https://github.com/oscar-ospina/saas-planner/issues/27)).
- ✅ **[Story #19 — scaffold](https://github.com/oscar-ospina/saas-planner/issues/19)** → the consumer app is **live**: [oscar-ospina/alta-vibracion-web](https://github.com/oscar-ospina/alta-vibracion-web) (Next.js 16 + TypeScript 6 + Tailwind v4 + `@saas/ui@0.2.0`, CI green on `main`).
- ⬜ **Remaining:** [#20](https://github.com/oscar-ospina/saas-planner/issues/20) brand theme (logo lockups) · [#21](https://github.com/oscar-ospina/saas-planner/issues/21) app shell (routing/TopBar/Footer) · [#22](https://github.com/oscar-ospina/saas-planner/issues/22) SEO/a11y/responsive · [#23](https://github.com/oscar-ospina/saas-planner/issues/23) hero + WhatsApp · [#24](https://github.com/oscar-ospina/saas-planner/issues/24) trust sections · [#25](https://github.com/oscar-ospina/saas-planner/issues/25) consultations grid · [#26](https://github.com/oscar-ospina/saas-planner/issues/26) legal pages.

### Deferred & backlog

- **Booking & checkout epic** (follow-up to #16, not yet filed): the real in-app *Agenda* (availability backend, calendar, slot hold) and *Pago* (payment gateway for card/PSE/Nequi, server-verified webhooks, email/reminders, DIAN invoicing). This makes the app **full-stack** (activates a future `api/` sibling); the MVP routes booking intent to WhatsApp until it lands.
- **Future theming epic** (no product driver yet): dark mode ([#13](https://github.com/oscar-ospina/saas-planner/issues/13) — add `Dark/*` to Figma or derive in code), AA-safe status-badge tints, [#27](https://github.com/oscar-ospina/saas-planner/issues/27) (Button → semantic roles for full re-theme), component-level VR, a motion system, and data-display components (Table, Charts).

## The product

**Alta Vibración** — the online numerology practice of consultant **Liliana Tobón**. Spanish (Colombia): a marketing site where visitors learn numerology, book a consultation (*"Cita"*), pick a date/time, and pay (COP — card / PSE / Nequi). Tagline: *"No es casualidad. Es vibración."* Brand: cosmic & warm — orange `#f37d3e` + violet `#7f5af8`, Archivo + Open Sans, generous rounding. `@saas/ui` encodes the **brand-agnostic foundations**; product-specific surfaces live in the consumer app.

## Design sources

- **Figma — `UI-Exercise`** (`figma.com/design/i4WmV5Gfk9uivVQXC5NY8j`), via the Framelink Figma MCP. ⚠️ Starter/Viewer tier with a **harsh API rate limit** — targeted `nodeId` fetches only (never whole-file / `depth` dumps), commit each pull as a snapshot, or you trip a multi-hour 429. The file has **no dark palette** (confirmed) — don't plan dark-mode extraction against it.
- **Claude Design bundle** — the highest-leverage design source. A handoff generated from the same `.fig` (+ this DS repo) at [claude.ai/design](https://claude.ai/design): a brand guide, `colors_and_type.css` tokens, component **specimen cards** (`preview/comp-*.html`), and clickable UI kits (*Home → Agenda → Pago*). The #14 component-parity audit used these specimens — **no Figma API needed**. See [CLAUDE.md → Claude Design access](CLAUDE.md#claude-design-access) for how to fetch it.

## Conventions at a glance

- Hierarchy: **Epic → Story** via native GitHub sub-issues
- AC format: **Given / When / Then**
- Story sizing: **T-shirt** (XS / S / M / L) on the Project v2 `Size` field
- Sprint length: **2 weeks** (`Sprint` iteration field)
- Tooling: **`gh` CLI only** (the GitHub MCP server was evaluated and not adopted)

## Documentation

- Design specs: [`docs/superpowers/specs/`](docs/superpowers/specs/)
- Implementation plans: [`docs/superpowers/plans/`](docs/superpowers/plans/)
- Conventions for Claude Code: [`CLAUDE.md`](CLAUDE.md)

## Issue templates

Open a new issue at [`/issues/new/choose`](../../issues/new/choose) and pick **Epic** or **User Story**.
