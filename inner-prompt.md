# Inner Loop Subagent Prompt Template

Use this template when spawning the inner loop Task subagent.

## Template

```markdown
You are implementing part of a larger task. Work until you hit a blocker
or feel context getting heavy (~15-20 turns), then summarize and exit.

{{#if PROJECT_GUIDANCE}}
## Project Guidance

{{PROJECT_GUIDANCE}}
{{/if}}

## State Context

{{FULL_STATE_MARKDOWN}}

## Current Focus

**Work Unit:** {{WORK_UNIT_NAME}}

**Goal:** {{WORK_UNIT_CONTENT}}

{{#if KNOWN_BAD_ROUTES}}
## Known Bad Routes (Avoid These)

{{KNOWN_BAD_ROUTES}}
{{/if}}

{{#if VERIFIED_FINDINGS}}
## Verified Findings (Build On These)

{{VERIFIED_FINDINGS}}
{{/if}}

## Instructions

1. Implement the current work unit
2. Run tests as you go to verify your work
3. Use any tools you need - you have full access including spawning subagents
4. If blocked (need decision, unclear requirement, external dependency), stop and report
5. If you've made ~15-20 turns or context feels heavy, find a natural stopping point
6. Return a structured summary in the format below

## Exit Criteria

Stop and return your summary when ANY of these apply:
- Work unit implementation is complete
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

Be specific in your summary - the outer loop will use this for review.
```

## Substitution Variables

Required:
- `{{FULL_STATE_MARKDOWN}}`: The entire state file content (frontmatter + body)
- `{{WORK_UNIT_NAME}}`: The work unit being implemented (section heading, criterion text, etc.)
- `{{WORK_UNIT_CONTENT}}`: Details about the work unit (section content, related context)

Optional (include if present in state file):
- `{{PROJECT_GUIDANCE}}`: Contents of .ralph.md if found
- `{{KNOWN_BAD_ROUTES}}`: Approaches that failed (from state file)
- `{{VERIFIED_FINDINGS}}`: Validated facts (from state file)

## Notes

The template uses mustache-style conditionals (`{{#if}}...{{/if}}`) for optional sections.
If a variable is not available, omit that section entirely.

Project guidance is critical context - always include it when available. It tells the
inner loop how progress should be tracked, what review expectations are, and any
project-specific conventions to follow.
