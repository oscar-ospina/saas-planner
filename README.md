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

_Recently done (2026-06-03):_ fixed the red `Release` workflow (enabled the `saas-packages` *Allow GitHub Actions to create and approve pull requests* setting → Release green, **Version Packages PR #2** `0.1.0 → 0.1.1` now open); filed + linked stories #14 and #15 under epic #5.

_Unblocked now (no Figma needed):_

1. **Publish to npm — [story #15](https://github.com/oscar-ospina/saas-planner/issues/15).** Gated on owning the `@saas` scope on npm (currently unclaimed, `npm view @saas/ui` → 404) **and** adding the `NPM_TOKEN` repo secret; the Release workflow is ready and token-gated — once the secret is set, merging the Version Packages PR publishes automatically. Independent of Figma.

_Blocked on Figma (429 resets ~2026-06-05):_

2. **Dark mode — [story #13](https://github.com/oscar-ospina/saas-planner/issues/13)** (ready, high priority). ⚠️ **BLOCKED** on the Figma API rate limit — a 429 with a ~55 h `Retry-After` hit 2026-06-02 (starter/Viewer tier), resetting **~2026-06-05**. The dark palette is **not cached** (verified: `ui/tokens/figma-all-palettes.yaml` holds only the light "All palettes" frame) and must be pulled fresh; the story's AC forbids eyeballing. **Decision: wait for the token pipeline — no hand-transcription.** When access returns: targeted `nodeId` fetch of the dark palette → commit a snapshot → extend `build-palette.mjs` → restructure `semantic.css` to `@theme inline` + `.dark{}` → dark contrast audit → dark Storybook + VR baselines.
3. **Component ↔ Figma parity for the other 9 primitives — [story #14](https://github.com/oscar-ospina/saas-planner/issues/14)** (only Button is piloted) — also needs Figma.

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
