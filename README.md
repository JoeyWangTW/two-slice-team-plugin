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

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- Agent Teams enabled: `export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`

## Install

Clone this repo and symlink it into your Claude Code plugins directory:

```sh
git clone https://github.com/JoeyWangTW/two-slice-team-plugin.git
mkdir -p ~/.claude/plugins
ln -s "$(pwd)/two-slice-team-plugin/plugin" ~/.claude/plugins/two-slice-team
```

## Setup

Initialize a directory as your HQ (where all data lives):

```
/tst setup                    # Use current directory
/tst setup ~/my-hq            # Use a specific path
```

This creates the HQ structure and writes the config so all other commands know where to find it:

```
~/my-hq/
├── state/projects.json       # Project registry
├── discussions/              # Co-founder session notes
├── standups/                 # Daily standup summaries
└── meetings/                 # Project meeting notes
```

## Commands

| Command | Description |
|---------|-------------|
| `/tst setup [path]` | Initialize a directory as HQ |
| `/project create <name>` | Create a new project with templates |
| `/project list` | List all projects |
| `/project status <id>` | View a project's status |
| `/project pause <id>` | Pause a project (hide from standups) |
| `/project resume <id>` | Resume a paused project |
| `/project meeting <id>` | Deep planning session with a project's VP |
| `/project work <id>` | Kick off Ralph Loop for autonomous execution |
| `/cofounder` | Spawn co-founder agents for ideation |
| `/standup` | Run a standup across all active projects |

## Typical Workflow

1. **Set up HQ** — `/tst setup ~/two-slice-hq`
2. **Brainstorm** — `/cofounder` to discuss an idea with your co-founders
3. **Create a project** — `/project create my-app`
4. **Plan it** — `/project meeting my-app` to define vision and user stories with the VP
5. **Build it** — `/project work my-app` to kick off autonomous execution
6. **Check in** — `/standup` to get status reports from all VPs

## Architecture

The plugin separates code from data:

- **Plugin** (`~/.claude/plugins/two-slice-team/`) — skills, templates, config
- **HQ** (wherever you set it up) — projects, discussions, standups, meetings

This means you can reinstall the plugin without losing data, and your HQ can be git-tracked or synced independently.

## License

MIT
