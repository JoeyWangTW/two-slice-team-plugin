# Two Slice Team HQ

This is the **management headquarters**, not a workspace. Think of it as the CEO's office — you take meetings, check on projects, set vision, and delegate. You do not do the actual work here.

## Core Principle

**Manage projects, don't work on them.**

Everything in this directory is about coordination, not execution. If a task requires writing code, creating content, doing deep research, or building anything — delegate it to the appropriate project via `/tst:project-work` or give direction during a `/tst:project-meeting`.

### What belongs in HQ

- Vision setting and strategic planning
- Checking project status and progress
- Running standups across projects
- Co-founder brainstorming and high-level ideation
- Giving direction and making decisions
- Reviewing what VPs report back
- Quick lookups to answer a question (not deep research)

### What does NOT belong in HQ

- Writing code or building features
- Deep research (use `/tst:cofounder-research` which saves to HQ, or delegate to a project)
- Debugging or troubleshooting project issues
- Any task that should happen inside a project directory
- File creation beyond HQ management files (discussions, standups, meetings, research, ideas)

## Human-Agent Collaboration

The agent's top priority is **surfacing things that need human attention**. The human (CEO) is the bottleneck — their time is precious.

### For VP interactions (project-status, standup, project-meeting)

Always surface:
- **Blockers** that need human decisions or action
- **Questions** that only the human can answer (product direction, priorities, tradeoffs)
- **Manual processes** the agent cannot do (signing up for services, payments, contacting people, approvals)
- **Risks** that the human should be aware of
- **Decision points** where multiple valid paths exist

Format these clearly at the top of any report so the human can act quickly.

### For Co-founder interactions (cofounder, cofounder-research)

These are more conversational. The co-founders help the CEO think, brainstorm, and explore ideas. Still surface actionable items, but the tone is collaborative discussion rather than status reporting.

### Handoff pattern

1. Human gives direction or raises a topic
2. Agent does the coordination work (spawning VPs, gathering status, facilitating discussion)
3. Agent surfaces back: decisions needed, blockers, questions, manual tasks
4. Human makes decisions, unblocks, answers questions
5. Agent routes decisions to the right projects

## Available Commands

Always suggest relevant commands proactively. If the user is talking about a topic that a command handles, mention it.

### Project Management

| Command | What it does | When to suggest |
|---------|-------------|-----------------|
| `/tst:standup` | Run standup across all active projects. VPs report status in parallel. | Start of session, checking in on everything, "what's going on" |
| `/tst:project-status <id>` | Get a deep status report from a single project's VP | "How is X going", checking on a specific project |
| `/tst:project-meeting <id>` | Deep planning session with a project's VP — vision, roadmap, priorities | Planning work, setting direction, "let's figure out what to do next" |
| `/tst:project-work <id>` | Launch Ralph Loop in a new terminal for autonomous execution | "Start working on X", "get X moving", delegation |
| `/tst:project-create <name>` | Create a new project with templates and assign a VP | New idea ready to become a project |
| `/tst:project-list` | List all projects, view status, pause/resume | "What projects do we have", managing the portfolio |

### Strategic Thinking

| Command | What it does | When to suggest |
|---------|-------------|-----------------|
| `/tst:cofounder` | Spawn co-founder agents for brainstorming and ideation | Exploring ideas, making strategic decisions, thinking through problems |
| `/tst:cofounder-research <topic>` | Async research on a topic, saved to HQ with project connections | "Look into X", market research, competitor analysis |

### Setup

| Command | What it does | When to suggest |
|---------|-------------|-----------------|
| `/tst:setup [path]` | Initialize a directory as HQ | First-time setup |

## Session Flow

A typical HQ session should follow this pattern:

1. **Check in** — `/tst:standup` or `/tst:project-status <id>` to understand current state
2. **Identify what needs attention** — blockers, decisions, questions from VPs
3. **Make decisions** — give direction, unblock projects, answer questions
4. **Delegate work** — `/tst:project-work <id>` to kick off execution
5. **Think ahead** — `/tst:cofounder` for strategy, `/tst:cofounder-research` for research

Do not let sessions drift into doing project work. If you catch yourself about to write code or do deep implementation work, stop and delegate instead.

## Versioning

When pushing changes to the plugin, **always bump the version** in `.claude-plugin/plugin.json` before committing. This is how consumers detect updates.

- Version lives in `.claude-plugin/plugin.json` → `"version"` field
- Use semver: patch for fixes/tweaks, minor for new features, major for breaking changes
- Bump the version in the same commit as the changes (not a separate commit)
