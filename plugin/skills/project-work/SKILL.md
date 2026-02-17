# /project work

Prepare and kick off a Ralph Loop for autonomous execution on a project.

## Usage

```
/project work <project-id>
```

**Example:**
```
/project work my-cool-app
```

## Instructions

When this skill is triggered, perform the following steps:

### 0. Load HQ Path

Read `~/.claude/plugins/two-slice-team/config.json` and extract the `hq_path` value. The project registry (`state/projects.json`) is stored at this path.

If `config.json` doesn't exist, display: "HQ not configured. Run `/tst setup` to initialize your HQ first."

### 1. Look Up Project

1. Read `{{HQ_PATH}}/state/projects.json`
2. Find the project with matching `id`
3. If not found, display: "Project '<project-id>' not found. Run `/project list` to see available projects."
4. Extract: `name`, `path`

### 2. Verify Prerequisites

Change into the project directory and verify the following files exist:

**prd.json:**
- Check that `<project-path>/prd.json` exists
- Check that it contains at least one user story where `passes` is `false`
- If prd.json doesn't exist or has no stories, display:
  ```
  No PRD found (or no incomplete stories) for this project.
  Run `/project meeting <project-id>` first to plan user stories.
  ```
  And stop.

**progress.txt:**
- Check that `<project-path>/progress.txt` exists
- If not, create it with initial content:
  ```
  ## Codebase Patterns

  ---
  ```

**scripts/ralph/ralph.sh:**
- Check that `<project-path>/scripts/ralph/ralph.sh` exists
- If not, create the `scripts/ralph/` directory and copy `ralph.sh` from the Two Slice Team repo or create it with the standard Ralph Loop script
- Ensure `ralph.sh` is executable: run `chmod +x <project-path>/scripts/ralph/ralph.sh`

**scripts/ralph/CLAUDE.md:**
- Check that `<project-path>/scripts/ralph/CLAUDE.md` exists
- If not, create it with the standard Ralph agent instructions that tell the agent to:
  1. Read the PRD at `prd.json`
  2. Read `progress.txt` and check Codebase Patterns
  3. Check out the correct branch from PRD `branchName`
  4. Pick the highest priority story where `passes: false`
  5. Implement that story
  6. Run quality checks
  7. Commit changes
  8. Update the PRD and progress log
  9. Reply with `<promise>COMPLETE</promise>` when all stories pass

### 3. Log Work Session Start

Append an entry to `<project-path>/docs/worklog.md`:

```markdown
## YYYY-MM-DD - Ralph Loop started

- Initiated autonomous work session via `/project work`
- Stories to complete: <count of stories where passes is false>
- Starting with: <title of highest priority incomplete story>
```

### 4. Start Ralph Loop

Run the Ralph Loop from the project directory:

```bash
cd <project-path> && scripts/ralph/ralph.sh --tool claude
```

**Important:** The ralph.sh script runs iteratively — each iteration picks up the next incomplete story. Let it run until it completes or reaches max iterations.

### 5. Post-Completion Update

After Ralph finishes (or reaches max iterations):

1. Read `<project-path>/prd.json` to check which stories now pass
2. Read `<project-path>/progress.txt` for the latest progress
3. Update `<project-path>/docs/status.md` with:
   - Updated "Recently Completed" section with stories Ralph completed
   - Updated "In Progress" section (if any stories remain incomplete)
   - Updated "Last updated" date
4. Append to `<project-path>/docs/worklog.md`:
   ```markdown
   ## YYYY-MM-DD - Ralph Loop completed

   - Stories completed: <list of completed story IDs and titles>
   - Stories remaining: <count of remaining incomplete stories>
   - Result: <completed all / reached max iterations>
   ```

### 6. Report to User

Display a summary:

```
Ralph Loop finished for <project-name>!

  Completed: <N> stories
  Remaining: <M> stories

  Completed stories:
  - [US-001] <title>
  - [US-002] <title>

  Remaining stories:
  - [US-003] <title> (priority: 3)

Next steps:
  - Run `/project work <id>` again to continue
  - Run `/project status <id>` to see full status
  - Run `/project meeting <id>` to adjust plans
```
