# PRD: Two Slice Team - Agent Orchestrator

## Introduction

A Claude Code plugin that acts as an operations team for orchestrating AI agents across multiple projects. The system models a startup org structure: two Co-Founder agents for ideation and research, and VP agents for project execution. The user (CEO) interacts with co-founders to discuss ideas, then VPs translate those into executed work via Ralph Loop.

Built on Claude Code's **Agent Teams** (experimental). Each co-founder and VP is a separate teammate with its own context window. They communicate via the agent teams messaging system, coordinate through a shared task list, and operate in parallel without blowing up a single context window.

The plugin is manual-first: all interactions are triggered via slash commands. Automated scheduling is planned for Phase 2 after the manual workflow is validated.

## Goals

- Provide a structured way to go from ideation to execution across multiple projects
- Enable async, autonomous project progress without constant user input
- Persist all discussions and decisions for context continuity across sessions
- Give each project a VP that proactively identifies and drives work
- Make project status instantly visible through standups and status reports
- Keep everything within the Claude Code plugin ecosystem (skills, hooks, CLAUDE.md)
- Maintain a clean communication channel between the management layer and each project via a standardized project template

## Architecture: Agent Teams

The orchestrator uses Claude Code Agent Teams as the runtime layer. This is critical — it means each agent gets its own context window, they message each other directly, and nothing overflows.

### Runtime Model

```
┌─────────────────────────────────────────────────────┐
│  TEAM LEAD (CEO session - your Claude Code)         │
│  - Spawns co-founders and VPs as teammates          │
│  - Coordinates standups, routes decisions           │
│  - Synthesizes reports                              │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────────┐  ┌──────────────┐                 │
│  │ Co-Founder 1 │←→│ Co-Founder 2 │  (can message   │
│  │ (Discussion) │  │ (Research)   │   each other)   │
│  └──────────────┘  └──────────────┘                 │
│                                                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │ VP Proj1 │ │ VP Proj2 │ │ VP Proj3 │  (isolated  │
│  │ cwd: /p1 │ │ cwd: /p2 │ │ cwd: /p3 │  contexts)  │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘            │
│       │             │             │                  │
│       ▼             ▼             ▼                  │
│  Ralph Loop    Ralph Loop    Ralph Loop              │
│  (execution)   (execution)   (execution)            │
└─────────────────────────────────────────────────────┘
```

### Why Agent Teams (Not Subagents)

| Concern | Subagents | Agent Teams |
|---|---|---|
| Context isolation | Own window, but results return to caller (bloats lead context) | Fully independent windows |
| Cross-agent communication | Cannot talk to each other | Teammates message directly |
| Co-founder debate | Would need to funnel through lead | Co-founders argue directly |
| VP standup with 5 projects | All 5 reports land in one context | Each VP reports independently, lead gets summaries |
| Token cost | Lower | Higher, but necessary for isolation |

### Agent Teams Setup

Requires enabling the experimental feature:

```json
// ~/.claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Recommended display mode: `tmux` split panes for visibility across all agents.

## User Stories

### US-001: Plugin scaffold and project registry
**Description:** As the user, I want a plugin structure with a central project registry so I can manage all projects from one place.

**Acceptance Criteria:**
- [ ] Plugin lives at `~/.claude/plugins/two-slice-team/` with valid plugin structure
- [ ] Central registry file at `~/.claude/plugins/two-slice-team/state/projects.json` tracks all projects
- [ ] Each project entry has: id, name, path, status (active/inactive), vp_name, created_at, vision summary
- [ ] Registry can be read and updated by plugin skills
- [ ] Plugin includes `plugin.json` with skill declarations

### US-002: Co-Founder Discussion session (Agent Team)
**Description:** As the user, I want to kick off a discussion with one or both co-founders as agent team teammates so that ideas are debated in isolated contexts without filling my main session.

**Acceptance Criteria:**
- [ ] `/cofounder` slash command spawns co-founder teammates via agent teams
- [ ] `/cofounder discuss` spawns only Co-Founder 1 (Discussion Partner) as a teammate
- [ ] `/cofounder research` spawns only Co-Founder 2 (Research/Serendipity) as a teammate
- [ ] `/cofounder both` spawns both as teammates — they can message each other directly to debate
- [ ] Co-Founder 1 spawn prompt includes: Socratic questioning, devil's advocate, structured decision-making persona
- [ ] Co-Founder 2 spawn prompt includes: web research directive, instruction to read past discussions from `discussions/`, lateral thinking persona
- [ ] User (lead) can message either co-founder directly via Shift+Up/Down
- [ ] Discussion is interactive — co-founders respond to each other and the lead

### US-003: Discussion documentation
**Description:** As the user, I want all co-founder discussions automatically saved so that decisions and context persist across sessions.

**Acceptance Criteria:**
- [ ] At end of a co-founder session, the lead (or a co-founder via instruction) writes a summary to `~/.claude/plugins/two-slice-team/discussions/YYYY-MM-DD-topic.md`
- [ ] File includes YAML frontmatter: date, participants, topics, related_projects
- [ ] Body includes: key points discussed, decisions made, action items (tagged with project IDs), open questions
- [ ] Co-Founder 2 (Research) spawn prompt includes instruction to read past discussion files for context
- [ ] Action items are extractable for routing to projects

### US-004: Project template and creation
**Description:** As the user, I want to spin up a new project with a standardized template so that the VP and Ralph Loop agents can work autonomously while maintaining communication with the management layer.

**Acceptance Criteria:**
- [ ] `/project create <name> [--path /custom/path]` command creates a new project
- [ ] Creates project directory at the specified path (or defaults to `~/Documents/<name>`)
- [ ] Bootstraps the full project template (see Project Template section below)
- [ ] Adds project to central registry (`state/projects.json`)
- [ ] VP name is assigned (can be auto-generated or user-specified)
- [ ] Project is immediately ready for a VP teammate to be spawned against it

### US-005: VP standup meeting (Agent Team)
**Description:** As the user, I want to run a standup where each VP is spawned as a separate teammate, reads its own project's state, and reports back — all in parallel without overloading context.

**Acceptance Criteria:**
- [ ] `/standup` slash command reads `state/projects.json`, filters to active projects
- [ ] Spawns one teammate per active project, each with a spawn prompt that includes: project path, VP persona, instruction to read `docs/status.md`, `docs/worklog.md`, and `docs/inbox.md`
- [ ] Each VP teammate CDs into its project directory as its working directory
- [ ] Each VP messages the lead with: what was done since last standup, what's planned next, blockers
- [ ] Lead synthesizes all VP reports and presents a unified standup summary to the user
- [ ] User can message individual VPs directly via Shift+Up/Down for follow-up
- [ ] VP updates `docs/status.md` with any new direction received
- [ ] Standup summary saved to `~/.claude/plugins/two-slice-team/standups/YYYY-MM-DD.md`

### US-006: VP full project meeting (first meeting / planning)
**Description:** As the user, I want a deeper meeting with a specific VP to clarify vision, roadmap, and process for their project.

**Acceptance Criteria:**
- [ ] `/project meeting <project-id>` spawns a single VP teammate for that project
- [ ] VP spawn prompt includes: full project context, instruction to read CLAUDE.md, status, worklog, related discussions from `discussions/` that mention this project
- [ ] Meeting covers: vision clarification, roadmap priorities, process agreements, documentation standards
- [ ] VP generates/updates `docs/roadmap.md` in the project folder
- [ ] VP generates initial work items in `prd.json` format if the project is new (Ralph-compatible)
- [ ] Meeting notes saved to `~/.claude/plugins/two-slice-team/meetings/YYYY-MM-DD-<project>.md`

### US-007: VP kicks off Ralph Loop for execution
**Description:** As the user, I want a VP to kick off autonomous work on their project using Ralph Loop so that progress happens without my constant involvement.

**Acceptance Criteria:**
- [ ] `/project work <project-id>` triggers the VP to prepare and start a Ralph Loop
- [ ] VP reads `docs/roadmap.md` and `docs/next-tasks.md` to determine current work
- [ ] VP ensures a valid `prd.json` exists in the project root with incomplete stories
- [ ] VP ensures `progress.txt` exists (Ralph's memory across iterations)
- [ ] VP ensures `scripts/ralph/ralph.sh` is present in the project (copies from ralph plugin if needed)
- [ ] VP kicks off Ralph via: `./scripts/ralph/ralph.sh --tool claude [max_iterations]`
- [ ] VP logs the work session start in `docs/worklog.md`
- [ ] Ralph runs in the project directory, making git commits, updating `prd.json` and `progress.txt` as it works

### US-008: Proactive VP work identification
**Description:** As the user, I want VPs to proactively identify work to do even when I haven't given specific tasks, so projects keep moving.

**Acceptance Criteria:**
- [ ] When a VP has no specific tasks from the user, it reads `docs/roadmap.md` and `docs/status.md`
- [ ] VP breaks down vague roadmap items into concrete user stories in `prd.json` format
- [ ] VP creates/updates `docs/next-tasks.md` with prioritized work items
- [ ] During standups, VP presents self-identified work for user approval before execution
- [ ] VP reads `docs/inbox.md` for action items routed from co-founder discussions
- [ ] VP can reference discussion docs to find relevant ideas that haven't been actioned

### US-009: Project status and listing
**Description:** As the user, I want to see all my projects and their status at a glance.

**Acceptance Criteria:**
- [ ] `/project list` reads `state/projects.json` and shows: name, status (active/inactive), VP name, last activity date, one-line status summary
- [ ] `/project status <project-id>` reads that project's `docs/status.md` and shows detailed status
- [ ] `/project pause <project-id>` sets a project to inactive in `state/projects.json`
- [ ] `/project resume <project-id>` sets a project back to active
- [ ] Inactive projects are skipped during `/standup`

### US-010: Route discussion outcomes to projects
**Description:** As the user, I want discussion outcomes to flow into relevant projects so that VPs have context from co-founder sessions.

**Acceptance Criteria:**
- [ ] At end of a co-founder discussion, action items tagged with project IDs are extracted
- [ ] Tagged items are appended to the relevant project's `docs/inbox.md` with date and source discussion reference
- [ ] During standup, VP checks `docs/inbox.md` for new items from discussions
- [ ] VP acknowledges and incorporates inbox items into their planning
- [ ] Processed inbox items are marked as acknowledged (not deleted — maintains audit trail)

## Project Template

Every project created by `/project create` gets this standardized structure. This template is the **communication protocol** between the management layer and the project. It must be consistent across all projects so that VPs, Ralph Loop, and the management team can interoperate.

```
<project-root>/
├── CLAUDE.md                    # Project identity + agent instructions
├── prd.json                     # Ralph-compatible task list (stories with passes: true/false)
├── progress.txt                 # Ralph's append-only memory across iterations
├── scripts/
│   └── ralph/
│       └── ralph.sh             # Ralph loop script (copied from ralph plugin)
├── docs/
│   ├── status.md                # Current project status (read by VP at standup)
│   ├── worklog.md               # Append-only log of work sessions
│   ├── roadmap.md               # High-level vision and milestones
│   ├── inbox.md                 # Action items routed from co-founder discussions
│   └── next-tasks.md            # VP-identified upcoming work
└── .claude/
    ├── settings.json            # Allowed commands for this project
    └── skills/                  # Project-specific skills (if any)
```

### CLAUDE.md Template

The generated `CLAUDE.md` is the most critical file. It tells every agent (VP, Ralph Loop iteration, subagent) how to behave in this project:

```markdown
# <Project Name>

## Vision
<one-paragraph vision statement provided during project creation>

## Work Documentation

IMPORTANT: Every agent working in this project MUST follow these conventions.

### Status Updates (docs/status.md)
After completing meaningful work, update docs/status.md with:
- Current state of the project (1-2 sentences)
- What was just completed
- What's next
- Any blockers

### Work Logging (docs/worklog.md)
Append an entry after each work session:
```
## YYYY-MM-DD HH:MM - <brief description>
- What was done
- Files changed
- Decisions made
- Issues encountered
```

### Inbox (docs/inbox.md)
Check docs/inbox.md at the start of each session for action items
from the management team. Acknowledge items by marking them [SEEN].

## Project Conventions
<language, framework, testing, and style conventions — filled in during /project meeting>

## Allowed Commands
<populated from .claude/settings.json>
```

### docs/status.md Template

```markdown
# Project Status

**Last updated:** <date>
**Current state:** Not started

## Recently Completed
- (none yet)

## In Progress
- (none yet)

## Up Next
- Initial project meeting to clarify vision and roadmap

## Blockers
- (none)
```

### docs/inbox.md Template

```markdown
# Inbox

Action items routed from management team discussions.
Mark items [SEEN] after acknowledging. Do not delete — maintains audit trail.

---
(no items yet)
```

### docs/worklog.md Template

```markdown
# Work Log

Append-only log of all work sessions on this project.

---
## <date> - Project created
- Project bootstrapped by Two Slice Team orchestrator
- Awaiting initial project meeting
```

## Functional Requirements

- FR-1: Plugin must follow Claude Code plugin structure with `plugin.json`, skills in `skills/`, and state in `state/`
- FR-2: Central project registry (`state/projects.json`) must be the single source of truth for all project metadata
- FR-3: Co-Founder 1 system prompt must emphasize: Socratic questioning, structured decision-making, playing devil's advocate, summarizing conclusions
- FR-4: Co-Founder 2 system prompt must emphasize: web research, reading past discussion files, making lateral connections, referencing what user has been interested in
- FR-5: All discussion files must follow a consistent markdown template with YAML frontmatter (date, participants, topics, related_projects)
- FR-6: Project bootstrap (`/project create`) must generate the full project template — every file listed in the Project Template section
- FR-7: The `CLAUDE.md` generated for each project must include work documentation instructions so any agent (VP or Ralph iteration) knows how to update status, log work, and check inbox
- FR-8: VP standup must read project state from the filesystem, not from memory or conversation history
- FR-9: Ralph Loop kickoff must use `ralph.sh --tool claude` from the project's `scripts/ralph/` directory
- FR-10: All plugin state must be file-based (markdown + JSON) — no databases, no external services
- FR-11: Each skill must be a separate SKILL.md file with clear trigger descriptions
- FR-12: Agent teams must be used for all multi-agent interactions (standups, co-founder sessions). Subagents should only be used for quick, focused tasks within a single agent's work.
- FR-13: VP teammates must be spawned with their project's directory as the working directory so they have access to that project's CLAUDE.md and .claude/ configuration
- FR-14: Ralph integration must follow the snarktank/ralph workflow: `prd.json` with user stories (passes: true/false), `progress.txt` for cross-iteration memory, `ralph.sh` for the loop script

## Non-Goals

- No web UI or dashboard — everything is CLI/terminal based
- No real-time collaboration between VPs — they operate independently on their projects
- No automated scheduling in MVP — all interactions are user-triggered via slash commands
- No integration with external project management tools (Jira, Linear, etc.)
- No multi-user support — this is a single-user system
- No AI model selection per agent — all agents use whatever Claude model is active in Claude Code
- No cost tracking or token budgeting in MVP (see Phase 3: Finance)
- No agent/tool marketplace browsing in MVP (see Phase 3: HR/IT)

## Technical Considerations

- **Agent Teams (experimental):** Requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`. Has known limitations: no session resumption for in-process teammates, one team per session, no nested teams. The lead session cannot be transferred.
- **Display mode:** Recommend `tmux` split panes for standup visibility. Fall back to in-process mode with Shift+Up/Down navigation.
- **State management:** All state is file-based. `state/projects.json` for registry, markdown files for discussions/standups/meetings. Each project's state lives in its own `docs/` directory.
- **Ralph integration:** Each project gets `scripts/ralph/ralph.sh` copied from the installed ralph plugin. Ralph reads `prd.json` for stories, writes to `progress.txt`, makes git commits. The VP is responsible for ensuring `prd.json` has valid incomplete stories before kicking off Ralph.
- **Context window management:** No longer a concern at the orchestrator level — each teammate has its own context. The project template's `CLAUDE.md` keeps individual agent context focused.
- **Skill isolation:** Each slash command maps to one SKILL.md. Skills are stateless — they read from and write to the filesystem, then spawn agent teams as needed.
- **Permission management:** All teammates inherit the lead's permission mode. Configure allowed commands in each project's `.claude/settings.json` before spawning VPs.
- **File conventions:**
  - Discussions: `discussions/YYYY-MM-DD-<topic>.md`
  - Standups: `standups/YYYY-MM-DD.md`
  - Meetings: `meetings/YYYY-MM-DD-<project>.md`
  - Project docs: `<project-path>/docs/{status,worklog,roadmap,inbox,next-tasks}.md`
  - Ralph files: `<project-path>/{prd.json,progress.txt,scripts/ralph/ralph.sh}`

## Phase 2: Automated Scheduling

After validating the manual workflow, add:
- **Morning standup cron:** A launchd job (macOS) that triggers `/standup` at a configured time each morning
- **Notification:** System notification when standup summary is ready for review
- **Auto-kickoff:** Option for VPs to automatically start Ralph Loops after standup if no blockers
- **Periodic research:** Co-Founder 2 runs research on a schedule and drops findings into a `research/` folder
- **End-of-day summary:** Automated collection of worklog entries across all projects into a daily digest

## Phase 3: Full Operations (Future Work)

The management layer is the first piece of a full operations team. Future phases expand into other organizational functions:

### HR: Talent Acquisition
- **Model scouting:** Evaluate and select AI models for specific project needs (e.g., Sonnet for fast iteration, Opus for complex reasoning, specialized models for domain tasks)
- **Skill discovery:** Browse and evaluate Claude Code skills/plugins from marketplaces for project use
- **Tool sourcing:** Find and integrate MCP servers, APIs, and tools that agents need
- **Agent onboarding:** Standardized process for introducing new agent capabilities to a project (install, configure, test, document)

### IT: Agent Infrastructure
- **Tool provisioning:** Set up and manage MCP servers, API keys, and tool configurations for agents
- **Environment management:** Ensure each project has the right development environment, dependencies, and toolchain
- **Access control:** Manage what tools and permissions each project/agent has access to
- **Infrastructure monitoring:** Track which tools are working, which are failing, and whether agents have what they need

### Finance: Budget Management
- **Token tracking:** Monitor token usage per project, per agent, per session
- **ROI analysis:** Assess whether each project/agent is producing value relative to cost
- **Budget allocation:** Set token/cost budgets per project, alert when approaching limits
- **Cost optimization:** Recommend model downgrades for routine tasks, identify wasteful patterns
- **Spending reports:** Daily/weekly cost breakdowns by project and agent type

## Success Metrics

- User can go from idea discussion to project execution in a single session
- VPs produce actionable work items without explicit task assignment within 2 standups
- All project status is answerable by reading `docs/status.md` — no need to dig through code or logs
- Standup for 3 active projects completes in under 5 minutes of user time
- Discussion context from co-founder sessions shows up in VP planning within one standup cycle
- Ralph Loop can start from a VP-prepared `prd.json` without manual intervention

## Open Questions

1. Should co-founder personas be customizable (e.g., different expertise domains), or fixed?
2. How granular should worklog entries be? Per-commit? Per-task? Per-session?
3. Should VPs have different "personalities" or management styles, or all follow the same template?
4. How many Ralph Loop iterations should a VP kick off by default? (Ralph default is 10)
5. Should there be a concept of "urgency" or "priority" across projects for standup ordering?
6. When agent teams limitation of "one team per session" conflicts with needing both a co-founder session and a VP standup — do we clean up one team before starting another, or find a workaround?
7. How should the project template handle projects that already exist (have code but weren't created via `/project create`)?
