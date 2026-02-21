# Two Slice Team

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that gives you an AI operations team. It models a startup org structure — two co-founder agents for ideation and research, VP agents for per-project execution — all coordinated through slash commands.

You're the CEO. Your co-founders help you think. Your VPs run the projects.

The name is inspired by Every's article [*The Two-Slice Team*](https://every.to/chain-of-thought/the-two-slice-team) — the idea that one person plus AI can do the work of a much larger team.

## How It Works

Built on Claude Code's **Agent Teams** (experimental). Each co-founder and VP is a separate teammate with its own context window. They communicate via the agent teams messaging system and operate in parallel.

```
┌─────────────────────────────────────────────────┐
│  YOU (CEO) ─ Claude Code session                │
│                                                 │
│  ┌──────────────┐  ┌──────────────┐             │
│  │ Co-Founder 1 │  │ Co-Founder 2 │             │
│  │ (Discussion) │  │ (Research)   │             │
│  └──────────────┘  └──────────────┘             │
│                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐        │
│  │ VP Proj1 │ │ VP Proj2 │ │ VP Proj3 │        │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘        │
│       ▼             ▼             ▼              │
│  Ralph Loop    Ralph Loop    Ralph Loop          │
└─────────────────────────────────────────────────┘
```

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) v1.0.33+
- Agent Teams enabled: `export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
- [Ralph Loop](https://github.com/snarktank/ralph) installed (for autonomous execution via `/tst:project-work`)

## Install

**Option A: Marketplace (recommended)**

```sh
/plugin marketplace add JoeyWangTW/two-slice-team-plugin
/plugin install tst@two-slice-team
```

**Option B: Load directly during development**

```sh
claude --plugin-dir /path/to/two-slice-team-plugin
```

**Option C: From a clone**

```sh
git clone https://github.com/JoeyWangTW/two-slice-team-plugin.git
claude --plugin-dir ./two-slice-team-plugin
```

All skills are namespaced under `tst:` (e.g. `/tst:setup`, `/tst:cofounder`).

## Setup

Initialize a directory as your HQ (where all data lives):

```
/tst:setup                    # Use current directory
/tst:setup ~/my-hq            # Use a specific path
```

This creates the HQ structure and saves the path to `~/.config/tst/config.json`:

```
~/my-hq/
├── CLAUDE.md                 # HQ operating guide — keeps sessions focused on management
├── state/projects.json       # Project registry
├── discussions/              # Co-founder session notes
├── standups/                 # Daily standup summaries
├── meetings/                 # Project meeting notes
└── research/                 # Co-founder research results
```

The `CLAUDE.md` is key — it tells Claude that HQ is for managing projects (vision, planning, check-ins, delegation), not for doing the actual work. It lists all available commands and defines how the human-agent handoff should work.

## Commands

| Command | Description |
|---------|-------------|
| `/tst:setup [path]` | Initialize a directory as HQ |
| `/tst:project-create <name>` | Create a new project with templates |
| `/tst:project-list` | List all projects |
| `/tst:project-list status <id>` | View a project's status |
| `/tst:project-list pause <id>` | Pause a project (hide from standups) |
| `/tst:project-list resume <id>` | Resume a paused project |
| `/tst:project-status <id>` | Get a comprehensive status report from a project's VP |
| `/tst:project-meeting <id>` | Deep planning session with a project's VP |
| `/tst:project-work <id>` | Launch Ralph Loop in a new terminal for a project |
| `/tst:cofounder` | Spawn co-founder agents for ideation |
| `/tst:cofounder-research <topic>` | Research a topic asynchronously |
| `/tst:standup` | Run a standup across all active projects |

## Typical Workflow

1. **Set up HQ** — `/tst:setup ~/two-slice-hq`
2. **Brainstorm** — `/tst:cofounder` to discuss an idea with your co-founders
3. **Create a project** — `/tst:project-create my-app`
4. **Plan it** — `/tst:project-meeting my-app` to define vision and user stories with the VP
5. **Execute** — `/tst:project-work my-app` to launch Ralph Loop in a new terminal for autonomous execution
6. **Check in** — `/tst:standup` to get status reports from all VPs

## Plugin Structure

```
two-slice-team-plugin/
├── .claude-plugin/
│   ├── plugin.json           # Plugin manifest
│   └── marketplace.json      # Marketplace catalog
├── skills/
│   ├── setup/SKILL.md        # /tst:setup
│   ├── cofounder/SKILL.md    # /tst:cofounder
│   ├── cofounder-research/SKILL.md  # /tst:cofounder-research
│   ├── standup/SKILL.md      # /tst:standup
│   ├── project-create/SKILL.md
│   ├── project-list/SKILL.md
│   ├── project-status/SKILL.md
│   ├── project-meeting/SKILL.md
│   └── project-work/SKILL.md
└── templates/                # Project scaffolding templates
```

The plugin separates code from data:

- **Plugin** (this repo) — skills and templates, installed via Claude Code
- **HQ** (wherever you set it up) — projects, discussions, standups, meetings
- **Config** (`~/.config/tst/config.json`) — points to your HQ

This means you can reinstall the plugin without losing data, and your HQ can be git-tracked or synced independently.

## License

MIT
