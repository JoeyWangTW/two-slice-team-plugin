---
name: project-work
description: Start a work session on a project. Uses Ralph Loop for code projects with user stories, or a VP-guided session for general work.
argument-hint: "<project-id>"
disable-model-invocation: true
---

Start a work session on a project. Automatically detects whether to use Ralph Loop (code projects with user stories) or a VP-guided session (general work).

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

### 2. Determine Work Mode

Change into the project directory and check which work mode to use:

- **Code Mode (Ralph Loop):** If `<project-path>/prd.json` exists AND contains at least one user story where `passes` is `false` → proceed to **Step 3A**
- **General Mode (VP Work Session):** If `prd.json` does not exist, or has an empty `userStories` array, or all stories have `passes: true` → proceed to **Step 3B**

---

### 3A. Code Mode — Ralph Loop

This path handles code projects with incomplete user stories.

#### 3A.1 Verify Prerequisites

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

#### 3A.2 Log Work Session Start

Append an entry to `<project-path>/docs/worklog.md`:

```markdown
## YYYY-MM-DD - Ralph Loop started

- Initiated autonomous work session via `/tst:project-work`
- Stories to complete: <count of stories where passes is false>
- Starting with: <title of highest priority incomplete story>
```

#### 3A.3 Start Ralph Loop

Run the Ralph Loop from the project directory:

```bash
cd <project-path> && scripts/ralph/ralph.sh --tool claude
```

**Important:** The ralph.sh script runs iteratively — each iteration picks up the next incomplete story. Let it run until it completes or reaches max iterations.

#### 3A.4 Post-Completion Update

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

#### 3A.5 Report to User

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
  - Run /tst:project-work <id> again to continue
  - Run /tst:project-list status <id> to see full status
  - Run /tst:project-meeting <id> to adjust plans
```

---

### 3B. General Mode — VP Work Session

This path handles any project type — research, physical business, marketing, content, social experiments, etc.

#### 3B.1 Load Project Context

Read the project's VP name from `{{HQ_PATH}}/state/projects.json`.

Read the following files from the project directory:
- `CLAUDE.md` (project overview)
- `docs/status.md` (current status)
- `docs/roadmap.md` (roadmap and milestones)
- `docs/next-tasks.md` (prioritized task list)
- `docs/inbox.md` (pending action items)
- `docs/worklog.md` (recent work history)

#### 3B.2 Log Work Session Start

Append an entry to `<project-path>/docs/worklog.md`:

```markdown
## YYYY-MM-DD - VP work session started

- Initiated work session via `/tst:project-work`
- Mode: General (VP-guided)
```

#### 3B.3 Identify Work Items

As the VP, review the project documents and identify work to be done. Check in this priority order:
1. **Inbox items** — action items from discussions, standups, or research
2. **Next tasks** — items listed in `docs/next-tasks.md`
3. **Roadmap items** — unchecked items from `docs/roadmap.md`

Present the work items to the user and confirm what to focus on.

#### 3B.4 Execute Work

Work through the confirmed items. This may include:
- **Research:** Use web search to investigate topics, gather information, write findings
- **Writing:** Draft documents, content, plans, proposals
- **Planning:** Break down milestones into tasks, create timelines, identify dependencies
- **Outreach:** Draft emails, messages, or communication materials
- **Analysis:** Analyze data, competitors, market conditions
- **Organization:** Organize files, update documentation, clean up project structure
- **Any other work** relevant to the project's goals

For each completed item:
- Mark it done in the relevant document (check off in roadmap, remove from next-tasks, mark as [SEEN] in inbox)
- Note what was accomplished

#### 3B.5 Update Project Documents

After the work session:

1. Update `docs/status.md` with:
   - What was completed
   - What's in progress
   - Updated "Last updated" date

2. Update `docs/next-tasks.md` with:
   - Remove completed items
   - Add any new tasks discovered during the session

3. Append to `docs/worklog.md`:
   ```markdown
   ## YYYY-MM-DD - VP work session completed

   - Items completed: <list of completed items>
   - Items remaining: <count of remaining items>
   - New items discovered: <any new tasks added>
   ```

#### 3B.6 Report to User

Display a summary:

```
Work session finished for <project-name>!

  Mode: VP-guided
  Items completed: <N>
  Items remaining: <M>

  Completed:
  - <item 1>
  - <item 2>

  Up next:
  - <next item 1>
  - <next item 2>

Next steps:
  - Run /tst:project-work <id> again to continue
  - Run /tst:project-list status <id> to see full status
  - Run /tst:project-meeting <id> to adjust plans
```
