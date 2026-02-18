---
description: "Define or edit target user personas and mission lists for an app. Creates a persistent profile used by /slh-test."
argument-hint: "[url]"
---

# SLH Profile Command

You are helping the user define **who** uses their app and **what those users need to accomplish**. This profile drives all future `/slh-test` sessions — the agent will explore the app through a selected persona's eyes.

An app can have **multiple personas** — different user types who see the app differently. Each persona has its own description and mission list. During `/slh-test`, the user picks which persona to test as.

## Phase 1: Detect Existing Profiles

Check if any profile files exist at `./slh-reports/app-profile/`. List any `.md` files found there.

If a URL was provided as an argument (`$ARGUMENTS`), extract the hostname and check if a profile already exists for it.

### If profiles exist

Present the user with a choice using `AskUserQuestion`:

1. **Update existing profile** — list each existing profile by hostname with a brief summary (persona count + total mission count). Show up to 4 profiles.
2. **Create new profile** — start fresh for a different app

If they choose an existing profile, jump to **Phase 4: Edit Existing Profile**.

### If no profiles exist (or user chose "new")

Continue to Phase 2.

## Phase 2: Define Personas

First, collect the app URL if not already provided. Extract the hostname for the profile filename.

Then have a conversation to build the persona list. Use the personas reference as starter archetypes.

### Ask about user types

Start with: **"Who uses this app? Are there different types of users with different needs?"**

Common patterns:
- Some apps have one primary user type → create one persona
- Most apps have 2-4 distinct user types → create multiple personas
- If the user says "all of these" when shown archetypes, create a persona for each relevant one

### For each persona

Build through conversation (not all questions at once — build on their answers):

1. **Give it a name** — a short label like "Ops Lead", "Team Manager", "Executive", "New Employee". This is how the persona is referenced in `/slh-test`.

2. **"What's their role and background?"**
   - Job title, department, seniority level
   - What's their relationship to the app? Daily user, occasional checker, admin?

3. **"What's their experience level?"**
   - Tech-savvy power user or needs things obvious?
   - What other tools are they used to? (frames their expectations)

4. **"Any other context?"**
   - Age range, work environment, device preferences
   - Patience level, what frustrates them

Synthesize each into a paragraph. Show them all to the user:

> "Here are the personas I'll use when testing. Adjust anything?"

Refine through conversation until they're happy.

## Phase 3: Build Mission Lists

For each persona, build a list of tasks they need to accomplish.

Ask: **"What does {persona name} need to accomplish in the app?"**

Guide them:
- Start with the most critical workflow — the thing they'd do on day one
- Then the regular tasks — what they do daily/weekly
- Then the occasional tasks — setup, configuration, edge cases
- Each mission should be a concrete goal, not a page name. "Find out who reports to you" not "View the org chart"
- Some missions may be shared across personas (e.g., "Log in and orient yourself") — that's fine, list them under each persona that needs them

For each mission the user suggests:
- Confirm you understand the goal
- Suggest refinements if the mission is too vague ("Browse the dashboard" → "Check if any of your projects are behind schedule")
- Suggest related missions they might not have thought of

Also proactively suggest missions based on common patterns:
- "First-time orientation — figure out what this app does and where things are"
- "Find something specific using search"
- "Complete a common creation workflow"
- "Review and act on notifications or pending items"

Build a numbered list per persona, ranked by priority (most important first). Show the full list:

> "Here are the missions for each persona. Want to add, remove, reword, or reorder anything?"

Refine through conversation until they're happy.

## Phase 4: Edit Existing Profile

Read the existing profile file. Display all personas with their descriptions and mission lists.

Ask: **"What would you like to change? You can add/remove personas, edit persona descriptions, add/remove/reword missions, or reorder priorities."**

Handle edits through conversation:
- **Add persona**: Go through the Phase 2 questions for the new persona, then Phase 3 for its missions
- **Remove persona**: Confirm which to remove
- **Edit persona description**: Show the updated paragraph for confirmation
- **Add missions**: Ask which persona and where in the priority order
- **Remove missions**: Confirm which to remove, renumber
- **Reword missions**: Show the before/after
- **Reorder**: Show the new order for confirmation
- **Move mission between personas**: Remove from one, add to another

Multiple edits are fine in one conversation. Show the final state before saving.

## Phase 5: Save Profile

Write the profile to `./slh-reports/app-profile/{hostname}.md`.

The hostname is derived from the URL: replace `:` with `-`, `/` with nothing, keep dots and hyphens. For example:
- `http://localhost:5173` → `localhost-5173`
- `https://app.example.com` → `app.example.com`

Create the `./slh-reports/app-profile/` directory if it doesn't exist.

### Profile File Format

```markdown
# App Profile: {hostname}

> Last updated: {YYYY-MM-DD}
> Sessions: {count}
> Personas: {count}

## Persona: {Name}

{Paragraph describing this user type — their role, background, experience level,
what tools they're used to, what they expect, key context. Written in third
person — "This user is a..." — so the testing agent can adopt this lens.}

### Missions

1. {Most important task} — {brief context if needed}
2. {Second task} — {context}
3. {Third task}
...

## Persona: {Name}

{Description paragraph}

### Missions

1. {Task}
2. {Task}
...
```

For new profiles, set Sessions to 0. For existing profiles, preserve the session count (it's incremented by `/slh-test`).

After saving, confirm:

> "Profile saved for {hostname} with {n} personas and {total} missions.
>
> **Next step**: Type `/clear` to free up context, then run `/slh-test` to start testing. The test session is context-heavy — starting fresh avoids running out of room mid-run."
