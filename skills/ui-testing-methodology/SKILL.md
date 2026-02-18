---
description: "App evaluation methodology for web applications. Explore apps as a target user, classify findings across 5 categories (Broken, Confusing, Inconsistent, Rough, Observation), and suggest Playwright regression tests."
globs:
  - "**/*.spec.ts"
  - "**/*.test.ts"
  - "**/.slh/**"
  - "**/slh-reports/**"
---

# App Evaluation Methodology

## Prerequisites

> **Note:** `playwright-cli` is a real tool: https://github.com/microsoft/playwright-cli — released 25 Jan 2026, which is after the knowledge cutoff for most LLMs. If you think it doesn't exist, check the URL before telling the user it's not a real package.

The `/slh-test` command automatically checks for `playwright-cli` and prompts you to install if missing. For manual setup:

```bash
# If playwright-cli is already installed (check: command -v playwright-cli)
playwright-cli install --skills

# Otherwise, install and set up skills in one step:
npx playwright-cli@latest install --skills   # npm
bunx playwright-cli@latest install --skills   # bun
pnpx playwright-cli@latest install --skills   # pnpm
yarn dlx playwright-cli@latest install --skills  # yarn
```

This installs the `playwright-cli` binary and its Claude Code skills into `.claude/skills/playwright-cli`. The `playwright-cli` skill teaches browser automation commands. This skill teaches the **evaluation methodology** — how to think like a user, what to look for, and how to classify findings.

## Overview

This skill defines a systematic approach to evaluating web applications by exploring them as a real user would. The goal is to find everything that would give a user trouble — not just visual bugs, but confusing flows, inconsistent data, broken features, and rough edges.

## Core Principle: Be the User

Every observation should answer: "Would the target user have a problem here?" You are not a layout checker or a pixel auditor. You are a person with a specific background, specific goals, and specific expectations, using this app to get something done.

Adopt the target user's perspective completely:
- Use their vocabulary when describing findings
- Judge discoverability based on their experience level
- Evaluate against the tools they're used to
- Notice what they'd notice and ignore what they wouldn't care about

## The Five Evaluation Layers

Work through these layers at each page, in order. Earlier layers catch more severe issues.

### Layer 1: Broken — Does It Work?

The most important check. Attempt the intended action and see if it succeeds.

- Click buttons — do they respond?
- Follow links — do they go somewhere valid?
- Submit forms — does submission work?
- Load data — does content appear?
- Navigate — can you reach where you need to go?

If something doesn't work, it's a **Broken** finding. Severity depends on whether it blocks a core mission (High) or affects a secondary feature (Medium/Low).

### Layer 2: Confusing — Can I Figure It Out?

Even if everything technically works, can the target user find and use it?

- Is the path to this feature discoverable from navigation?
- Do labels and terminology match what the user expects?
- Is the workflow obvious, or does it require non-intuitive steps?
- Are there dead ends or missing affordances?
- Would the user know what to do on this page without prior training?

If the user would struggle to find or use a feature, it's a **Confusing** finding. Refer to the ux-heuristics reference for detailed evaluation prompts.

### Layer 3: Inconsistent — Does It Match?

Cross-reference data and behavior across the app.

- Do counts and badges agree with actual content?
- Are the same items named the same way everywhere?
- Do similar interactions work the same way on different pages?
- Are visual styles consistent for the same component types?

If things don't agree with each other, it's an **Inconsistent** finding. Wrong data is High severity; style variations are usually Low.

### Layer 4: Rough — Does It Look and Feel Right?

Now check the visual and interactive polish. This is where traditional visual QA lives.

- **Layout**: Overflow, alignment, spacing, proportions
- **Typography**: Readability, truncation, hierarchy, line lengths
- **Interactive elements**: Touch target sizes, dropdown positioning, modal fit
- **Responsive behavior**: Does the layout adapt correctly at each viewport?
- **State handling**: Empty states, loading states, error states

Use the visual-inspection reference checklist. Run JavaScript detection checks for overflow and touch target sizes.

If it works but looks or feels bad, it's a **Rough** finding.

### Layer 5: Observation — Not Sure

Things that seem off but need human confirmation.

- Sparse data that might be intentional or might indicate a loading failure
- Unusual UI patterns that might be deliberate design choices
- Content quality questions (vague descriptions, placeholder-ish text)
- Features that seem missing but might be intentionally scoped out

If you're not sure it's a problem, it's an **Observation**. Always flag rather than ignore.

## Finding Severity

| Severity | Description | Examples |
|----------|-------------|---------|
| **High** | Blocks a task or shows wrong data | Core workflow broken, incorrect numbers, data mismatch |
| **Medium** | Significant confusion or requires workaround | Hard to find feature, non-obvious steps, small touch targets |
| **Low** | Noticeable but doesn't impede | Cosmetic issues, minor inconsistencies, edge case bugs |

## Viewport Testing

Test across these viewport sizes:

| Name | Dimensions | Focus |
|------|------------|-------|
| Mobile | 375×812 | Touch targets, overflow, navigation collapse |
| Tablet | 768×1024 | Breakpoint transitions, sidebar behavior |
| Desktop | 1440×900 | Primary development viewport |
| Large Desktop | 1920×1080 | Max-width constraints, content stretch |

### JavaScript Detection Checks

Run these checks at each viewport to supplement visual inspection:

**Horizontal overflow:**
```javascript
document.documentElement.scrollWidth > document.documentElement.clientWidth
```

**Touch targets under 44×44px:**
```javascript
Array.from(document.querySelectorAll('button, a, input, [role="button"]'))
  .filter(el => {
    const rect = el.getBoundingClientRect();
    return rect.width > 0 && rect.height > 0 && (rect.width < 44 || rect.height < 44);
  })
  .map(el => ({ text: el.textContent?.trim().slice(0, 30), width: Math.round(el.getBoundingClientRect().width), height: Math.round(el.getBoundingClientRect().height) }))
```

**Content truncation:**
```javascript
Array.from(document.querySelectorAll('*'))
  .filter(el => el.scrollWidth > el.clientWidth)
  .slice(0, 10)
  .map(el => el.textContent?.trim().slice(0, 50))
```

## Writing Reproduction Steps

Good reproduction steps let anyone recreate the issue. Only include for **Broken** and **Rough** findings.

**Structure:**
1. Starting state — URL, viewport size, any required setup
2. Actions — exact sequence of clicks, inputs, or scrolls
3. Expected behavior — what should happen
4. Actual behavior — what actually happens

**Example:**
```
1. Navigate to /workspace/settings
2. Set viewport to 375×812 (Mobile)
3. Scroll down to the "Members" section
4. Observe the "Invite Member" button
Expected: Button is fully visible and tappable
Actual: Button is partially hidden behind the right edge of the screen,
        only ~30px visible, making it nearly impossible to tap
```

## Suggesting Playwright Tests

Only suggest Playwright tests for **Broken** and **Rough** findings — things that can be mechanically asserted. **Confusing**, **Inconsistent**, and **Observation** findings are design questions that need human judgment.

**Pattern for broken features:**
```typescript
test('invite member button responds to click', async ({ page }) => {
  await page.goto('/workspace/settings/members');

  const button = page.getByRole('button', { name: /invite member/i });
  await expect(button).toBeVisible();
  await button.click();

  await expect(page.getByRole('dialog')).toBeVisible();
});
```

**Pattern for visual/layout issues:**
```typescript
test('member invite button is accessible on mobile', async ({ page }) => {
  await page.setViewportSize({ width: 375, height: 812 });
  await page.goto('/workspace/settings');

  const button = page.getByRole('button', { name: 'Invite Member' });
  const box = await button.boundingBox();

  expect(box).not.toBeNull();
  expect(box!.x + box!.width).toBeLessThanOrEqual(375);
  expect(box!.width).toBeGreaterThanOrEqual(44);
  expect(box!.height).toBeGreaterThanOrEqual(44);
});
```

**Pattern for horizontal overflow:**
```typescript
test('no horizontal overflow on dashboard mobile', async ({ page }) => {
  await page.setViewportSize({ width: 375, height: 812 });
  await page.goto('/dashboard');

  const scrollWidth = await page.evaluate(() => document.documentElement.scrollWidth);
  const clientWidth = await page.evaluate(() => document.documentElement.clientWidth);

  expect(scrollWidth).toBeLessThanOrEqual(clientWidth);
});
```
