---
name: project-work
description: Kick off a Ralph Loop in a new terminal for autonomous execution on a project.
argument-hint: "<project-id>"
disable-model-invocation: true
---

Kick off a Ralph Loop in a new terminal for autonomous execution on a project.

## Usage

```
/tst:project-work my-cool-app
```

## Instructions

When this skill is triggered, perform the following steps:

### 0. Load HQ Path

Read `~/.config/tst/config.json` and extract the `hq_path` value. The project registry (`state/projects.json`) is stored at this path.

If `config.json` doesn't exist, display: "HQ not configured. Run `/tst:setup` to initialize your HQ first."

### 1. Look Up Project

1. Read `{{HQ_PATH}}/state/projects.json`
2. Find the project with matching `id` from `$ARGUMENTS`
3. If not found, display: "Project '<project-id>' not found. Run `/tst:project-list` to see available projects."
4. Extract: `name`, `path`

### 2. Verify Prerequisites

Change into the project directory and check the following:

**prd.json:**
- Check that `<project-path>/prd.json` exists AND contains at least one user story where `passes` is `false`
- If `prd.json` doesn't exist, or has no stories, or all stories have `passes: true`, display:
  ```
  No incomplete user stories found for this project.
  Run /tst:project-meeting <project-id> to define user stories first.
  ```
  Then stop.

**scripts/ralph/ralph.sh:**
- Check that `<project-path>/scripts/ralph/ralph.sh` exists and is executable
- If not, attempt to fetch from GitHub:
  ```bash
  mkdir -p <project-path>/scripts/ralph
  curl -fsSL -o <project-path>/scripts/ralph/ralph.sh https://raw.githubusercontent.com/snarktank/ralph/main/ralph.sh
  curl -fsSL -o <project-path>/scripts/ralph/CLAUDE.md https://raw.githubusercontent.com/snarktank/ralph/main/CLAUDE.md
  curl -fsSL -o <project-path>/scripts/ralph/prompt.md https://raw.githubusercontent.com/snarktank/ralph/main/prompt.md
  chmod +x <project-path>/scripts/ralph/ralph.sh
  ```
- If download fails, display:
  ```
  Could not find or download Ralph Loop. Install it manually from https://github.com/snarktank/ralph
  ```
  Then stop.

**progress.txt:**
- Check that `<project-path>/progress.txt` exists
- If not, create it with initial content:
  ```
  ## Codebase Patterns

  ---
  ```

### 3. Check Terminal Preference

Read `~/.config/tst/config.json` and check for a `terminal_app` value.

If `terminal_app` is not set, ask the user which terminal app they use:
- **Terminal** (macOS default)
- **iTerm2**
- **Warp**
- **Other** (ask for app name)

Save the choice to `~/.config/tst/config.json` as `"terminal_app"` (one of: `"Terminal"`, `"iTerm2"`, `"Warp"`, or the custom app name).

### 4. Log Work Session Start

Append an entry to `<project-path>/docs/worklog.md`:

```markdown
## YYYY-MM-DD - Ralph Loop launched

- Initiated autonomous work session via `/tst:project-work`
- Stories to complete: <count of stories where passes is false>
- Starting with: <title of highest priority incomplete story>
```

### 5. Launch Ralph Loop in New Terminal

Build the Ralph command:
```bash
cd <project-path> && scripts/ralph/ralph.sh --tool claude
```

Open a new terminal window using the configured terminal app:

**Terminal.app:**
```bash
osascript -e 'tell application "Terminal" to do script "cd <project-path> && scripts/ralph/ralph.sh --tool claude"'
```

**iTerm2:**
```bash
osascript -e 'tell application "iTerm2" to create window with default profile command "cd <project-path> && scripts/ralph/ralph.sh --tool claude"'
```

**Warp:**
```bash
osascript -e 'tell application "Warp" to activate'
osascript -e 'tell application "System Events" to tell process "Warp" to keystroke "t" using command down'
sleep 1
osascript -e 'tell application "System Events" to tell process "Warp" to keystroke "cd <project-path> && scripts/ralph/ralph.sh --tool claude"'
osascript -e 'tell application "System Events" to tell process "Warp" to key code 36'
```

**Other (custom app):**
Try the Terminal.app approach with the custom app name. If it fails, display the command and ask the user to run it manually.

### 6. Report to User

Display a summary:

```
Ralph Loop launched for <project-name>!

  Terminal: <terminal-app>
  Project: <project-path>
  Stories to complete: <N>

  Ralph is running in a new terminal window.
  You can continue working here while Ralph executes autonomously.

Check progress:
  - Run /tst:project-status <project-id> to check on Ralph's progress
  - Run /tst:standup for a status report across all projects
```
