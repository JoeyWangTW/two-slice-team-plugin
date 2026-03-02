---
name: sync
description: Compare the plugin (source of truth) against HQ and all active projects, show what's out of sync, and offer to fix it.
disable-model-invocation: true
---

Sync HQ and project installations with the latest plugin files. Detects drift (missing dirs, outdated files, broken symlinks) and offers to fix it.

## Usage

```
/tst:sync
```

## Instructions

When this skill is triggered, perform the following steps:

### 0. Resolve Paths

1. **Plugin root** — derive from the path of this SKILL.md file: go up two levels from `skills/sync/SKILL.md` to get the plugin root directory.
2. **HQ path** — read `~/.config/tst/config.json` and extract `hq_path`. If the config doesn't exist, display: "HQ not configured. Run `/tst:setup` to initialize your HQ first." and stop.
3. **Projects** — read `<hq_path>/state/projects.json` and collect all projects with `status: "active"`.

### 1. Check HQ Sync

Compare the plugin against the HQ installation:

#### CLAUDE.md

- Read `<plugin_root>/CLAUDE.md` (source of truth)
- Read `<hq_path>/CLAUDE.md` (installed copy)
- If the HQ copy doesn't exist, flag as **missing**
- If both exist, compare their contents. If different, flag as **out of date** and prepare a diff summary (note which sections were added/changed/removed — don't dump the full diff, just describe the changes)

#### Directory structure

Check that all expected HQ directories exist:
- `state/`
- `discussions/`
- `ideas/`
- `standups/`
- `meetings/`
- `research/`

Flag any that are missing.

### 2. Check Project Sync

For each active project, check the following:

#### post-ralph-debrief.sh

- Read `<plugin_root>/scripts/post-ralph-debrief.sh` (source of truth)
- Read `<project_path>/scripts/post-ralph-debrief.sh` (installed copy)
- If the project copy doesn't exist, flag as **missing**
- If both exist, compare their contents. If different, flag as **out of date**

#### Directory structure

Check that these directories exist in the project:
- `docs/`
- `scripts/`
- `scripts/ralph/`

Flag any that are missing.

#### Ralph symlinks

Check that these symlinks exist and point to the correct targets:
- `<project_path>/scripts/ralph/prd.json` → should be a symlink to `../../prd.json`
- `<project_path>/scripts/ralph/progress.txt` → should be a symlink to `../../progress.txt`

Flag if missing, not a symlink, or pointing to the wrong target.

### 3. Present Findings

Display a sync report grouped by target:

```
## Sync Report

### HQ (<hq_path>)

- CLAUDE.md: ✅ Up to date / ⚠️ Out of date (describe changes) / ❌ Missing
- Directories: ✅ All present / ⚠️ Missing: ideas/, research/

### <project-name> (<project_path>)

- post-ralph-debrief.sh: ✅ Up to date / ⚠️ Out of date / ❌ Missing
- Directories: ✅ All present / ⚠️ Missing: scripts/ralph/
- Ralph symlinks: ✅ OK / ⚠️ prd.json missing, progress.txt wrong target

### <project-name-2> (...)
...
```

If everything is in sync, display:

```
Everything is in sync! No updates needed.
```

And stop here — no need to continue to step 4.

### 4. Ask What to Update

Use `AskUserQuestion` to let the user choose what to sync. Options:

- **Sync everything** — apply all fixes
- **HQ only** — only sync HQ files and directories
- **Projects only** — only sync project files, directories, and symlinks
- **Skip** — don't apply any changes

### 5. Apply Updates

Based on the user's choice, apply the relevant fixes:

#### HQ fixes

- **CLAUDE.md missing or out of date** — copy `<plugin_root>/CLAUDE.md` to `<hq_path>/CLAUDE.md`
- **Missing directories** — create them with `mkdir -p`

#### Project fixes

- **post-ralph-debrief.sh missing or out of date** — copy from `<plugin_root>/scripts/post-ralph-debrief.sh` to `<project_path>/scripts/post-ralph-debrief.sh` and `chmod +x` it
- **Missing directories** — create them with `mkdir -p`
- **Missing or broken symlinks** — create/fix with:
  ```bash
  ln -sf ../../prd.json <project_path>/scripts/ralph/prd.json
  ln -sf ../../progress.txt <project_path>/scripts/ralph/progress.txt
  ```

### 6. Confirm

After applying updates, display what was changed:

```
Sync complete!

  Updated:
    - HQ: CLAUDE.md updated
    - HQ: Created ideas/ directory
    - my-project: Copied post-ralph-debrief.sh
    - my-project: Fixed prd.json symlink
```
