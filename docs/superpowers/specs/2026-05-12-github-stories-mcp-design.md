# Managing User Stories in GitHub via MCP from Claude Code

**Date:** 2026-05-12
**Author:** brainstorming session with Claude Code
**Status:** Approved for planning

## Context and goal

Set up a user-story management system that can be operated end-to-end from Claude Code, using only free tooling. The system must serve a single developer initially and scale to a small team (2-5 people) without migration.

**Decisions from brainstorm:**

- Platform: GitHub (Issues + Projects v2) — the user already lives in GitHub
- Hierarchy: 2 levels (Epic → Story) using native sub-issues
- MCP usage: bidirectional (create stories during brainstorm + read/update while implementing)

## Architecture

```
Claude Code ←→ GitHub MCP ←→ GitHub API ←→ Repo + Project v2
```

**Components:**

1. **GitHub repo** — code + issue templates
2. **GitHub Project v2** — board with views (Backlog, Sprint, Roadmap) and custom fields
3. **Issue templates** — `epic.yml` and `story.yml`, acceptance criteria in Given/When/Then
4. **Official GitHub MCP server** — installed in Claude Code, authenticated with a fine-grained PAT
5. **Label conventions** — `epic`, `story`, `bug`, `priority:*`, `status:*`

**Why GitHub vs alternatives (Linear, Notion):** avoids splitting the world between code and stories, the free tier has no issue cap, and it scales without migration when the team grows.

## Repo setup

- **Name:** to be defined before implementation (placeholder: `<repo-name>`)
- **Visibility:** private at start
- **Initialization:** README, `.gitignore`, LICENSE, `.github/ISSUE_TEMPLATE/` directory

## Project v2 setup

- **Default view:** Board (Kanban)
- **Additional views:**
  - Backlog (table, filters `status:ready`, sorted by priority)
  - Current Sprint (board, grouped by status)
  - Roadmap (timeline, optional)
- **Custom fields:**
  - `Priority` — single-select: high / med / low
  - `Story Points` — number, Fibonacci scale (1/2/3/5/8)
  - `Sprint` — iteration field, 2-week cycles (default; adjustable)
  - `Type` — single-select: epic / story / bug / spike
- **Sub-issues:** enabled (native GitHub feature, 2024+)

## Issue templates

### `.github/ISSUE_TEMPLATE/epic.yml`

```yaml
name: Epic
description: Large initiative grouping multiple stories
title: "[Epic] "
labels: ["epic"]
body:
  - type: textarea
    id: goal
    attributes:
      label: Goal
      description: What problem it solves, for whom, why it matters
    validations: { required: true }
  - type: textarea
    id: success
    attributes:
      label: Success criteria
      description: How we know the epic is complete (metrics or deliverables)
    validations: { required: true }
  - type: textarea
    id: stories
    attributes:
      label: Planned stories
      description: Initial list (will become sub-issues)
```

### `.github/ISSUE_TEMPLATE/story.yml`

```yaml
name: User Story
description: Actionable story, implementable within one sprint
title: "[Story] "
labels: ["story"]
body:
  - type: textarea
    id: story
    attributes:
      label: Story
      description: "As a [role], I want [action], so that [benefit]"
    validations: { required: true }
  - type: textarea
    id: ac
    attributes:
      label: Acceptance criteria
      description: |
        Given/When/Then format:
        - Given [context], When [action], Then [expected result]
    validations: { required: true }
  - type: textarea
    id: notes
    attributes:
      label: Technical notes
      description: Constraints, dependencies, decisions
  - type: input
    id: epic
    attributes:
      label: Parent epic (optional)
      description: "#number or epic URL"
```

**Why Given/When/Then:** structured, testable, and Claude can generate tests directly from the AC.

## MCP setup

```bash
# 1. Create a fine-grained PAT at github.com/settings/personal-access-tokens
#    Required permissions:
#      - Issues (Read & Write)
#      - Pull requests (Read & Write)
#      - Contents (Read & Write)
#      - Projects (Read & Write)
#      - Metadata (Read)

# 2. Add the MCP to Claude Code
claude mcp add github -- npx -y @modelcontextprotocol/server-github
# Set environment variable: GITHUB_PERSONAL_ACCESS_TOKEN=<your-pat>
```

## Workflows from Claude Code

### A) Create a story during brainstorm

```
User:   "Let's brainstorm a Google login feature"
Claude: [brainstorming conversation]
        [proposes story with AC]
        [creates issue via MCP using story.yml template]
        → returns issue link
```

### B) Implement an assigned story

```
User:   "Let's work on #42"
Claude: [reads issue #42 via MCP]
        [reads AC and technical notes]
        [implements]
        [updates status to in-progress when starting]
        [opens PR linked to the issue with "Closes #42"]
```

### C) Epic planning

```
User:   "I need an epic for a notification system"
Claude: [creates epic via MCP]
        [proposes 4-6 sub-issue stories]
        [after user approval, creates the sub-issues]
        [adds them to the Project Backlog column]
```

**Commit/PR convention:** `feat: <story title> (#42)`. The `Closes #N` link auto-closes the issue on merge.

## Post-setup validation

Manual test in order, no intervention beyond what is described:

1. **MCP responds:** ask Claude `list my repos`. Must return a list → MCP+PAT OK
2. **Templates load:** open `github.com/<user>/<repo>/issues/new/choose` → see "Epic" and "User Story"
3. **Create epic via MCP:** Claude creates a test epic. Verify correct label and Project fields in the UI
4. **Create story with sub-issue:** Claude creates a story under the epic. Verify parent/child relationship in the UI
5. **Close via PR:** branch + commit `Closes #N` + PR + merge. Issue closes automatically and moves to "Done"
6. **Cleanup:** delete test issues

**Success criterion:** all 6 steps complete without errors.

## Known risks

- **PAT expiration** (fine-grained max 1 year) → set a calendar reminder for rotation
- **GitHub API rate limit** — 5000 req/h with PAT. Sufficient for solo use; revisit when team grows
- **Sub-issues is a relatively new feature** — if UI/API changes, adjust templates and workflow

## Out of scope (YAGNI)

The following is explicitly out of scope and will be considered in future specs only if real need arises:

- CI/CD automation (GitHub Actions beyond defaults)
- External integrations (Slack, Discord, email)
- Metrics/velocity dashboards
- Multi-repo / monorepo
- Fine-grained Project roles and permissions (deferred until team is invited)

## Open decisions for the planning phase

- Real repo name
- Sprint length (1 vs 2 weeks)
- Story points vs t-shirt sizes (XS/S/M/L)
- Final repo visibility (stay private, or flip to public under some criterion)
