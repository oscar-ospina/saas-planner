# Working in this repo with Claude Code

This repo is the **planning workspace** for the SaaS product. It holds specs, plans, issues, and the Project v2 board — **no application code lives here**. Product code lives in sibling repositories under `~/code/saas/` (for example `web/`, `api/`), and their stories are tracked here through this repo's issues and Project.

If a task asks you to write product code, the work belongs in the relevant sibling repo, not here. Use this repo to read or update the corresponding story (Epic / User Story issue), and reference the issue number from the sibling repo's commits and PRs.

All issues, sub-issues, projects, PRs, and labels for planning are managed on GitHub.

## Current state (read before reasoning about the design system)

Active epic: **[#5 Establish the design system foundation](https://github.com/oscar-ospina/saas-planner/issues/5)**. Both blocking spikes (`#6` stack, `#7` token pipeline) are **closed with ADRs**. Read those before proposing changes — they answer the load-bearing questions and the decisions are not open for re-litigation in a fresh session:

- [`docs/superpowers/specs/2026-05-27-ds-stack-decision.md`](docs/superpowers/specs/2026-05-27-ds-stack-decision.md) — Tailwind v4 + Radix + shadcn sources, bundled as `@saas/ui`; consumer wiring via `@source` directive + `@import "@saas/ui/theme.css"`; shadcn-source copy strategy; Playwright visual regression replaces Code Connect.
- [`docs/superpowers/specs/2026-05-27-ds-tokens-pipeline.md`](docs/superpowers/specs/2026-05-27-ds-tokens-pipeline.md) — Custom transformer over the Figma file tree → W3C DTCG `tokens.json` + Tailwind v4 `theme.css`; DTCG → Tailwind v4 namespace mapping rules per category; Style Dictionary evaluated and deferred. Transformer source is inlined in the ADR (the executable ships with `packages/ui`, not here).

PoC outputs from the token spike (real end-to-end run): [`docs/superpowers/plans/2026-05-27-ds-tokens-poc/`](docs/superpowers/plans/2026-05-27-ds-tokens-poc/).

**Next planned story:** `Bootstrap packages/ui` (not yet opened — open it under epic #5 when the user is ready). It scaffolds the future sibling repo using both ADRs as input.

## Figma access (for design system work)

Design source of truth: **`UI-Exercise`** (key `i4WmV5Gfk9uivVQXC5NY8j`, [figma.com/design/i4WmV5Gfk9uivVQXC5NY8j](https://www.figma.com/design/i4WmV5Gfk9uivVQXC5NY8j)).

The **Framelink Figma MCP** is configured at user scope, so the tools `mcp__figma__get_figma_data` and `mcp__figma__download_figma_images` are available in any Claude Code session run from this user account (after a session restart if just installed).

API endpoint constraints (relevant if reasoning about Figma data sources):

- `/v1/files/:key` — works with `file_content:read`. The canonical path; used by the MCP under the hood. Returns the document tree plus `file.styles` for naming local styles.
- `/v1/files/:key/styles` — works with `library_content:read`. Returns only library-**published** styles. `UI-Exercise` has 0 published / 93 local, so this endpoint is empty for our case.
- `/v1/files/:key/variables/local` — requires **Enterprise plan** *and* `file_variables:read` scope. Double-blocked. Do not propose native-Variables-based pipelines unless the plan changes.

## GitHub tooling: use `gh` CLI

Use the `gh` CLI for all GitHub operations: creating issues, reading acceptance criteria, opening PRs, managing the Project v2 board, labels, releases, and admin tasks. The official GitHub MCP server was evaluated and not installed for this project — `gh` covers the workflow at lower operational cost.

For Projects v2 operations not exposed by high-level `gh project` commands, fall back to `gh api graphql` rather than introducing the MCP.

## Repository conventions

- Default branch: `main`
- Issue templates: `.github/ISSUE_TEMPLATE/{epic,story,config}.yml` (Epic and User Story; blank issues hidden for non-maintainers)
- Acceptance criteria format: Given/When/Then
- Hierarchy: Epic → Story via native GitHub sub-issues
- Story sizing: T-shirt sizes (XS / S / M / L) on the Project v2 `Size` field
- Sprint length: 2 weeks (`Sprint` iteration field on the Project)
- PR/commit convention: `feat: <story title> (#42)` with `Closes #N` in the PR body to auto-close the issue on merge

## Documentation locations

- Design specs: `docs/superpowers/specs/`
- Implementation plans: `docs/superpowers/plans/`

## GitHub Project

- Project: `Saas Planner Roadmap` (number `1`, owner `oscar-ospina`)
- URL: `https://github.com/users/oscar-ospina/projects/1`
- Custom fields: `Priority` (high/med/low), `Size` (XS/S/M/L), `Type` (epic/story/bug/spike), `Sprint` (iteration)

## Common workflows

### Create a story under an epic

```bash
gh issue create --repo oscar-ospina/saas-planner \
  --title "[Story] short summary" \
  --body "$(cat <<'EOF'
### Story
As a <role>, I want <action>, so that <benefit>.

### Acceptance criteria
- Given ..., When ..., Then ...

### Technical notes
...

### Parent epic
#<epic-number>
EOF
)" \
  --label story
```

Then link it as a sub-issue of the parent epic using GitHub's sub-issues UI or `gh api graphql`.

### Implement a story (work happens in a sibling repo)

```bash
gh issue view <number> --repo oscar-ospina/saas-planner   # read AC and notes
cd ~/code/saas/<sibling-repo>                              # switch to the code repo
git checkout -b story/<short-slug>
# implement
git commit -m "feat: <story title> (oscar-ospina/saas-planner#<number>)"
gh pr create --repo oscar-ospina/<sibling-repo> --base main \
  --body "Closes oscar-ospina/saas-planner#<number>"
```

Cross-repo `Closes oscar-ospina/saas-planner#<N>` in the sibling-repo PR body auto-closes the planning issue on merge.
