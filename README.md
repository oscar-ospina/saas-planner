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
