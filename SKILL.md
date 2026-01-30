---
name: double-ralph
description: Iterative implementation loop with review checkpoints. Use for multi-step tasks that benefit from chunked execution and verification. Adapts to project-specific conventions via .ralph.md guidance file.
---

# Double Ralph: Iterative Implementation Loop

Execute work through iterative cycles with review checkpoints between chunks.

## Invocation

```
/double-ralph [state-file]
```

Examples:
- `/double-ralph plans/my-feature.md` - Execute a plan file
- `/double-ralph` - Auto-detect state file per project guidance

## Project Guidance

Double-ralph adapts to project-specific conventions via a `.ralph.md` file.

### Finding .ralph.md

Search in order:
1. **Git repository root**: `git rev-parse --show-toplevel` then check for `.ralph.md`
2. **Walk up from current directory**: Check each parent until `.ralph.md` found or root reached
3. **State file's directory**: If state file provided, check its directory

```bash
# Get git project root
git rev-parse --show-toplevel 2>/dev/null
```

### If No .ralph.md Found

Use AskUserQuestion to offer creating one:

"No `.ralph.md` found for this project. Would you like to create one?"
- **Yes, help me create it** - Ask design questions, generate .ralph.md
- **Use defaults** - Continue with plan-file conventions
- **Skip for now** - Continue without guidance

See `examples/` in this skill's directory for templates and the README for guidance on crafting .ralph.md files.

## The Loop

### 1. LOAD

1. Find and load `.ralph.md` if present (provides project-specific guidance)
2. Read the state file
3. Parse state according to guidance (or use default plan format)

**Default format** (when no .ralph.md):
```yaml
---
status: pending  # pending | in_progress | complete | archived
gaps: []
edge_cases: []
progress: []
last_review: null
---

# Title

## Section 1
...
```

If file lacks frontmatter, add defaults.

### 2. ASSESS

- Check completion status → if complete/archived, inform user and exit
- Update status to `in_progress` if pending
- Identify work units per guidance:
  - **Default**: `## ` headings not in progress array
  - **Per guidance**: acceptance criteria, issues, custom sections
- Group related units if guidance suggests logical groupings
- If all units complete → proceed to final review

### 3. SPAWN INNER LOOP

Use the Task tool to spawn a subagent:

```
Task(
  subagent_type: "general-purpose",
  description: "Implement: [work unit summary]",
  prompt: [see inner-prompt.md, include project guidance if present]
)
```

Inner loop works until:
- Work unit complete
- Blocked (needs decision, unclear requirement)
- ~15-20 turns (context management)
- Context feels heavy

Returns structured summary (see inner-prompt.md for format).

### 4. REVIEW

Review method per `.ralph.md` guidance. Default: magi synthesis.

```
/magi "Review this implementation work:

## Work Summary
[inner loop's returned summary]

## Work Unit
[what was being implemented]

## Evaluate
1. Implementation correctness - Does it work? Tests pass?
2. Alignment - Did the work match the intended unit?
3. Gap discovery - Any new gaps, edge cases, or TODOs?
4. Completeness - Is the overall work fully realized?

Return structured assessment:
- verdict: pass | fail | needs_work
- gaps_discovered: [list]
- edge_cases_discovered: [list]
- remaining_work: [description]
- recommendation: continue | needs_human_input | archive
- rationale: [brief explanation]"
```

**Fallback**: If magi unavailable, perform the review yourself.

### 5. UPDATE STATE

Update the state file per guidance:
- **Default**: Update frontmatter arrays (gaps, edge_cases, progress)
- **Per guidance**: Check off criteria, append to sections, etc.

Set review timestamp.

### 6. ROUTE

Based on review recommendation:

**`continue`**:
- Add any `gaps_discovered` or `edge_cases_discovered` from review to state file
- Go to step 2

Note: This naturally handles "conditional pass" - issues are tracked and prevent archiving until resolved.

**`needs_human_input`**:
- Use AskUserQuestion to surface the decision
- Present: what was attempted, what needs clarification, options
- After response → step 2

**`archive`** (or equivalent completion):
- Verify no remaining gaps/issues per guidance
- Mark complete and move/archive per guidance
- If issues remain → inform user, continue to step 2

## Completion Criteria

The loop completes when:
1. Review assesses work as "fully realized"
2. No remaining gaps or blockers
3. Per-guidance completion signals sent (if applicable)

## Error Handling

| Scenario | Action |
|----------|--------|
| Magi unavailable | Self-review (continue functioning) |
| Inner loop blocked | Surface via AskUserQuestion |
| State file parse error | Show error, ask user to fix |
| No .ralph.md | Offer to create or use defaults |

## Manual Control

Interrupt anytime. State file preserves progress. Resume with `/double-ralph [state-file]`.

## Reference

- `inner-prompt.md` - Inner loop subagent template
- `examples/` - Example .ralph.md files for different project types
- `examples/README.md` - Guide for crafting .ralph.md files
- `docs/ARCHITECTURE.md` - Full architecture documentation
