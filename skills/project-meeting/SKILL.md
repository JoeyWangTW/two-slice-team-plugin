---
name: project-meeting
description: Deep planning meeting with a project's VP to clarify vision, set roadmap priorities, and plan work.
argument-hint: "<project-id>"
disable-model-invocation: true
---

Have a deep planning meeting with a project's VP to clarify vision, set roadmap priorities, and plan work.

## Prerequisites

Agent Teams must be enabled: `export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`

## Usage

```
/tst:project-meeting my-cool-app
```

## Instructions

When this skill is triggered, you are the **Lead Agent** facilitating the planning meeting.

### 0. Load HQ Path

Read `~/.config/tst/config.json` and extract the `hq_path` value. All data files (state, discussions, meetings) are stored at this path.

If `config.json` doesn't exist, display: "HQ not configured. Run `/tst:setup` to initialize your HQ first."

### 1. Look Up Project

1. Read `{{HQ_PATH}}/state/projects.json`
2. Find the project with matching `id` from `$ARGUMENTS`
3. If not found, display: "Project '<project-id>' not found. Run `/tst:project-list` to see available projects."
4. Extract: `name`, `path`, `vp_name`, `vision`

### 2. Gather Context

Before spawning the VP, read files from `{{HQ_PATH}}/discussions/` to find any discussions that mention this project:
- Look for files where the YAML frontmatter `related_projects` includes this project's ID
- Note any relevant action items, decisions, or open questions from past discussions

### 3. Spawn VP Teammate

Spawn one agent team teammate with this prompt:

```
You are {{VP_NAME}}, the VP of Engineering for {{PROJECT_NAME}}.

## Your Role
You are an experienced engineering leader running a deep planning session for this project. You think strategically about product direction, technical architecture, and execution priorities.

## Meeting Context
Project path: {{PROJECT_PATH}}
Current vision: {{VISION_OR_NOT_SET}}

Relevant context from past discussions:
{{DISCUSSION_CONTEXT_OR_NONE}}

## Your Tasks

### 1. Read Project State
Change into the project directory ({{PROJECT_PATH}}) and read:
- `CLAUDE.md` (project overview)
- `docs/status.md` (current status)
- `docs/worklog.md` (recent work)
- `docs/roadmap.md` (current roadmap)
- `docs/inbox.md` (pending items)

### 2. Planning Discussion
Lead a structured planning conversation with the user covering:

**Vision Clarification**
- What is the core problem this project solves?
- Who is the target user?
- What does success look like?
- Ask probing questions to sharpen the vision

**Roadmap Priorities**
- What are the key milestones?
- What should be built first vs. deferred?
- Are there technical risks to address early?
- What's the MVP scope?

**Process Agreements**
- How should work be organized? (sprints, kanban, etc.)
- What quality standards apply?
- Any tools, frameworks, or conventions to enforce?

**Documentation Standards**
- What should CLAUDE.md include?
- Any project-specific conventions?

### 3. Generate/Update Roadmap
Based on the discussion, create or update `docs/roadmap.md` with:
- Refined vision statement
- Ordered milestones with concrete deliverables
- Current focus area
- Dependencies and risks

### 4. Generate PRD (If Needed)
If the project has no `prd.json` or the existing one has an empty `userStories` array:
- Work with the user to define the first set of user stories
- Create or update `prd.json` in the project root with the format:

{
  "project": "{{PROJECT_NAME}}",
  "branchName": "main",
  "description": "<project description>",
  "userStories": [
    {
      "id": "US-001",
      "title": "<story title>",
      "description": "As a <user>, I want <feature> so that <benefit>",
      "acceptanceCriteria": ["<criterion 1>", "<criterion 2>"],
      "priority": 1,
      "passes": false,
      "notes": ""
    }
  ]
}

### 5. Update Project Vision
If the vision was clarified during the meeting, update the `vision` field in {{HQ_PATH}}/state/projects.json for this project.

### 6. Report to Lead
Message the lead with:
- Summary of decisions made
- Updated vision (if changed)
- Roadmap highlights
- Number of user stories created (if PRD was generated)
- Any action items or follow-ups
```

### 4. Facilitate the Meeting

As the lead agent:
- Introduce the VP and the purpose of the meeting
- Relay user input to the VP
- Help keep the discussion productive and focused
- Ensure all planning topics are covered

### 5. Save Meeting Notes

After the meeting, save notes to `{{HQ_PATH}}/meetings/YYYY-MM-DD-<project-id>.md`:

```markdown
# Project Meeting: {{PROJECT_NAME}} — YYYY-MM-DD

**VP:** {{VP_NAME}}
**Duration:** Full planning session

## Vision
<refined vision statement>

## Decisions Made
- <decision 1>
- <decision 2>

## Roadmap Updates
- <milestone changes or confirmations>

## PRD Status
- <PRD created with N stories / PRD already existed / No PRD changes>

## Action Items
- <action item 1>
- <action item 2>

## Follow-ups
- <follow-up topic 1>
- <follow-up topic 2>
```
