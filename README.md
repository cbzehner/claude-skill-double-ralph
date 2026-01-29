# Double Ralph

Nested agent loops for executing implementation plans. What could possibly go wrong?

> *"More Ralphs is always better, rite?"*
>
> An outer Ralph orchestrates. Inner Ralphs implement. Magi reviews.
> It's Ralphs all the way down.

```
        ┌──────────────────────────┐
        │      OUTER RALPH         │
        │    (the orchestrator)    │
        │            │             │
        │            ▼             │
        │   ┌────────────────┐     │
        │   │  INNER RALPH   │     │
        │   │  (does work)   │     │
        │   └────────────────┘     │
        │            │             │
        │            ▼             │
        │      /magi review        │
        │            │             │
        │            ▼             │
        │    repeat until done     │
        │    (or until chaos)      │
        └──────────────────────────┘
```

## Why?

Long tasks lose context. Claude forgets what it was doing 47 tool calls ago. Double Ralph fixes this by:

1. **Chunking work** - Inner Ralphs work on one section at a time
2. **Reviewing progress** - Magi checks each chunk before continuing
3. **Persisting state** - Everything is saved to the plan file, so you can stop and resume

It's like having a responsible adult supervise the hyperactive code monkeys.

## Prerequisites

Works best with [magi](https://github.com/cbzehner/claude-skill-magi) installed for reviews. Falls back to self-review if unavailable (Ralph reviewing Ralph's work—*totally objective*).

## Installation

### From Marketplace

```bash
/plugin marketplace add cbzehner/claude-skill-double-ralph
/plugin install double-ralph@cbzehner
```

### Manual Installation

```bash
cd ~/.claude/skills/
git clone https://github.com/cbzehner/claude-skill-double-ralph.git double-ralph
```

## Usage

```
/double-ralph <plan-file>
```

Point it at a plan file and watch the Ralphs go:

```
You: /double-ralph plans/auth-system.md

Claude: [Outer Ralph reads the plan]
        [Spawns Inner Ralph for Section 1]
        [Inner Ralph builds stuff, runs tests]
        [Inner Ralph returns: "I did things!"]
        [Magi reviews: "The things are acceptable."]
        [Outer Ralph updates plan, moves to Section 2]
        [Repeat until victory or /triple-ralph]
```

## Plan File Format

Plans use YAML frontmatter to track state:

```markdown
---
status: pending  # pending | in_progress | complete | archived
gaps: []         # things we discovered we need
edge_cases: []   # things that might explode
progress: []     # sections completed
last_review: null
---

# My Feature Plan

## Section 1: The Setup
What to build...

## Section 2: The Hard Part
The actual work...

## Section 3: The Tests
Proof it works...
```

Frontmatter gets added automatically if missing. Double Ralph is helpful like that.

## The Loop

```
┌─────────────────────────────────────────────────────────┐
│                     OUTER RALPH                         │
│                                                         │
│  1. LOAD    → Read plan, parse frontmatter              │
│  2. ASSESS  → Find next incomplete section              │
│  3. SPAWN   → Inner Ralph implements it                 │
│  4. REVIEW  → Magi checks the work                      │
│  5. UPDATE  → Save progress, gaps, edge cases           │
│  6. ROUTE   → Continue | Ask human | Archive            │
│                                                         │
│  Inner Ralph exits when:                                │
│  • Section complete                                     │
│  • Hit a blocker                                        │
│  • ~15-20 turns (context getting heavy)                 │
│  • Existential crisis                                   │
└─────────────────────────────────────────────────────────┘
```

## Completion

The plan gets archived when:
1. Magi says "this is done"
2. AND no gaps remain
3. AND no edge cases remain

Until then, the Ralphs keep Ralphing.

## Error Handling

| Problem | Solution |
|---------|----------|
| Magi unavailable | Ralph reviews Ralph (it's fine) |
| Inner Ralph blocked | Asks you for help |
| Plan file broken | Shows error, asks you to fix it |
| Triple Ralph attempted | *inconceivable* |

## Files

```
double-ralph/
├── SKILL.md              # The actual skill
├── inner-prompt.md       # Template for Inner Ralph
├── README.md             # You are here
├── LICENSE               # MIT (Ralphs are free)
├── docs/
│   └── ARCHITECTURE.md   # The serious documentation
├── examples/
│   └── example-plan.md   # A sample plan
└── plans/
    └── archived/         # Where completed plans go to rest
```

## Manual Override

Interrupt anytime. State is saved to the plan file. Come back later and `/double-ralph` picks up where it left off. The Ralphs are patient.

## License

MIT

---

*"What could go wrong?"* — Everyone, right before finding out
