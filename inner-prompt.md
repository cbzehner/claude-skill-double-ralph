# Inner Loop Subagent Prompt Template

Use this template when spawning the inner loop Task subagent.

## Template

```markdown
You are implementing part of a plan. Work until you hit a blocker
or feel context getting heavy (~15-20 turns), then summarize and exit.

## Plan Context

{{FULL_PLAN_MARKDOWN}}

## Current Focus

**Section:** {{SECTION_NAME}}

**Goal:** {{SECTION_CONTENT}}

## Instructions

1. Implement the current focus section
2. Run tests as you go to verify your work
3. Use any tools you need - you have full access including spawning subagents
4. If blocked (need decision, unclear requirement, external dependency), stop and report
5. If you've made ~15-20 turns or context feels heavy, find a natural stopping point
6. Return a structured summary in the format below

## Exit Criteria

Stop and return your summary when ANY of these apply:
- Section implementation is complete
- You hit a blocker requiring decisions from the outer loop
- You've made ~15-20 tool calls and should find a stopping point
- Context feels heavy (you're losing track of earlier work)

## Summary Format

When exiting, return this YAML structure:

```yaml
status: completed | partial | blocked
work_done:
  - "Description of what was accomplished"
  - "Another accomplishment"
blockers:
  - "Description of any blockers (empty if none)"
gaps_discovered:
  - "New gaps found during implementation"
edge_cases_discovered:
  - "Edge cases that need handling"
files_changed:
  - path/to/file.ext
  - another/file.ext
tests_status: passed | failed | not_run
next_steps:
  - "What should be done next"
  - "Another follow-up item"
```

Be specific in your summary - the outer loop will use this for magi review.
```

## Substitution Variables

- `{{FULL_PLAN_MARKDOWN}}`: The entire plan file content (frontmatter + body)
- `{{SECTION_NAME}}`: The `## ` heading being worked on
- `{{SECTION_CONTENT}}`: The content under that heading until the next `## `
