---
description: "Re-test previously reported findings to verify which are fixed, still present, or changed. Works with both old-format (issues/content questions) and new-format (5-category findings) reports."
argument-hint: "[url] [username] [password]"
---

# SLH Retest Command

You are running a regression check. Your job is to take a previous evaluation report, re-test each finding, and produce a clear verdict: FIXED, STILL PRESENT, or CHANGED.

## Phase 0: Prerequisites — playwright-cli Required

**This plugin requires `playwright-cli` for browser automation. Do NOT use any Playwright MCP server tools (`mcp__*playwright*` or `browser_snapshot`, `browser_click`, etc.). All browser interaction MUST go through `playwright-cli` bash commands.**

Check that `playwright-cli` is installed and its skills are available:

```bash
command -v playwright-cli && test -d .claude/skills/playwright-cli && echo "READY" || echo "MISSING"
```

- If **"READY"** — proceed to Phase 1.
- If **"MISSING"** — stop and tell the user:

> "SLH requires `playwright-cli` for browser automation. Please install it by running:
>
> ```
> npx playwright-cli@latest install --skills
> ```
>
> This installs the CLI and its Claude Code skills into `.claude/skills/playwright-cli`. Once installed, re-run `/slh-retest`."

**Do not proceed without playwright-cli.** Do not fall back to MCP tools, Chrome DevTools, or any other browser automation method.

## Phase 1: Collect Inputs

Parse any arguments provided: `$ARGUMENTS`

The format is `[url] [username] [password]`. Any missing values must be collected.

### 1a. Find Previous Reports

Scan `./slh-reports/` for run directories (format: `{YYYY-MM-DD_HH-MM}_{hostname}/`). List any that contain a `report.md` file, sorted newest first.

Also check for saved credentials at `./slh-reports/credentials.json`. If found, pre-fill the URL and credentials.

Present the user with the available reports using `AskUserQuestion`:

1. **Report to re-test** — list the most recent 4 run directories with their date and hostname. Show the finding/issue count from each report's summary if possible.
2. **App URL** — pre-fill from saved credentials or the report header if available, but let the user override.
3. **Credentials** — pre-fill from saved credentials if available. Otherwise collect username and password. Offer "No login required." If collecting new credentials, tell the user: "Credentials saved to `slh-reports/credentials.json` (gitignored)."

If new credentials were provided, update `./slh-reports/credentials.json`.

### 1b. Parse the Report

Read the selected `report.md`. Detect the report format:

**New format** (v2 — has "By the Numbers" matrix and Finding #{n} entries):
- Extract all findings with: number, title, category, severity, page path, viewport, description
- For Broken/Rough findings: extract steps to reproduce and Playwright test suggestions
- Extract any Observation findings (these get re-checked too)

**Old format** (v1 — has Issue #{n} and Content Questions):
- Extract all issues with: number, title, severity, page path, viewport, steps to reproduce, Playwright test
- Extract content questions (Q1, Q2, etc.)
- Map to new categories for the retest report:
  - CRITICAL issues → Broken/High
  - MODERATE issues → Rough/Medium
  - MINOR issues → Rough/Low
  - Content Questions → Observation/Medium

Tell the user: "Found {n} findings to re-test from {date} report."

### 1c. Set Up Output Directory

Create a new run directory: `./slh-reports/{YYYY-MM-DD_HH-MM}_{hostname}/`

Create a `screenshots/` subdirectory.

### 1d. Load App Knowledge

Use the Glob tool with pattern `slh-reports/app-knowledge/{hostname}.md` (substituting the actual hostname). If no match, also try `slh-reports/app-knowledge/*.md` and check filenames. If found, read it with the Read tool.

## Phase 2: Login

Skip if "No login required" was selected.

Follow the same login steps as `/slh-test`.

## Phase 3: Re-test Each Finding

For each finding from the original report:

### 3a. Reproduce

Use `playwright-cli` for browser automation (run `playwright-cli --help` to see commands):

1. Navigate to the finding's page path
2. Set viewport to the specified dimensions
3. Wait for content to load
4. Follow the steps to reproduce from the original report (if available)
5. For findings without explicit repro steps (Confusing, Inconsistent, Observation), navigate to the page and re-evaluate the described condition

### 3b. Evaluate

Take a screenshot and save as `{run-directory}/screenshots/retest-{n}-{page-slug}-{viewport}.png`

Determine the verdict:

**FIXED** — The finding no longer applies.
- The broken feature now works
- The confusing flow has been clarified
- The inconsistency has been resolved
- The rough visual has been polished
- The observation is no longer relevant (data changed, feature added, etc.)

**STILL PRESENT** — The finding persists as originally described.
- Same symptoms, same conditions
- Screenshot shows the same problem

**CHANGED** — The area has changed but the original finding doesn't cleanly apply.
- The page has been redesigned or the element has been removed/replaced
- The finding might be fixed but the context is so different that the original doesn't apply
- A new, different issue may have appeared in the same area

For each verdict, record:
- The original finding number, title, category, and severity
- The verdict
- A brief explanation of what you observed
- Screenshot filename
- If CHANGED: describe what's different and whether a new finding exists

### 3c. Re-run Assertions (when applicable)

For Broken and Rough findings that included Playwright test suggestions with specific assertions, translate the assertion into a JavaScript check and run it:

```javascript
// Example: re-checking touch target size
() => {
  const el = document.querySelector('button[aria-label="Expand"]');
  if (!el) return { found: false };
  const rect = el.getBoundingClientRect();
  return { found: true, width: Math.round(rect.width), height: Math.round(rect.height), passes: rect.width >= 44 && rect.height >= 44 };
}
```

## Phase 4: Retest Report

Generate the report at `{run-directory}/retest-report.md`:

```markdown
# Retest Report

| | |
|---|---|
| **Original Report** | {original run directory name} |
| **Original Date** | {date from original report} |
| **Retest Date** | {today} |
| **URL** | {app-url} |

## Summary

| Verdict | Count |
|---------|-------|
| FIXED | {n} |
| STILL PRESENT | {n} |
| CHANGED | {n} |
| **Total** | **{n}** |

{1-2 sentence overview — e.g., "4 of 6 findings are fixed. 2 medium-severity findings remain (1 Confusing, 1 Rough)."}

## Still Present — Action Needed

{List only STILL PRESENT findings here, ordered by severity. This is the quick-reference for what still needs work. Omit this section if all findings are fixed.}

- **#{n}: {title}** — {Category} / {Severity} — {one-line description}
- **#{n}: {title}** — {Category} / {Severity} — {one-line description}
```

Then for each finding:

```markdown
## Finding #{n}: {Original title} — {Category} / {Severity}

**Verdict: {FIXED | STILL PRESENT | CHANGED}**

**Original:** {1-line summary of the original finding}

**Now:** {What you observed during retest}

| | |
|---|---|
| **Page** | {page-path} |
| **Viewport** | {viewport-name} ({width}x{height}) |
| **JS Check** | {PASS / FAIL / N/A} — {brief detail if applicable} |

**Screenshot**

![Retest finding {n}](screenshots/retest-{n}-{page-slug}-{viewport}.png)
```

### Present Results

After writing the report, tell the user:
- The run directory path
- How many findings are FIXED vs STILL PRESENT vs CHANGED
- Highlight any STILL PRESENT items by severity (High first)
- If all findings are fixed, celebrate briefly

## Phase 5: Feedback & Learning

Same as `/slh-test` — offer to update the app knowledge file with any corrections.

If the user confirms findings are fixed, record in the knowledge file: "Finding with X on Y page was fixed as of {date}" for future reference.

## Error Handling

- If a page path from the original report no longer exists (404), mark the finding as CHANGED with a note
- If the viewport or element can't be found, note it and continue
- Never stop because one finding can't be re-tested — always continue to the next
