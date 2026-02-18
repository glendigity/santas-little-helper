# Report Templates

Generate **two** report files: a narrative evaluation report (primary) and a page results log (reference).

## File 1: `report.md` — Evaluation Report (Primary)

This is what the user reads. Narrative structure with mission walkthroughs and classified findings.

```markdown
# App Evaluation Report

| | |
|---|---|
| **URL** | {app-url} |
| **Date** | {date} |
| **Persona** | {persona name} — {one-line summary from profile, e.g., "Ops Lead — Corporate employee, used to Confluence/Jira, expects enterprise polish"} |
| **Missions Tested** | {count} of {total} |
| **Viewports** | {viewport-list} |
| **Pages Visited** | {count} |

## Executive Summary

{2-4 sentences. Overall verdict: is this app usable for the target user? What's the biggest concern? What's the strongest positive? End with a clear recommendation.}

## By the Numbers

| | High | Medium | Low | Total |
|---|---|---|---|---|
| Broken | {n} | {n} | {n} | {n} |
| Confusing | {n} | {n} | {n} | {n} |
| Inconsistent | {n} | {n} | {n} | {n} |
| Rough | {n} | {n} | {n} | {n} |
| Observation | {n} | {n} | {n} | {n} |
| **Total** | **{n}** | **{n}** | **{n}** | **{n}** |

## Mission Walkthroughs

{Repeat for each mission attempted}

### Mission {n}: {Mission goal from profile}

**Verdict: {Accomplished | Partially Accomplished | Failed}**

{Narrative walkthrough written in first person as the target user:

"I wanted to find out who reports to me. Starting from the main navigation, I looked for something like 'My Team' or 'Org Chart.' The sidebar has 'People' which seemed promising. Clicking it showed a list of all employees — not just my team. I couldn't find a filter for 'reports to me' so I tried the search bar..."

Describe:
- The path you took and why
- Where you hesitated, guessed, or backtracked
- What worked well (give credit where due)
- Where you got stuck or confused
- Whether you accomplished the mission

Reference specific findings by number: "This is where I hit Finding #3 — the filter dropdown extends past the viewport on mobile."}

**Findings from this mission:** #{n}, #{n}, #{n}

---

## Findings

{All findings, ordered by severity (High first, then Medium, then Low). Within the same severity, order by category: Broken > Confusing > Inconsistent > Rough > Observation.}

### Finding #{n}: {Short title} — {Category} / {Severity}

| | |
|---|---|
| **Page** | {page-path} |
| **Viewport** | {viewport-name} ({width}x{height}) or "All viewports" |

{Description through the target user's lens. "I couldn't figure out how to..." not "The affordance for the action lacks sufficient visual prominence." Be specific about what's wrong and why it matters.}

**Screenshot**

![{alt-text}]({absolute-screenshots-dir}/{filename}.png)

{For Broken and Rough findings only:}

**Steps to Reproduce**

1. Navigate to `{page-path}`
2. Set viewport to {width}x{height}
3. {Specific action}
4. Observe {specific element}

**Expected:** {What should happen}
**Actual:** {What actually happens}

**Suggested Playwright Test**

```typescript
test('{descriptive test name}', async ({ page }) => {
  await page.setViewportSize({ width: {width}, height: {height} });
  await page.goto('{page-path}');

  // {assertion}
});
```

---

## Cross-Page Consistency

{Include when inconsistencies were found across pages. Omit if none.}

### {Element type}: {What varies}

| Page | Value |
|------|-------|
| {page-path-1} | {specific value} |
| {page-path-2} | {specific value} |

**Category:** Inconsistent — **Severity:** {Low | Medium}

{Brief explanation of why this matters.}

---

## What Works Well

{Every report must have this section. 2-4 positives to preserve.}

1. **{Positive title}** — {Brief description of what the app does well. E.g., "Navigation is clear and consistent — the sidebar labels matched my expectations for every mission."}
2. **{Positive title}** — {Description}
3. **{Positive title}** — {Description}

---

{If no findings at all, replace Findings + Consistency sections with:}

## No Issues Found

All {count} pages passed evaluation across all tested viewports. The target user should be able to accomplish all tested missions without difficulty.
```

## File 2: `page-results.md` — Full Page Log (Reference)

Detailed record of every page + viewport tested.

```markdown
# Page Results Log

| | |
|---|---|
| **URL** | {app-url} |
| **Date** | {date} |
| **Viewports Tested** | {viewport-list} |
| **Pages Visited** | {count} |

{Repeat for each page visited}

## {page-path}

**Context:** {Why this page was visited — which mission, or curiosity expansion}

{Repeat for each viewport tested}

### {Viewport Name} ({width}x{height}) — {PASS | FAIL (Finding #{n})}

![{page-slug}-{viewport-name}]({absolute-screenshots-dir}/{page-slug}-{viewport-name}-{width}x{height}.png)

{Brief note if FAIL — otherwise just the screenshot}

---

## Test Configuration

- **Persona**: {persona name} — {one-line summary}
- **Missions Tested**: {list}
- **Viewports**: {list with dimensions}
- **Pages visited**: {count}
- **Screenshots captured**: {count}
- **Test duration**: Approximately {duration}
```

## Screenshot Paths

All screenshot references in `report.md` and `page-results.md` must use **absolute paths** so the images render correctly when the markdown is converted to PDF (e.g., with pandoc, mdpdf, or a VS Code extension). Use the full absolute path to the screenshots directory, not relative paths.

Example: `![alt text](/absolute/path/to/slh-reports/2026-02-18_14-30_localhost-3000/screenshots/dashboard-mobile-375x812.png)`

## Screenshot Naming Convention

Format: `{page-slug}-{viewport-name}-{width}x{height}[-{state}].png`

Examples:
- `dashboard-mobile-375x812.png`
- `settings-members-desktop-1440x900.png`
- `settings-members-mobile-375x812-invite-modal-open.png`
- `00-login-success.png` (special case for login)

## Example Finding Entry

```markdown
### Finding #1: Can't find team member list — Confusing / High

| | |
|---|---|
| **Page** | /people |
| **Viewport** | Desktop (1440x900) |

I wanted to see who reports to me. The sidebar has "People" which shows everyone in the organization — but there's no filter for "my team" or "reports to me." I tried the search bar but it only searches by name, not by reporting relationship. The org chart view exists but is buried under a submenu I found by accident.

A corporate user coming from tools like Workday or BambooHR would expect a prominent "My Team" view. Having to browse the entire organization to find your direct reports is a significant gap.

**Screenshot**

![People page with no team filter]({absolute-screenshots-dir}/people-desktop-1440x900.png)
```

```markdown
### Finding #2: Invite button hidden on mobile — Rough / Medium

| | |
|---|---|
| **Page** | /workspace/acme/settings/members |
| **Viewport** | Mobile (375x812) |

The "Invite Member" button extends past the right edge of the viewport. Only about 30px is visible, making it nearly impossible to tap. I only found it because I happened to scroll right.

**Screenshot**

![Invite button overflow on mobile]({absolute-screenshots-dir}/settings-members-mobile-375x812.png)

**Steps to Reproduce**

1. Navigate to `/workspace/acme/settings/members`
2. Set viewport to 375x812
3. Look at the header area containing the "Invite Member" button
4. Observe the button extends past the right edge

**Expected:** Button is fully visible and tappable within the viewport.
**Actual:** Button overflows viewport, only ~30px visible.

**Suggested Playwright Test**

```typescript
test('invite member button is fully visible on mobile', async ({ page }) => {
  await page.setViewportSize({ width: 375, height: 812 });
  await page.goto('/workspace/acme/settings/members');

  const button = page.getByRole('button', { name: /invite member/i });
  await expect(button).toBeVisible();

  const box = await button.boundingBox();
  expect(box).not.toBeNull();
  expect(box!.x + box!.width).toBeLessThanOrEqual(375);
});
```
```
