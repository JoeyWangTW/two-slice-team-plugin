---
name: project-create
description: Create a new project with full template structure and register it in the Two Slice Team system.
argument-hint: "<project-name> [--path /custom/path]"
disable-model-invocation: true
---

Create a new project with full template structure and register it in the Two Slice Team system.

## Usage

```
/tst:project-create my-cool-app
/tst:project-create my-cool-app --path /Users/me/projects/my-cool-app
```

## Arguments

- `<project-name>` (required): The name of the project. Used to generate the project ID (kebab-case) and directory name.
- `--path` (optional): Custom directory path. Defaults to `~/Documents/<project-name>`.

## Instructions

When this skill is triggered, perform the following steps:

### 0. Load HQ Path

Read `~/.config/tst/config.json` and extract the `hq_path` value. The project registry (`state/projects.json`) is stored at this path.

If `config.json` doesn't exist, display: "HQ not configured. Run `/tst:setup` to initialize your HQ first."

### 1. Parse Arguments

Extract the project name from `$ARGUMENTS`. If `--path` is provided, use that as the project directory. Otherwise, default to `~/Documents/<project-name>`.

Convert the project name to kebab-case for the project ID (e.g., "My Cool App" becomes "my-cool-app").

### 2. Create Project Directory Structure

Create the following directories and files in the project path:

```
<project-path>/
├── CLAUDE.md
├── prd.json
├── progress.txt
├── .claude/
│   └── settings.json
├── docs/
│   ├── status.md
│   ├── worklog.md
│   ├── inbox.md
│   ├── roadmap.md
│   └── next-tasks.md
└── scripts/
    └── ralph/
        ├── ralph.sh
        ├── CLAUDE.md
        ├── prompt.md
        ├── prd.json -> ../../prd.json          (symlink)
        └── progress.txt -> ../../progress.txt  (symlink)
```

### 3. Install Ralph Loop

Fetch the Ralph Loop files from GitHub into `<project-path>/scripts/ralph/`:

```bash
curl -fsSL -o <project-path>/scripts/ralph/ralph.sh https://raw.githubusercontent.com/snarktank/ralph/main/ralph.sh
curl -fsSL -o <project-path>/scripts/ralph/CLAUDE.md https://raw.githubusercontent.com/snarktank/ralph/main/CLAUDE.md
curl -fsSL -o <project-path>/scripts/ralph/prompt.md https://raw.githubusercontent.com/snarktank/ralph/main/prompt.md
chmod +x <project-path>/scripts/ralph/ralph.sh
```

Then create symlinks so ralph.sh can find `prd.json` and `progress.txt` (it looks in its own directory):

```bash
ln -sf ../../prd.json <project-path>/scripts/ralph/prd.json
ln -sf ../../progress.txt <project-path>/scripts/ralph/progress.txt
```

If any download fails, warn the user but continue with project creation:
```
Warning: Could not download Ralph Loop files. You can install them manually later from https://github.com/snarktank/ralph
```

### 4. Generate CLAUDE.md

Create `CLAUDE.md` in the project root with this content (replace `{{PROJECT_NAME}}` with the actual project name):

```markdown
# {{PROJECT_NAME}}

## Vision

(Vision not yet defined — use `/tst:project-meeting` to set direction)

## Work Documentation

### Status Updates
- Read `docs/status.md` at the start of every session to understand current project state
- Update `docs/status.md` after completing work with what was done and what's next

### Work Logging
- Append entries to `docs/worklog.md` for every work session
- Format: `## YYYY-MM-DD - Brief description` followed by bullet points of changes

### Inbox
- Check `docs/inbox.md` at the start of every session for action items from co-founder discussions or standups
- Mark items as `[SEEN]` after reading them
- Address action items in your current work session

## Project Conventions

(No conventions defined yet)

## Allowed Commands

(No custom commands yet)
```

### 5. Generate docs/ Files

Create the following files in the `docs/` directory:

**docs/status.md:**
```markdown
# Project Status

**Last updated:** {{TODAY_DATE}}

**Current state:** Getting started

## Recently Completed

- Project created

## In Progress

- (nothing yet)

## Up Next

- Define vision and roadmap
- Set up initial project structure

## Blockers

- (none)
```

**docs/worklog.md:**
```markdown
# Work Log

## {{TODAY_DATE}} - Project created

- Initial project setup
- Created project directory and documentation structure
```

**docs/inbox.md:**
```markdown
# Inbox

Action items from co-founder discussions, standups, and meetings appear here.
Mark items as `[SEEN]` after reading them.

---

(no items yet)
```

**docs/roadmap.md:**
```markdown
# Roadmap

## Vision

(Vision not yet defined)

## Milestones

### Milestone 1: First Deliverable
- [ ] Define scope
- [ ] Complete initial version
- [ ] Get first feedback

## Current Focus

- Define vision and initial direction
```

**docs/next-tasks.md:**
```markdown
# Prioritized Tasks

## Next Up

- (no tasks yet - use `/tst:project-meeting` to plan tasks)
```

### 6. Generate Configuration Files

**prd.json** (empty PRD ready for stories):
```json
{
  "project": "{{PROJECT_NAME}}",
  "branchName": "",
  "description": "",
  "userStories": []
}
```

**progress.txt** (empty progress log):
```
## Codebase Patterns

---
```

**.claude/settings.json:**
```json
{
  "allowedCommands": []
}
```

### 7. Generate VP Name

Generate a VP name for this project. Use a creative, memorable name that relates to the project domain. Examples:
- For a search project: "VP Sierra Search"
- For a payments project: "VP Parker Payments"
- For a chat app: "VP Charlie Chat"
- For a bakery business: "VP Betty Bakes"
- For a research project: "VP Reese Research"

Format: "VP [First Name] [Domain Word]"

### 8. Register in projects.json

Read `{{HQ_PATH}}/state/projects.json` and add a new entry:

```json
{
  "id": "<kebab-case-name>",
  "name": "<project-name>",
  "path": "<absolute-project-path>",
  "status": "active",
  "vp_name": "<generated-vp-name>",
  "created_at": "<ISO-8601-date>",
  "vision": ""
}
```

Write the updated projects.json back to disk.

### 9. Confirm to User

After completing all steps, display a summary:

```
Project "<project-name>" created!

  Path: <project-path>
  ID: <project-id>
  VP: <vp-name>
  Status: active

Next steps:
  - Run /tst:project-meeting <project-id> to define vision and roadmap
  - Or start working and use /tst:standup to check in
```
