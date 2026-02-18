# App Knowledge Guide

Per-app knowledge files store things you've learned from the user about how a specific app works. Think of it as training notes from a senior tester — things that aren't obvious from looking at the UI but that you need to know to avoid false positives and test accurately.

## File Location

Knowledge files are stored in the committed `.slh/` directory:

```
.slh/
├── profiles/
│   ├── app.example.com.md
│   └── localhost-5173.md
├── knowledge/
│   ├── app.example.com.md
│   ├── localhost-5173.md
│   └── staging.example.com.md
├── orientation/
│   └── {hostname}--{persona-slug}.md
└── config.yml
```

File is named by the app's hostname (with colons replaced by hyphens): `{hostname}.md`

## File Structure

```markdown
# App Knowledge: {hostname}

> Last updated: {date}
> Sessions: {count}

## Navigation & Layout

- {Things about how the nav works, sidebar behavior, expected layout patterns}

## Known Patterns

- {Things that look like bugs but aren't — intentional design decisions}

## Terminology

- {App-specific terms, what they mean, how they map to UI elements}

## Interactive Behavior

- {How forms work, expected modal flows, drag-and-drop patterns}

## Data & States

- {What empty states look like, expected loading patterns, how errors display}

## Auth & Permissions

- {Login quirks, role-based UI differences, session behavior}

## UX Patterns

- {Intentional UX decisions that might seem like issues}
- {Things like "sidebar collapses to icons on mobile by design" or "empty states don't have messages for X, that's known"}
- {Design system conventions: "all modals use side panels", "forms auto-save on blur", etc.}

## Known Complexity

- {Areas of the app that are genuinely complex and shouldn't be over-simplified}
- {Features that require domain knowledge to understand — not confusing, just specialized}
- {Workflows that look long but are intentionally multi-step for safety/accuracy}
```

## What to Record

Good entries explain the **what** and **why** so you can apply the knowledge correctly:

```markdown
## Known Patterns

- The sidebar collapses to icons-only below 1024px — this is intentional, not a bug.
  The icons have tooltips on hover for labels.
- Asset cards show a colored left border based on type. No border = "uncategorized",
  which is valid.
- The "Suggest" button opens a side panel, not a modal. The main content shifts left.
  This is by design for side-by-side editing.

## UX Patterns

- Empty states in data tables intentionally show no message — just the empty table.
  This is a known design decision, not a missing empty state.
- Navigation uses "Assets" as the umbrella term for all organizational items
  (people, teams, projects). This is domain-specific, not confusing terminology.

## Known Complexity

- The suggestion workflow (suggest → review → apply/reject) is intentionally multi-step.
  It's not unnecessarily complex — it's a review gate for data quality.
- Relationship types between assets have temporal properties (start/end dates).
  A relationship showing "past" is correct for ended relationships, not a data error.
```

Bad entries are too vague to be useful:

```markdown
- The sidebar is fine
- Don't flag the colors
```

## When to Write

Only update the knowledge file during a **feedback conversation** with the user — not during testing itself. The flow is:

1. Run the test, produce the report
2. User reviews and gives feedback: "Finding #3 isn't a bug, that's how X works because..."
3. Record the learning in the knowledge file
4. Next test run reads the file and avoids the same false positive

## When to Read

- At the **start** of every test session for that app
- Before **classifying** each potential finding — check if it matches a known pattern
- When **writing the report** — reference known patterns to explain why something was not flagged
