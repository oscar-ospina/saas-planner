# Working in this repo with Claude Code

This repo is the planning and tracking workspace for the `saas-planner` project. All issues, sub-issues, projects, PRs, and labels are managed on GitHub.

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

### Implement a story

```bash
gh issue view <number> --repo oscar-ospina/saas-planner
git checkout -b story/<short-slug>
# implement
git commit -m "feat: <story title> (#<number>)"
gh pr create --repo oscar-ospina/saas-planner --base main --body "Closes #<number>"
```

The PR auto-closes the issue on merge to `main`.
