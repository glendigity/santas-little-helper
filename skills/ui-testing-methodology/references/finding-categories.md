# Finding Categories

Every finding from a test session is classified by **category** (what kind of problem) and **severity** (how bad it is). This replaces the old "visual issues" + "content questions" split with a unified system.

## Categories

### Broken

**The thing doesn't work.** A user attempting this action would fail.

Examples:
- Button does nothing when clicked
- Link goes to a 404 page
- Form can't be submitted (validation error with no way to fix it, submit button disabled for no reason)
- Page crashes or shows an error screen
- Data fails to load (infinite spinner, empty page that should have content)
- Navigation item leads to wrong page
- Feature is completely missing at a specific viewport

**Key question**: "Can the user complete this action?" If no → Broken.

### Confusing

**It works, but the user can't figure out how.** The functionality exists but discoverability, labeling, or flow makes it hard to use.

Examples:
- "I couldn't find how to invite someone" — the button exists but is buried or poorly labeled
- Terminology doesn't match what the user would expect ("Initiatives" when they'd look for "Projects")
- Workflow requires non-obvious steps (need to save before you can add members, but nothing tells you that)
- Page purpose is unclear — you land on it and don't know what to do
- Important information is hidden behind a click with no indication it's there
- Navigation hierarchy doesn't match the user's mental model

**Key question**: "Would the target user figure this out without help?" If unlikely → Confusing.

### Inconsistent

**Data or behavior doesn't match across the app.** Something says one thing here and another thing there.

Examples:
- Dashboard says "5 projects" but the projects list shows 4
- An asset has one title in a list and a different title on its detail page
- Same action works differently on different pages (click opens detail on one list, does nothing on another)
- Same concept has different names in different places ("Members" vs "Users" vs "People")
- Counts, badges, or summary numbers disagree with the actual content
- Visual style varies for the same component type across pages

**Key question**: "Do these two things agree with each other?" If no → Inconsistent.

### Rough

**It works and is findable, but looks or feels bad.** The user can accomplish the task but the experience is degraded.

Examples:
- Layout overflow or horizontal scrolling on mobile
- Touch targets too small to tap comfortably
- Text truncated without any way to see the full content
- Modal or dropdown extends past the viewport edge
- Images stretched or distorted
- Broken responsive layout that's still technically usable
- Significant spacing or alignment issues
- Missing loading states (content pops in jarringly)
- Poor contrast or unreadable text

**Key question**: "Would a user notice this and think 'that's not right'?" If yes → Rough.

### Observation

**Something seems off but you're not sure.** Needs human input to determine if it's a problem.

Examples:
- Page is very sparse — maybe the data is genuinely thin, or maybe something isn't loading
- No empty state message, but the list might legitimately be empty
- A number seems too high or too low but could be correct
- Feature seems to be missing but might be intentionally scoped out
- UI pattern is unusual but might be a deliberate design choice
- Content quality seems low (vague descriptions, placeholder-ish text) but might be real user data

**Key question**: "I'm not sure if this is wrong — I need a human to tell me." → Observation.

## Severity

Each finding also gets a severity level:

### High

**Blocks a task or presents wrong data.**

- User cannot complete a core mission
- Data shown is incorrect (wrong numbers, wrong names, mismatched counts)
- Critical workflow is broken
- Security-relevant issue (sensitive data exposed, permissions not enforced)

### Medium

**Significant confusion or requires a workaround.**

- User can eventually accomplish the task but with difficulty
- Important information is hard to find or understand
- Workaround exists but is non-obvious
- Touch targets too small on mobile (functional but frustrating)
- Meaningful visual issues that affect usability

### Low

**Noticeable but doesn't impede the user.**

- Minor cosmetic issues (spacing, alignment)
- Subtle inconsistencies that don't cause confusion
- Nice-to-have improvements
- Edge case visual bugs (specific viewport + specific content length)

## Classification Guide

Use this matrix to classify findings:

| | High | Medium | Low |
|---|---|---|---|
| **Broken** | Core workflow blocked | Secondary feature broken | Edge case failure |
| **Confusing** | Can't find core feature | Non-obvious but findable | Slightly unclear label |
| **Inconsistent** | Wrong data displayed | Counts disagree | Minor style variation |
| **Rough** | Content inaccessible | Significant UX degradation | Cosmetic imperfection |
| **Observation** | Possibly wrong data | Uncertain behavior | Minor question |

## Mapping to Playwright Tests

Only **Broken** and **Rough** findings get Playwright test suggestions — these are things that can be mechanically asserted:

- **Broken**: Assert that a button is clickable, a page loads without errors, a form submits successfully
- **Rough**: Assert viewport overflow, touch target sizes, element visibility, layout dimensions

**Confusing**, **Inconsistent**, and **Observation** findings are design questions that require human judgment — don't suggest automated tests for these.
