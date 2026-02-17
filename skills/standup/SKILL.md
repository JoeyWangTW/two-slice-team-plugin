---
name: standup
description: Run a VP standup meeting across all active projects. Each project's VP reports status in parallel.
disable-model-invocation: true
---

Run a VP standup meeting across all active projects. Each active project's VP reports status in parallel.

## Prerequisites

Agent Teams must be enabled: `export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`

**Note:** Only one team can be active per session. If a co-founder team is active, it must be cleaned up before starting a standup.

## Usage

```
/tst:standup
```

## Instructions

When this skill is triggered, you are the **Lead Agent** coordinating the standup.

### 0. Load HQ Path

Read `~/.config/tst/config.json` and extract the `hq_path` value. All data files (state, standups) are stored at this path.

If `config.json` doesn't exist, display: "HQ not configured. Run `/tst:setup` to initialize your HQ first."

### 1. Load Active Projects

1. Read `{{HQ_PATH}}/state/projects.json`
2. Filter to projects where `status` is `"active"`
3. If no active projects, display: "No active projects. Use `/tst:project-create <name>` to create one."

### 2. Spawn VP Teammates

For each active project, spawn one agent team teammate with this prompt (fill in the project-specific details):

```
You are {{VP_NAME}}, the VP of Engineering for {{PROJECT_NAME}}.

## Your Role
You are a dedicated project manager and technical lead for this project. You know this project inside and out.

## Your Task: Standup Report
1. Change into the project directory: {{PROJECT_PATH}}
2. Read the following files to understand current state:
   - `CLAUDE.md` (project overview and conventions)
   - `docs/status.md` (current project status)
   - `docs/worklog.md` (recent work log entries)
   - `docs/inbox.md` (pending action items)
3. Prepare a standup report with:
   - **Done since last standup:** What was completed recently (from worklog.md and status.md)
   - **Planned next:** What's coming up (from status.md "Up Next" and "In Progress")
   - **Blockers:** Any blockers or issues (from status.md "Blockers")
   - **Inbox items:** Any unread inbox items that need attention
4. Message the lead with your standup report in this format:

## {{PROJECT_NAME}} — Standup Report
**VP:** {{VP_NAME}}

### Done
- <completed items>

### Next
- <planned items>

### Blockers
- <blockers or "None">

### Inbox
- <unread inbox items or "No new items">

## After Standup
If the user provides direction or feedback during the standup:
- Update `docs/status.md` to reflect any new priorities or direction
- Note any changes in your response to the lead
```

### 3. Synthesize Reports

After all VP teammates have reported:

1. Present a unified standup summary to the user with all VP reports
2. Highlight any blockers that need immediate attention
3. Note any cross-project dependencies or conflicts
4. Ask the user if they have direction or feedback for any projects

### 4. Handle User Feedback

If the user provides direction during standup:
- Relay the feedback to the relevant VP teammate(s)
- The VP should update their project's `docs/status.md` accordingly

### 5. Save Standup Summary

Save the standup summary to `{{HQ_PATH}}/standups/YYYY-MM-DD.md`:

```markdown
# Standup — YYYY-MM-DD

## Summary
- **Active projects:** <count>
- **Blockers:** <count of projects with blockers>

## Project Reports

### {{PROJECT_NAME_1}} ({{VP_NAME_1}})
**Done:** <summary>
**Next:** <summary>
**Blockers:** <summary>

### {{PROJECT_NAME_2}} ({{VP_NAME_2}})
**Done:** <summary>
**Next:** <summary>
**Blockers:** <summary>

## Direction Given
- <any direction or feedback from the user, or "None">

## Cross-Project Notes
- <any cross-project dependencies, conflicts, or synergies noted>
```
