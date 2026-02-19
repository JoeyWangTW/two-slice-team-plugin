---
name: project-status
description: Get a comprehensive status report from a project's VP. The VP reads all project docs and reports back.
argument-hint: "<project-id>"
disable-model-invocation: true
---

Spawn a project's VP to read all project documents and deliver a comprehensive status report.

## Prerequisites

Agent Teams must be enabled: `export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`

## Usage

```
/tst:project-status my-cool-app
```

## Instructions

When this skill is triggered, you are the **Lead Agent** requesting a status report.

### 0. Load HQ Path

Read `~/.config/tst/config.json` and extract the `hq_path` value.

If `config.json` doesn't exist, display: "HQ not configured. Run `/tst:setup` to initialize your HQ first."

### 1. Look Up Project

1. Read `{{HQ_PATH}}/state/projects.json`
2. Find the project with matching `id` from `$ARGUMENTS`
3. If no argument provided, display: "Please specify a project. Usage: `/tst:project-status <project-id>`"
4. If not found, display: "Project '<project-id>' not found. Run `/tst:project-list` to see available projects."
5. Extract: `name`, `path`, `vp_name`, `vision`

### 2. Spawn VP Teammate

Spawn one agent team teammate with this prompt:

```
You are {{VP_NAME}}, the VP of Operations for {{PROJECT_NAME}}.

## Your Role
You are the dedicated operational lead for this project. You know this project inside and out.

## Your Task: Comprehensive Status Report

1. Change into the project directory: {{PROJECT_PATH}}
2. Read ALL of the following files to build a complete picture:
   - `CLAUDE.md` (project overview and conventions)
   - `docs/status.md` (current project status)
   - `docs/worklog.md` (recent work log entries)
   - `docs/roadmap.md` (roadmap and milestones)
   - `docs/next-tasks.md` (prioritized task list)
   - `docs/inbox.md` (pending action items)
   - `prd.json` (if it exists — user stories and progress)
   - `progress.txt` (if it exists — codebase patterns and progress)
3. Prepare a comprehensive report and message the lead with:

## {{PROJECT_NAME}} — Status Report
**VP:** {{VP_NAME}}
**Vision:** {{VISION_OR_NOT_SET}}

### Current State
<high-level summary of where the project stands>

### Recently Completed
<what was done recently, from worklog and status>

### In Progress
<what's actively being worked on>

### Up Next
<prioritized upcoming work from next-tasks, roadmap, and PRD>

### Roadmap Progress
<milestone progress — how far along each milestone is, what's checked off vs remaining>

### Inbox
<any unread action items that need attention, or "No pending items">

### Blockers & Risks
<any blockers, risks, or concerns — or "None identified">

### Recommendations
<your assessment as VP — what should the CEO focus on, any suggestions for next steps>
```

### 3. Present Report

After the VP teammate reports back:

1. Present the VP's full report to the user
2. Ask if they have any questions or want to give direction

### 4. Handle Follow-Up

If the user provides direction or asks questions:
- Relay to the VP
- The VP should update project docs if the user requests changes (e.g., reprioritizing tasks, updating status)
