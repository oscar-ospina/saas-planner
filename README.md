# saas-planner

Planning workspace for the SaaS product. **No application code lives here** — this repo only holds specs, plans, issues, and the Project v2 board (`Saas Planner Roadmap`, project number `1`). Product code is implemented in sibling repositories under `~/code/saas/` (e.g. `web/`, `api/`), and their stories are tracked here.

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

## Active work

**[Epic #5 — Establish the design system foundation](https://github.com/oscar-ospina/saas-planner/issues/5)** is **in progress** (open since 2026-05-27). The sibling code repo it produces, **`@saas/ui`**, now exists and is public at **[oscar-ospina/saas-packages](https://github.com/oscar-ospina/saas-packages)** — Storybook deployed, **CI / Release / Pages all green**. (`Release` had been failing because the changesets *Version Packages* PR couldn't be created; fixed 2026-06-03 by enabling the `saas-packages` repo setting **"Allow GitHub Actions to create and approve pull requests"**. The open **PR #2 `chore: version packages`** bumps `@saas/ui` 0.1.0 → 0.1.1 on merge.)

Both blocking spikes are closed with ADRs:

- **Stack** ([ADR](docs/superpowers/specs/2026-05-27-ds-stack-decision.md), [spike #6](https://github.com/oscar-ospina/saas-planner/issues/6)) — Tailwind v4 + Radix + shadcn sources, bundled as `@saas/ui`. Visual regression via Playwright (no Figma Dev seat, so Code Connect is not in scope).
- **Token pipeline** ([ADR](docs/superpowers/specs/2026-05-27-ds-tokens-pipeline.md), [spike #7](https://github.com/oscar-ospina/saas-planner/issues/7), [PoC](docs/superpowers/plans/2026-05-27-ds-tokens-poc/)) — custom transformer that walks the Figma file tree and emits a W3C DTCG `tokens.json` + Tailwind v4 `theme.css`. PoC ran end-to-end against `UI-Exercise` (88 of 93 local styles extracted).

**Done** (epic #5 stories #6–#12, all merged): repo bootstrap with semver/changesets release, v0 token package, Storybook (a11y + interaction), the 10 primitives, an example consumer app, Playwright visual-regression baselines, the real-Figma **token parity** + **Button** component-parity pilot + fonts, and a **keyboard-navigation + axe WCAG 2.2 AA E2E gate** (`ui/tests/e2e/`, runs per PR) — which closes the epic's "keyboard nav not yet E2E-verified" criterion.

**Open — resume here next session:**

_Recently done (2026-06-03/04):_ fixed the red `Release` workflow (enabled the `saas-packages` *Allow GitHub Actions to create and approve pull requests* setting); filed + linked stories #14/#15 under epic #5; **published `@saas/ui@0.1.1` to npm** — public, clean-room `npm install` verified, GitHub Release cut, publish-on-merge now wired ([story #15](https://github.com/oscar-ospina/saas-planner/issues/15) closed). _(Token note: `NPM_TOKEN` must be a classic **Automation** token — a non-2FA-bypass token gets `E403` from `changeset publish`.)_

**Deferred — [story #13](https://github.com/oscar-ospina/saas-planner/issues/13) (dark mode):** the `UI-Exercise` Figma file has **no dark palette** (confirmed 2026-06-04; the earlier "exists at `_Swatch/Light and Dark`" note was an unverified assumption — the cached snapshot is light-only). The real blocker was never the 429 — the source doesn't exist. Dark mode moves to a **future theming epic** (matches epic #5's own Notes) and is **unlinked from #5**; revisit by either adding `Dark/*` swatches to Figma (extract faithfully) or deriving a dark theme in code (then rewrite the AC).

**One open item — blocked on Figma** (429 resets **~2026-06-05**):

1. **Component ↔ Figma parity for the other 9 primitives — [story #14](https://github.com/oscar-ospina/saas-planner/issues/14)** (only Button is piloted) — needs targeted `nodeId` Figma fetches once the rate limit clears.

Design source of truth: Figma file **`UI-Exercise`** (`figma.com/design/i4WmV5Gfk9uivVQXC5NY8j`), via the Framelink Figma MCP. ⚠️ **Figma is on a starter/Viewer tier with a punishing API budget** — use targeted `nodeId` fetches only (never whole-file / `depth` dumps) and commit each pull as a snapshot, or you'll trip a multi-hour 429.

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
