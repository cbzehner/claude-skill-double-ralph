---
name: double-ralph
description: Nested agent loop for plan execution. Outer loop orchestrates planning/review, inner loops implement. Use when you have a plan file to execute incrementally with magi review between chunks.
---

# Double Ralph: Nested Agent Loop

Execute a plan file through iterative implementation cycles with magi review.

## Invocation

```
/double-ralph <plan-file>
```

Example: `/double-ralph plans/my-feature.md`

## Plan File Format

Plans must have YAML frontmatter:

```yaml
---
status: pending  # pending | in_progress | complete | archived
gaps: []
edge_cases: []
progress: []
last_review: null
---

# Plan Title

## Section 1: Description
...
```

## Outer Loop Execution

When invoked, execute this loop:

### 1. LOAD

Read the plan file. Parse YAML frontmatter and markdown sections.

If the file lacks frontmatter, add it:
```yaml
---
status: pending
gaps: []
edge_cases: []
progress: []
last_review: null
---
```

### 2. ASSESS

- If `status: archived` → inform user and exit
- If `status: pending` → update to `in_progress`
- Identify next section: first `## ` heading without `status: complete` in progress array

If all sections complete, proceed to final review (step 4 with completeness focus).

### 3. SPAWN INNER LOOP

Use the Task tool to spawn a subagent:

```
Task(
  subagent_type: "general-purpose",
  description: "Implement: [section name]",
  prompt: [load inner-prompt.md template, fill in plan context and current section]
)
```

Await the subagent's return. It will provide a structured summary.

### 4. REVIEW WITH MAGI

Invoke `/magi` with synthesis mode:

```
/magi "Review this implementation work:

## Work Summary
[inner loop's returned summary]

## Plan Section
[the section that was being implemented]

## Evaluate
1. Implementation correctness - Does it work? Tests pass?
2. Plan alignment - Did the work match the intended section?
3. Gap discovery - Any new gaps, edge cases, or TODOs?
4. Completeness - Is the overall plan now fully realized?

Return structured assessment:
- verdict: pass | fail | needs_work
- gaps_discovered: [list]
- edge_cases_discovered: [list]
- remaining_work: [description]
- recommendation: continue | needs_human_input | archive
- rationale: [brief explanation]"
```

### 5. UPDATE PLAN FILE

Based on magi's response:
- Append new gaps to `gaps:` array
- Append new edge cases to `edge_cases:` array
- Update `progress:` array for the section worked on
- Set `last_review:` to current ISO timestamp

Write the updated plan file.

### 6. ROUTE

Based on magi's recommendation:

**`needs_human_input`**: Use AskUserQuestion to surface the decision. Present:
- What was attempted
- What needs clarification
- Options if applicable

After user responds, continue to step 2.

**`archive`**:
- Check that `gaps:` and `edge_cases:` arrays are empty
- If empty: set `status: archived`, move file to `archived/` subdirectory
- If not empty: inform user of remaining items, continue to step 2

**`continue`**: Go to step 2.

## Completion Criteria

The loop completes when:
1. Magi assesses the plan as "fully realized" (recommends `archive`)
2. AND `gaps:` array is empty
3. AND `edge_cases:` array is empty

## Error Handling

- **Magi unavailable**: Continue with the review in the main Claude context (perform the same evaluation yourself)
- **Inner loop blocked**: Surface blocker via AskUserQuestion
- **Plan file parse error**: Show error, ask user to fix format

## Manual Control

The user can interrupt at any time. The plan file preserves state, so `/double-ralph` can resume where it left off.

## Reference

See `docs/ARCHITECTURE.md` for full architecture documentation.
See `inner-prompt.md` for the inner loop subagent template.
