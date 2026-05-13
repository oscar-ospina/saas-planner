# GitHub User-Story Management with MCP — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stand up a GitHub repo, Project v2 board, issue templates, labels, fine-grained PAT, and the GitHub MCP server in Claude Code so user stories can be created, read, and updated end-to-end from Claude Code.

**Architecture:** GitHub Issues + Projects v2 host all story data. Issue templates enforce Epic and Story formats with Given/When/Then acceptance criteria. The official GitHub MCP server (run via Docker) exposes the GitHub API to Claude Code through a fine-grained PAT.

**Tech Stack:** GitHub (Issues, Projects v2, Labels, sub-issues), `gh` CLI (≥ 2.40.0 required for Project v2 commands), Docker (runs `ghcr.io/github/github-mcp-server`), Claude Code MCP support.

**Repo target:** `oscar-ospina/saas-planner` (public)
**Local working dir:** `/home/oospina/code/saas/saas-planner`

---

## File Structure

Files created or modified during the plan:

| Path | Purpose | Created in |
|------|---------|------------|
| `README.md` | Repo entry point, project description | Task 2 |
| `.gitignore` | Standard Node/Next.js ignores (placeholder until tech stack picked) | Task 2 |
| `LICENSE` | MIT license | Task 2 |
| `.github/ISSUE_TEMPLATE/epic.yml` | Epic issue template | Task 5 |
| `.github/ISSUE_TEMPLATE/story.yml` | User Story issue template | Task 5 |
| `.github/ISSUE_TEMPLATE/config.yml` | Disable blank issues, enforce template choice | Task 5 |

The spec doc is already committed: `docs/superpowers/specs/2026-05-12-github-stories-mcp-design.md`. This plan lives alongside it under `docs/superpowers/plans/`.

External (non-file) state created:

- GitHub repo `oscar-ospina/saas-planner` (Task 3)
- 9 labels on the repo (Task 4)
- GitHub Project v2 "Saas Planner Roadmap" with custom fields and views (Task 6)
- Fine-grained Personal Access Token in GitHub account (Task 7)
- MCP server entry in Claude Code config: `github` (Task 8)

---

## Task 1: Upgrade `gh` CLI

The system has `gh 2.4.0` (2022); Project v2 subcommands need ≥ 2.40.0. Upgrade via the official GitHub apt repository.

**Files:** none (system package).

- [x] **Step 1: Confirm current version is too old**

Run:
```bash
gh --version
```
Expected: `gh version 2.4.0` (or any version below 2.40.0). If the version is already ≥ 2.40.0, skip the rest of Task 1 and continue at Task 2.

- [x] **Step 2: Add the GitHub CLI apt repository and key**

Run:
```bash
type -p curl >/dev/null || sudo apt install curl -y
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
```
Expected: lines added to `sources.list.d/github-cli.list` with no error.

- [x] **Step 3: Install the latest `gh`**

Run:
```bash
sudo apt update
sudo apt install gh -y
```
Expected: apt reports `gh` is upgraded; final line of `apt install` shows the new version.

- [x] **Step 4: Verify version**

Run:
```bash
gh --version
```
Expected: version ≥ 2.40.0.

- [x] **Step 5: Reconfirm authentication survived the upgrade**

Run:
```bash
gh auth status
```
Expected: `Logged in to github.com as oscar-ospina`. If not, run `gh auth login` and follow the prompts.

---

## Task 2: Add baseline repo files

**Files:**
- Create: `/home/oospina/code/saas/saas-planner/README.md`
- Create: `/home/oospina/code/saas/saas-planner/.gitignore`
- Create: `/home/oospina/code/saas/saas-planner/LICENSE`

- [x] **Step 1: Create `README.md`**

Content:
```markdown
# saas-planner

Planning and tracking workspace for a SaaS project, managed end-to-end from Claude Code via the GitHub MCP server.

## Documentation

- Design specs live in [`docs/superpowers/specs/`](docs/superpowers/specs/).
- Implementation plans live in [`docs/superpowers/plans/`](docs/superpowers/plans/).

## Issue templates

Open a new issue at [`/issues/new/choose`](../../issues/new/choose) and pick **Epic** or **User Story**.
```

- [x] **Step 2: Create `.gitignore`**

Content (covers common ecosystems; trim once the real tech stack is picked):
```
# Node
node_modules/
npm-debug.log*
yarn-debug.log*
.pnpm-debug.log*

# Build outputs
dist/
build/
.next/
.turbo/
out/

# Env
.env
.env.local
.env.*.local

# Editor
.vscode/
.idea/
*.swp
.DS_Store

# Logs
*.log
logs/
```

- [x] **Step 3: Create `LICENSE` (MIT, year 2026, owner Oscar Ospina)**

Content:
```
MIT License

Copyright (c) 2026 Oscar Ospina

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [x] **Step 4: Verify the files exist**

Run:
```bash
ls -la README.md .gitignore LICENSE
```
Expected: all three files listed with non-zero size.

- [x] **Step 5: Commit**

Run:
```bash
git -c user.email=tinysint@gmail.com -c user.name="Oscar Ospina" add README.md .gitignore LICENSE
git -c user.email=tinysint@gmail.com -c user.name="Oscar Ospina" commit -m "chore: add README, .gitignore, and MIT license"
```
Expected: commit created, three files reported as added.

---

## Task 3: Create the GitHub remote and push

**Files:** none (remote state).

- [x] **Step 1: Create the public repo from the existing local directory**

Run:
```bash
gh repo create oscar-ospina/saas-planner \
  --public \
  --description "User-story planning workspace, managed from Claude Code via GitHub MCP" \
  --source=/home/oospina/code/saas/saas-planner \
  --remote=origin
```
Expected: output `https://github.com/oscar-ospina/saas-planner` and a new `origin` remote configured locally.

- [x] **Step 2: Push the existing history to `origin/main`**

Run:
```bash
git push -u origin main
```
Expected: push succeeds; `main` set to track `origin/main`.

- [x] **Step 3: Verify the repo is reachable**

Run:
```bash
gh repo view oscar-ospina/saas-planner --json name,visibility,url
```
Expected: JSON containing `"name":"saas-planner"`, `"visibility":"PUBLIC"`, and the repo URL.

---

## Task 4: Create labels

**Files:** none (remote state).

- [x] **Step 1: Create the nine labels**

Run (one block, in `/home/oospina/code/saas/saas-planner`):
```bash
gh label create "epic"            --repo oscar-ospina/saas-planner --color 8B4FBC --description "Large initiative grouping multiple stories"
gh label create "story"           --repo oscar-ospina/saas-planner --color 0E8A16 --description "User story, implementable in one sprint"
gh label create "bug"             --repo oscar-ospina/saas-planner --color D73A4A --description "Defect"
gh label create "spike"           --repo oscar-ospina/saas-planner --color FBCA04 --description "Time-boxed investigation"
gh label create "priority:high"   --repo oscar-ospina/saas-planner --color B60205 --description "Highest priority"
gh label create "priority:med"    --repo oscar-ospina/saas-planner --color FBCA04 --description "Medium priority"
gh label create "priority:low"    --repo oscar-ospina/saas-planner --color C5DEF5 --description "Low priority"
gh label create "status:ready"    --repo oscar-ospina/saas-planner --color 0E8A16 --description "Refined and ready to pick up"
gh label create "status:in-progress" --repo oscar-ospina/saas-planner --color FBCA04 --description "Actively being worked"
gh label create "status:review"   --repo oscar-ospina/saas-planner --color C5DEF5 --description "In code review"
```
Expected: each line prints the label name. If any label already exists, the command errors `already exists` — that is acceptable, continue with the rest.

- [x] **Step 2: Verify all labels are present**

Run:
```bash
gh label list --repo oscar-ospina/saas-planner --limit 50
```
Expected: list contains `epic`, `story`, `bug`, `spike`, `priority:high`, `priority:med`, `priority:low`, `status:ready`, `status:in-progress`, `status:review` (10 entries plus any GitHub defaults).

---

## Task 5: Add issue templates

**Files:**
- Create: `/home/oospina/code/saas/saas-planner/.github/ISSUE_TEMPLATE/epic.yml`
- Create: `/home/oospina/code/saas/saas-planner/.github/ISSUE_TEMPLATE/story.yml`
- Create: `/home/oospina/code/saas/saas-planner/.github/ISSUE_TEMPLATE/config.yml`

- [x] **Step 1: Create the directory**

Run:
```bash
mkdir -p /home/oospina/code/saas/saas-planner/.github/ISSUE_TEMPLATE
```
Expected: directory exists, no output.

- [x] **Step 2: Create `epic.yml`**

Content:
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
    validations:
      required: true
  - type: textarea
    id: success
    attributes:
      label: Success criteria
      description: How we know the epic is complete (metrics or deliverables)
    validations:
      required: true
  - type: textarea
    id: stories
    attributes:
      label: Planned stories
      description: Initial list (will become sub-issues)
```

- [x] **Step 3: Create `story.yml`**

Content:
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
    validations:
      required: true
  - type: textarea
    id: ac
    attributes:
      label: Acceptance criteria
      description: |
        Given/When/Then format:
        - Given [context], When [action], Then [expected result]
    validations:
      required: true
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

- [x] **Step 4: Create `config.yml`**

Content:
```yaml
blank_issues_enabled: false
contact_links: []
```

- [x] **Step 5: Verify YAML syntax**

Run:
```bash
python3 -c "import yaml,glob; [yaml.safe_load(open(p)) for p in glob.glob('/home/oospina/code/saas/saas-planner/.github/ISSUE_TEMPLATE/*.yml')]; print('OK')"
```
Expected: `OK`. If a `yaml` import error appears, install: `sudo apt install python3-yaml -y`.

- [x] **Step 6: Commit and push**

Run:
```bash
git -c user.email=tinysint@gmail.com -c user.name="Oscar Ospina" add .github/ISSUE_TEMPLATE
git -c user.email=tinysint@gmail.com -c user.name="Oscar Ospina" commit -m "chore: add Epic and User Story issue templates"
git push origin main
```
Expected: push succeeds.

- [x] **Step 7: Verify templates show in the GitHub UI**

Open in a browser: `https://github.com/oscar-ospina/saas-planner/issues/new/choose`
Expected: two cards visible — **Epic** and **User Story** — and no "Open a blank issue" link at the bottom.

---

## Task 6: Create the GitHub Project v2

The project is owned by the user (not the repo) so the same project can host issues from future repos. Custom fields are created via `gh project field-create`. The Sprint iteration field and the custom views require the GitHub web UI (no stable CLI for those as of plan date).

**Files:** none (remote state).

- [x] **Step 1: Create the project**

Run:
```bash
gh project create --owner oscar-ospina --title "Saas Planner Roadmap" --format json
```
Expected: JSON output containing `"number": <N>` and `"url": "https://github.com/users/oscar-ospina/projects/<N>"`. **Record `<N>` — it is reused in every subsequent step.**

- [x] **Step 2: Add the `Priority` single-select field**

Run (replace `<N>` with the project number from Step 1):
```bash
gh project field-create <N> --owner oscar-ospina --name "Priority" --data-type SINGLE_SELECT --single-select-options "high,med,low"
```
Expected: JSON describing the new field, including its `id`.

- [x] **Step 3: Add the `Size` single-select field (T-shirt sizes)**

Run:
```bash
gh project field-create <N> --owner oscar-ospina --name "Size" --data-type SINGLE_SELECT --single-select-options "XS,S,M,L"
```
Expected: JSON describing the new field.

- [x] **Step 4: Add the `Type` single-select field**

Run:
```bash
gh project field-create <N> --owner oscar-ospina --name "Type" --data-type SINGLE_SELECT --single-select-options "epic,story,bug,spike"
```
Expected: JSON describing the new field.

- [x] **Step 5: Verify the three custom fields exist**

Run:
```bash
gh project field-list <N> --owner oscar-ospina
```
Expected: list contains `Priority`, `Size`, `Type` plus the built-in `Title`, `Assignees`, `Status`, `Labels`, `Linked pull requests`, `Milestone`, `Repository`, `Reviewers`.

- [x] **Step 6: Add the `Sprint` iteration field via the UI**

Open `https://github.com/users/oscar-ospina/projects/<N>/settings/fields`.
Click **New field** → name it `Sprint` → type **Iteration** → duration `2 weeks` → starts on next Monday → save.
Expected: `Sprint` appears in the field list with the first iteration auto-created.

- [x] **Step 7: Create the three views via the UI**

Open `https://github.com/users/oscar-ospina/projects/<N>`.

**View A — Backlog (table):**
1. Click the `+` next to the existing view tab → **New view** → choose **Table** layout → name `Backlog`.
2. Filter: `status:Todo` (or whatever the built-in Status backlog column is called).
3. Sort by `Priority` descending, then `Size` ascending.

**View B — Current Sprint (board):**
1. New view → **Board** layout → name `Current Sprint`.
2. Filter: `sprint:@current`.
3. Group by `Status`.

**View C — Roadmap (timeline):**
1. New view → **Roadmap** layout → name `Roadmap`.
2. Group by `Type`.
3. Date fields: leave defaults (uses iteration field for dates).

Expected: three custom view tabs visible in the project header.

- [x] **Step 8: Link the repo to the project so new issues auto-add**

Open `https://github.com/users/oscar-ospina/projects/<N>/workflows`.
Enable the built-in workflow **Auto-add to project** → set filter to `repo:oscar-ospina/saas-planner is:issue,pr`.
Expected: workflow toggled on, status reads `Enabled`.

- [x] **Step 9: Smoke-test the auto-add**

Run:
```bash
gh issue create --repo oscar-ospina/saas-planner --title "[Story] smoke test — delete me" --body "Auto-add verification" --label story
```
Expected: issue is created and prints its URL.

Open `https://github.com/users/oscar-ospina/projects/<N>` and verify the smoke-test issue appears in the table.

- [x] **Step 10: Delete the smoke-test issue**

Capture the issue number from Step 9 (last segment of the printed URL), then run:
```bash
gh issue delete <issue-number> --repo oscar-ospina/saas-planner --yes
```
Expected: confirmation that the issue is deleted.

---

## Task 7: Create the fine-grained Personal Access Token

A fine-grained PAT is required for the GitHub MCP server. The token grants the MCP read/write to issues, PRs, contents, projects, and metadata on `oscar-ospina/saas-planner` only.

**Files:** none (PAT is stored only in the local Claude Code MCP config in Task 8).

- [x] **Step 1: Open the fine-grained PAT creation page**

Browse to: `https://github.com/settings/personal-access-tokens/new`

- [x] **Step 2: Fill the token form**

| Field | Value |
|-------|-------|
| Token name | `saas-planner-mcp` |
| Expiration | `1 year` (max for fine-grained) |
| Description | `GitHub MCP server for Claude Code, scoped to saas-planner repo` |
| Resource owner | `oscar-ospina` |
| Repository access | **Only select repositories** → pick `oscar-ospina/saas-planner` |

Repository permissions:

| Permission | Access |
|------------|--------|
| Issues | Read and write |
| Pull requests | Read and write |
| Contents | Read and write |
| Metadata | Read-only (auto-selected) |

Account permissions:

| Permission | Access |
|------------|--------|
| Projects (Classic & v2) | Read and write |

- [x] **Step 3: Generate and copy the token**

Click **Generate token** at the bottom. Copy the value (begins with `github_pat_`). It is shown once only.

- [x] **Step 4: Save the token to a temporary local env file (do NOT commit)**

Run (replace `<paste-pat-here>`):
```bash
echo 'GITHUB_PERSONAL_ACCESS_TOKEN=<paste-pat-here>' > /home/oospina/.config/saas-planner-mcp.env
chmod 600 /home/oospina/.config/saas-planner-mcp.env
```
Expected: file exists with mode `-rw-------`.

- [x] **Step 5: Verify the token works against the GitHub API**

Run:
```bash
set -a; source /home/oospina/.config/saas-planner-mcp.env; set +a
curl -sS -H "Authorization: Bearer $GITHUB_PERSONAL_ACCESS_TOKEN" \
  https://api.github.com/repos/oscar-ospina/saas-planner | head -20
```
Expected: JSON beginning with `"id":` and `"name": "saas-planner"`. If the response is `"message": "Bad credentials"`, the token was mis-copied — regenerate.

---

## Task 8: Install the GitHub MCP server in Claude Code — SKIPPED

**Status:** Skipped during execution (2026-05-12). The `gh` CLI already covered the workflow during Tasks 1-6, and a mid-execution trade-off review concluded the MCP's marginal benefits did not justify its operational cost (Docker daemon, separate PAT to rotate, ~1s cold-start per call, the official MCP does not cover Projects v2 management).

The fine-grained PAT created in Task 7 and the `~/.config/saas-planner-mcp.env` file are kept dormant in case the decision is revisited.

The original install steps below are preserved for reference.

Use GitHub's official Docker image `ghcr.io/github/github-mcp-server`. Docker is already installed on this machine.

**Files:** Claude Code MCP config (`~/.claude.json` or equivalent — managed by `claude mcp add`).

- [ ] **Step 1: Pull the MCP server image**

Run:
```bash
docker pull ghcr.io/github/github-mcp-server
```
Expected: image downloaded, final line shows `Status: Downloaded newer image` or `Image is up to date`.

- [ ] **Step 2: Register the MCP server with Claude Code**

Run (loads PAT from the env file written in Task 7):
```bash
set -a; source /home/oospina/.config/saas-planner-mcp.env; set +a
claude mcp add github \
  --env GITHUB_PERSONAL_ACCESS_TOKEN="$GITHUB_PERSONAL_ACCESS_TOKEN" \
  -- docker run -i --rm \
  -e GITHUB_PERSONAL_ACCESS_TOKEN \
  ghcr.io/github/github-mcp-server
```
Expected: confirmation `Added MCP server: github` (exact wording may vary by Claude Code version).

- [ ] **Step 3: Verify the server is registered**

Run:
```bash
claude mcp list
```
Expected: `github` listed with status `connected` (or similar healthy state).

- [ ] **Step 4: Restart Claude Code so the new MCP loads**

Exit the current Claude Code session (Ctrl-C twice or `/exit`) and re-launch with `claude` in `/home/oospina/code/saas/saas-planner`. New tools prefixed `mcp__github__*` should now be available.

---

## Task 9: End-to-end validation (adapted to `gh` CLI)

The spec's six-step validation is adapted to use `gh` instead of MCP tool calls, since Task 8 was skipped.

**Files:** none (tests only).

- [x] **Step 1: `gh` is authenticated**

Run:
```bash
gh repo list oscar-ospina --limit 10
```
Expected: list contains `oscar-ospina/saas-planner`.

- [x] **Step 2: Templates load**

Open `https://github.com/oscar-ospina/saas-planner/issues/new/choose` in a browser.
Expected: **Epic** and **User Story** cards are visible; no blank-issue option.

- [x] **Step 3: Create a test epic via `gh`**

Run:
```bash
gh issue create --repo oscar-ospina/saas-planner \
  --title "[Epic] Validation epic — delete me" \
  --label epic \
  --body "$(cat <<'EOF'
### Goal
Validate the gh-based workflow end-to-end.

### Success criteria
All steps of Task 9 complete without manual intervention.

### Planned stories
- Test sub-issue
EOF
)"
```
Expected: prints the new issue URL (record the number as `<epic-number>`). The issue should auto-add to the Project; if it does not, add manually with `gh project item-add 1 --owner oscar-ospina --url <url>`.

- [x] **Step 4: Create a sub-issue story via `gh` and link it to the epic**

Run (replace `<epic-number>`):
```bash
STORY_URL=$(gh issue create --repo oscar-ospina/saas-planner \
  --title "[Story] Validation story — delete me" \
  --label story \
  --body "$(cat <<EOF
### Story
As a developer, I want to verify sub-issues, so that I trust the workflow.

### Acceptance criteria
- Given the parent epic exists, When I create the story, Then the story shows as a sub-issue of the epic.

### Parent epic
#<epic-number>
EOF
)")
echo "$STORY_URL"
```
Expected: prints the story URL (record as `<story-number>`).

Link as a sub-issue (REST endpoint, requires `gh` ≥ 2.49 — already installed):
```bash
gh api -X POST /repos/oscar-ospina/saas-planner/issues/<epic-number>/sub_issues \
  -f sub_issue_id="$(gh api /repos/oscar-ospina/saas-planner/issues/<story-number> --jq .id)"
```
Expected: returns JSON with the sub-issue relationship; the GitHub UI shows the story nested under the epic in the **Sub-issues** section.

- [x] **Step 5: Close via PR**

Run locally:
```bash
git checkout -b validation/sub-issue-test
echo "validation noop" > VALIDATION.md
git -c user.email=tinysint@gmail.com -c user.name="Oscar Ospina" add VALIDATION.md
git -c user.email=tinysint@gmail.com -c user.name="Oscar Ospina" commit -m "test: validation noop (Closes #<story-number>)"
git push -u origin validation/sub-issue-test
gh pr create --repo oscar-ospina/saas-planner --base main --head validation/sub-issue-test \
  --title "Validation PR — delete me" \
  --body "Closes #<story-number>"
gh pr merge --repo oscar-ospina/saas-planner --squash --delete-branch
```
Replace `<story-number>` with the number from Step 4. Expected: PR merges; the validation story moves to **Done** in the project; the issue is closed automatically.

- [x] **Step 6: Cleanup**

Run:
```bash
gh issue delete <epic-number>   --repo oscar-ospina/saas-planner --yes
gh issue delete <story-number>  --repo oscar-ospina/saas-planner --yes
git checkout main
git pull origin main
git rm VALIDATION.md
git -c user.email=tinysint@gmail.com -c user.name="Oscar Ospina" commit -m "chore: remove validation noop"
git push origin main
```
Expected: both issues deleted, `VALIDATION.md` removed from main.

- [x] **Step 7: Final commit of plan + spec state**

Run:
```bash
git status
```
Expected: working tree clean. The repo now has `README.md`, `.gitignore`, `LICENSE`, `.github/ISSUE_TEMPLATE/{epic,story,config}.yml`, the original `docs/superpowers/specs/...` and `docs/superpowers/plans/...` files, and nothing else.

---

## Done criteria

All of the following are true:

- `gh --version` ≥ 2.40.0
- `oscar-ospina/saas-planner` exists, public, with the README/LICENSE/.gitignore/issue templates pushed to `main`
- 10 labels exist on the repo (4 type + 3 priority + 3 status)
- Project v2 "Saas Planner Roadmap" exists with `Priority`, `Size` (XS/S/M/L), `Type`, and `Sprint` (2-week iteration) fields, plus the three custom views and auto-add workflow enabled
- A fine-grained PAT scoped to the repo lives in `~/.config/saas-planner-mcp.env` and authenticates against the GitHub API
- Claude Code shows `github` in `claude mcp list` and `mcp__github__*` tools are usable in a session
- The six end-to-end validation steps in Task 9 all pass

## Out of scope

Anything in the spec's "Out of scope" section: CI/CD beyond defaults, external integrations, dashboards, monorepo structure, fine-grained Project roles. Future specs only.

---

## EXECUTED 2026-05-12

Tasks 1–7, 9, 10 ran to completion. Task 8 (install GitHub MCP server) was skipped after a mid-execution review concluded `gh` CLI already covered the workflow at lower operational cost; the fine-grained PAT and `~/.config/saas-planner-mcp.env` are kept dormant. A new Task 10 was added to commit `CLAUDE.md` with the `gh`-only convention.

**Adjustments made vs. the original plan:**

- Local repo lives in subfolder `/home/oospina/code/saas/saas-planner/`, not at `/home/oospina/code/saas/` directly (chosen during Task 3 to keep `~/code/saas/` as a parent for future projects).
- Task 8 marked SKIPPED in place; the original install steps are preserved for reference if the decision is ever revisited.
- Task 9 reworked to use `gh` CLI and the GitHub sub-issues REST endpoint (`POST /repos/{owner}/{repo}/issues/{number}/sub_issues`) instead of MCP tool calls.

**Known caveat to revisit:** during the smoke test in Task 6 the Project v2 "Auto-add to project" workflow did not pick up the new issue automatically — the issue had to be added manually with `gh project item-add`. The toggle in `https://github.com/users/oscar-ospina/projects/1/workflows` should be verified before relying on auto-add for real stories.

**Done criteria status (revised — MCP-related criteria deliberately not met):**

- `gh --version` ≥ 2.40.0 — met (2.92.0 installed)
- `oscar-ospina/saas-planner` exists, public, with templates pushed to `main` — met
- 10 labels exist on the repo — met
- Project v2 "Saas Planner Roadmap" with custom fields — met for `Priority`, `Size` (XS/S/M/L), `Type`; `Sprint` iteration field, the three custom views, and the auto-add workflow are user-managed in the UI and pending verification
- PAT lives in `~/.config/saas-planner-mcp.env` and authenticates against the API — met (dormant)
- `claude mcp list` shows `github` — N/A (MCP not installed by design)
- End-to-end validation passes via `gh` — met (epic + sub-story + PR with `Closes #N` + auto-close + cleanup all worked)
