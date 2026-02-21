---
name: setup
description: Initialize a directory as the Two Slice Team HQ
argument-hint: "[path]"
disable-model-invocation: true
---

Initialize a directory as the Two Slice Team HQ. This creates the data directories and writes the config so all other skills know where to find HQ.

## Usage

```
/tst:setup                    # Initialize current directory as HQ
/tst:setup /path/to/my/hq     # Initialize a specific path as HQ
```

## Instructions

When this skill is triggered, perform the following steps:

### 1. Determine HQ Path

- If a path argument is provided via `$ARGUMENTS`, use that as the HQ path (resolve to absolute path)
- If no argument is provided, use the current working directory
- Expand `~` to the user's home directory if present

### 2. Create HQ Directory Structure

Create the following directories and files at the HQ path:

```
<hq-path>/
├── CLAUDE.md
├── state/
│   └── projects.json
├── discussions/
├── standups/
├── meetings/
└── research/
```

**state/projects.json** (empty project registry):
```json
{
  "projects": []
}
```

Create the `discussions/`, `standups/`, `meetings/`, and `research/` directories (they start empty).

### 3. Create HQ CLAUDE.md

Check if `<hq-path>/CLAUDE.md` already exists.

**If it does NOT exist:** Read the plugin's own `CLAUDE.md` (located at the root of the two-slice-team-plugin directory) and use it as the template to create `<hq-path>/CLAUDE.md`. This file defines how HQ sessions should work — managing projects, not working on them.

**If it ALREADY exists:** Do not overwrite it. Instead, note to the user:
```
CLAUDE.md already exists — skipping. See the plugin's CLAUDE.md for the latest HQ reference:
  https://github.com/JoeyWangTW/two-slice-team-plugin/blob/main/CLAUDE.md
```

### 4. Write Config

Write `~/.config/tst/config.json` with the HQ path:

```json
{
  "hq_path": "<absolute-hq-path>"
}
```

Create the `~/.config/tst/` directory if it doesn't exist.

If the file already exists, read it first and confirm with the user before overwriting:
- Show the current `hq_path` value
- Ask: "HQ is currently set to `<old-path>`. Update to `<new-path>`?"

### 5. Confirm to User

Display:

```
Two Slice Team HQ initialized!

  HQ path: <hq-path>
  Config:  ~/.config/tst/config.json

  Created:
    - CLAUDE.md (HQ operating guide)
    - state/projects.json
    - discussions/
    - standups/
    - meetings/
    - research/

Next steps:
  - /tst:project-create <name>         Create your first project
  - /tst:cofounder                     Start a co-founder discussion
  - /tst:cofounder-research <topic>    Research a topic asynchronously
```
