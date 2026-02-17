---
name: cofounder-research
description: Spawn the Research co-founder to asynchronously research a topic and save findings to HQ.
argument-hint: "<topic>"
disable-model-invocation: true
---

Spawn the Research co-founder to asynchronously research a topic. Results are saved to the HQ research directory.

## Usage

```
/tst:cofounder-research market trends in AI tooling
/tst:cofounder-research competitor analysis for note-taking apps
```

## Instructions

When this skill is triggered, perform the following steps:

### 0. Load HQ Path

Read `~/.config/tst/config.json` and extract the `hq_path` value. Research results are stored at `{{HQ_PATH}}/research/`.

If `config.json` doesn't exist, display: "HQ not configured. Run `/tst:setup` to initialize your HQ first."

### 1. Parse Topic

Extract the research topic from `$ARGUMENTS`.

If no topic is provided, display: "Please provide a research topic. Example: `/tst:cofounder-research market trends in AI tooling`"

Generate a `<topic-slug>` — a short kebab-case version of the topic (e.g., "market trends in AI tooling" becomes "market-trends-ai-tooling").

### 2. Gather Context

Before researching, read for relevant context:
- `{{HQ_PATH}}/state/projects.json` — to understand active projects and find connections
- Scan `{{HQ_PATH}}/discussions/` — for past discussions related to the topic
- Scan `{{HQ_PATH}}/research/` — for past research that might be related

### 3. Research the Topic

Use web search to research the topic thoroughly:
- Search for current information, trends, and data
- Look for relevant examples, case studies, and real-world applications
- Find expert opinions, blog posts, and research papers
- Identify competitors or similar efforts in the space
- Look for cross-domain connections and unexpected angles

### 4. Identify Strategic Connections

Connect research findings to existing context:
- How do findings relate to active projects?
- What strategic opportunities do the findings suggest?
- Are there connections to past discussions or other research?
- What risks or threats does the research reveal?

### 5. Save Research Results

Save the research to `{{HQ_PATH}}/research/YYYY-MM-DD-<topic-slug>.md`:

```markdown
---
topic: <full topic as provided by user>
date: YYYY-MM-DD
related_projects:
  - <project-id-1>
  - <project-id-2>
tags:
  - <tag-1>
  - <tag-2>
---

# Research: <Topic Title>

## Summary
<2-3 paragraph executive summary of key findings>

## Key Findings
- <finding 1 with supporting evidence>
- <finding 2 with supporting evidence>
- <finding 3 with supporting evidence>

## Strategic Insights
- <insight about how this connects to broader goals or opportunities>
- <insight about competitive landscape or market position>
- <insight about timing, trends, or momentum>

## Connections
- **To projects:** <how this relates to active projects>
- **To past discussions:** <links to relevant discussion files>
- **To other research:** <links to related research files>
- **Cross-domain:** <unexpected connections from adjacent fields>

## Sources
- [<source title>](<url>) — <brief note on relevance>
- [<source title>](<url>) — <brief note on relevance>
```

### 6. Route to Project Inboxes

If the research is relevant to specific projects (listed in `related_projects`):

For each related project:
1. Look up the project in `{{HQ_PATH}}/state/projects.json` to get its path
2. Append to that project's `docs/inbox.md`:

```markdown
## From research: <topic> (YYYY-MM-DD)
- <key finding or action item relevant to this project>
- See full research: {{HQ_PATH}}/research/YYYY-MM-DD-<topic-slug>.md
```

### 7. Report to User

Display a summary:

```
Research complete: <topic>

  Saved to: research/YYYY-MM-DD-<topic-slug>.md
  Key findings: <count>
  Related projects: <list or "none">

  Top insights:
  - <insight 1>
  - <insight 2>
  - <insight 3>
```
