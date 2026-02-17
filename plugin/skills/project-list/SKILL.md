# /project list, /project status, /project pause, /project resume

Manage and view Two Slice Team projects.

## Usage

```
/project list
/project status <project-id>
/project pause <project-id>
/project resume <project-id>
```

## Instructions

### 0. Load HQ Path

Read `~/.claude/plugins/two-slice-team/config.json` and extract the `hq_path` value. The project registry (`state/projects.json`) is stored at this path.

If `config.json` doesn't exist, display: "HQ not configured. Run `/tst setup` to initialize your HQ first."

### /project list

1. Read `{{HQ_PATH}}/state/projects.json`
2. For each project in the `projects` array, display a table or formatted list with:
   - **Name**: The project name
   - **Status**: `active` or `inactive`
   - **VP**: The VP name assigned to this project
   - **Path**: The project directory path
   - **Vision**: The vision summary (first 80 characters, or "(not set)" if empty)
3. If there are no projects, display: "No projects registered yet. Use `/project create <name>` to create one."

**Example output:**
```
## Projects

| Name | Status | VP | Path | Vision |
|------|--------|----|------|--------|
| my-cool-app | active | VP Sierra Search | ~/Documents/my-cool-app | Building a better... |
| payments | inactive | VP Parker Pay | ~/Documents/payments | (not set) |
```

### /project status <project-id>

1. Read `{{HQ_PATH}}/state/projects.json`
2. Find the project with matching `id`
3. If not found, display: "Project '<project-id>' not found. Run `/project list` to see available projects."
4. If found, read the project's `docs/status.md` file from the project's `path`
5. Display the contents of `docs/status.md` along with project metadata:
   - Project name, VP name, status, path
6. If `docs/status.md` doesn't exist, display: "No status file found at `<path>/docs/status.md`. The project may not be fully set up."

### /project pause <project-id>

1. Read `{{HQ_PATH}}/state/projects.json`
2. Find the project with matching `id`
3. If not found, display: "Project '<project-id>' not found. Run `/project list` to see available projects."
4. If found, set the project's `status` to `"inactive"`
5. Write the updated JSON back to `{{HQ_PATH}}/state/projects.json`
6. Display: "Project '<name>' paused. It won't appear in standups. Use `/project resume <id>` to reactivate."

### /project resume <project-id>

1. Read `{{HQ_PATH}}/state/projects.json`
2. Find the project with matching `id`
3. If not found, display: "Project '<project-id>' not found. Run `/project list` to see available projects."
4. If found, set the project's `status` to `"active"`
5. Write the updated JSON back to `{{HQ_PATH}}/state/projects.json`
6. Display: "Project '<name>' resumed! It will now appear in standups."
