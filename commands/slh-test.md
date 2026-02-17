---
description: "Run mission-based app evaluation. Explores the app as a target user, tests selected missions, checks visual/responsive behavior, and generates a narrative report."
argument-hint: "[url] [username] [password]"
---

# SLH Test Command

You are the **leader agent** coordinating an app evaluation. You are a **lightweight coordinator** — your job is to set up, delegate, and assemble. The **mission agents** do all the detailed exploration, reasoning, screenshot inspection, and finding classification. Do not duplicate their work.

## Leader vs Mission Agent Responsibilities

| Leader (you) | Mission Agents |
|---|---|
| Collect inputs, load profile | Navigate, explore, interact with pages |
| Login (once) | Reason about content, cross-reference data |
| Quick orientation — discover nav structure | Deep page inspection, viewport sweeps |
| Spawn mission agents with context | Screenshot analysis, JS checks |
| Brief progress updates to user | Write detailed walkthroughs and findings |
| Assemble report from mission result files | Write structured mission-{n}-results.md |
| Cross-page consistency (compare fingerprints) | Collect style fingerprints per page |
| Feedback conversation with user | — |

**Keep your context lean.** Don't take screenshots yourself (except login), don't inspect pages in detail, don't reason about content. Pass that work to the mission agents. Your value is in coordination and assembly.

## Architecture

```
Leader Agent (you) — lightweight coordinator
├── Phase 1: Setup — collect inputs, load profile
├── Phase 2: Login — authenticate, save auth state
├── Phase 3: Orient — quick nav discovery, write orientation file
├── Phase 4: Delegate Missions — spawn ALL agents in parallel (each gets own browser session)
│   ├── Mission Agent 1 (-s=mission-1) → writes mission-1-results.md
│   ├── Mission Agent 2 (-s=mission-2) → writes mission-2-results.md
│   └── Mission Agent N (-s=mission-N) → writes mission-N-results.md
├── Phase 5: Assemble — wait for all agents, cross-page consistency from fingerprints
├── Phase 6: Report — read all mission results, write unified report
└── Phase 7: Feedback — update knowledge + profile
```

**Parallel execution**: Each mission agent gets its own named `playwright-cli` session (`-s=mission-{n}`), so they each have an independent browser window. No viewport/navigation conflicts. Auth state is saved to a file after login and loaded by each agent on startup. Each agent closes its own session when done.

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
> This installs the CLI and its Claude Code skills into `.claude/skills/playwright-cli`. Once installed, re-run `/slh-test`."

**Do not proceed without playwright-cli.** Do not fall back to MCP tools, Chrome DevTools, or any other browser automation method.

## Phase 1: Setup

Parse any arguments provided: `$ARGUMENTS`

The format is `[url] [username] [password]`. Any missing values must be collected.

### 1a. Load App Profile (Required)

Use the Glob tool with pattern `slh-reports/app-profile/*.md` to find profile files. If no files match, tell the user:

> "No app profile found. Run `/slh-profile` first to define the target user and missions."

Stop here. Don't proceed without a profile.

If multiple profiles exist and a URL argument was provided, match by hostname in the filename. If multiple profiles exist and no URL was provided, ask the user which profile to use.

Read the matched profile file with the Read tool. Display all personas and their mission lists.

### 1b. Select Persona

If the profile has multiple personas, use `AskUserQuestion` to let the user pick which persona to test as this session. Show each persona's name and a one-line summary. Only one persona per session — testing as a specific user type keeps the evaluation focused and the narrative coherent.

Display the selected persona's full description and mission list.

### 1c. Select Missions

Use `AskUserQuestion` with multiSelect to let the user pick which of this persona's missions to run this session. List all missions from the selected persona. Offer an "All missions" option.

### 1d. Collect Remaining Inputs

Check if saved credentials exist at `./slh-reports/credentials.json`. If found, pre-fill the URL and credentials. Tell the user: "Found saved credentials for {url}."

Use `AskUserQuestion` to collect any missing inputs in a single call:

1. **App URL** — if not provided as argument or pre-filled from credentials/profile
2. **Credentials** — username and password if not provided. Offer "No login required." Tell the user: "Credentials saved to `slh-reports/credentials.json` (gitignored)."
3. **Viewport sizes** — multiSelect:
   - Mobile (375x812) — iPhone standard
   - Tablet (768x1024) — iPad portrait
   - Desktop (1440x900) — Standard desktop (Recommended)
   - Large Desktop (1920x1080) — Full HD
   - All viewports

### 1e. Set Up Output Directory

Extract hostname from URL (replace `:` with `-`, strip protocol/paths).

Build run directory: `./slh-reports/{YYYY-MM-DD_HH-MM}_{hostname}/`

**IMPORTANT**: Resolve the run directory to an **absolute path** (not relative like `./slh-reports/...`). Mission agents run in subprocesses that may resolve relative paths differently. Always pass absolute paths when spawning agents.

### 1f. Load App Knowledge

Use the Glob tool with pattern `slh-reports/app-knowledge/{hostname}.md` (substituting the actual hostname). If no match, also try `slh-reports/app-knowledge/*.md` and check if any filename contains the hostname. If found:
1. Read the file with the Read tool
2. Tell the user: "Loaded knowledge guide for {hostname} — {n} entries"
3. Store the content — it will be passed to each mission agent

### 1g. Create All Directories and Files Upfront

Do ALL file creation in a **single Bash command** so the user only approves once. This lets the test run autonomously from here on:

```bash
mkdir -p {run-directory}/screenshots && echo '{"url":"{url}","username":"{username}","password":"{password}"}' > ./slh-reports/credentials.json
```

Then immediately write the orientation file (Phase 3) and the app knowledge reference — get all the Write approvals done before starting browser work.

After this point, the test should run without further permission prompts (mission agents use `mode: "bypassPermissions"`).

## Phase 2: Login & Save Auth State

Skip if "No login required" was selected.

Use `playwright-cli` commands (see `playwright-cli --help` or the playwright-cli skill):
1. Open the app URL: `playwright-cli open {url}`
2. Take a snapshot to find form fields: `playwright-cli snapshot`
3. Fill credentials into the username and password fields
4. Submit the form
5. Wait for the page to load after login
6. Screenshot the result: `playwright-cli screenshot {absolute-run-directory}/screenshots/00-login-success.png`
7. **Save the authenticated state** for mission agents: `playwright-cli state-save {absolute-run-directory}/auth.json`
8. Close the leader's browser session: `playwright-cli close`

If login fails, report the failure and ask the user for guidance.

The saved `auth.json` contains cookies and storage state. Each mission agent will load this file into its own independent browser session so it starts already authenticated.

## Phase 3: Orient (Quick)

Map the navigation structure so mission agents know what's available. **Keep this brief — don't analyze pages, don't take extra screenshots, don't reason about content.** That's the mission agents' job.

Open a temporary browser session for orientation:
1. `playwright-cli -s=orient open {url}`
2. `playwright-cli -s=orient state-load {absolute-run-directory}/auth.json` (skip if no login)
3. `playwright-cli -s=orient snapshot` — read the page structure
4. Identify the navigation structure — sidebar links, header menus, tabs, breadcrumbs
5. Build an ordered list of discoverable pages/routes with their labels
6. `playwright-cli -s=orient close` — close the orientation session

**Save orientation data** for mission agents. Write `{run-directory}/orientation.md` with:
- The full navigation structure (all discovered pages/routes with labels)
- The persona description (from the profile)
- The app knowledge content (if loaded, paste it in full)

This write was pre-approved during Phase 1g setup. No additional permission prompt needed.

Tell the user what pages you discovered and which missions you're about to delegate.

## Phase 4: Delegate Missions (Parallel)

Spawn **all** mission agents at once using the Task tool with `run_in_background: true`. Each agent gets its own named `playwright-cli` session so there are no browser conflicts.

### Spawning Mission Agents

Use the Task tool with `subagent_type: "general-purpose"`, `run_in_background: true`, `mode: "bypassPermissions"`, and `model: "haiku"`.

**Spawn all missions in a single message** — include multiple Task tool calls in one response so they launch in parallel.

**The mission agent can ONLY see the prompt you pass it.** It has no access to this command file, the skill references, or this conversation. Everything it needs must be in the prompt text.

When constructing the prompt below, you MUST:
- Replace ALL `{placeholders}` with real values — the agent can't resolve them
- Use the **absolute** run directory and screenshots paths (from Phase 1e)
- Paste the real persona description, navigation structure, and app knowledge content
- Give each agent a unique session name: `mission-1`, `mission-2`, etc.
- Do NOT leave any `{...}` placeholder text in the prompt

Provide a detailed prompt that includes everything the agent needs:

```
You are a mission agent evaluating a web app. You are testing as a specific persona and your job is to attempt ONE mission, find issues, and write structured results.

CRITICAL RULES:
1. You have your own dedicated browser session: `-s={session-name}`. Prefix EVERY `playwright-cli` command with `-s={session-name}`. Example: `playwright-cli -s={session-name} snapshot`, `playwright-cli -s={session-name} click e5`, etc.
2. Use `playwright-cli` bash commands for ALL browser interaction — snapshots, clicks, navigation, screenshots, resizing, JS evaluation.
3. NEVER use Playwright MCP tools (browser_snapshot, browser_click, browser_evaluate, etc.). NEVER use any tool prefixed with `mcp__*playwright*`. Those are a DIFFERENT interface. You MUST use `playwright-cli` bash commands ONLY.
4. NEVER use Bash/curl to call APIs — you are testing the UI, not the API.
5. SCREENSHOTS: Always include the FULL ABSOLUTE PATH. Example: `playwright-cli -s={session-name} screenshot {screenshots_dir}/pagename-desktop-1440x900.png`

## Browser Setup — Do This First

Open your session and load the authenticated state:
1. `playwright-cli -s={session-name} open {url}`
2. `playwright-cli -s={session-name} state-load {absolute-run-directory}/auth.json`
3. `playwright-cli -s={session-name} goto {url}` (reload to apply auth)

When done with ALL work, close your session:
`playwright-cli -s={session-name} close`

## Your Mission
{mission number}: {mission goal text}

## Persona
{full persona description paragraph from the profile}

## Context
- App URL: {url}
- Session name: {session-name}
- Screenshots directory: {ABSOLUTE path to screenshots dir}
- Results file: {ABSOLUTE path to run dir}/mission-{n}-results.md
- Auth state file: {ABSOLUTE path to run dir}/auth.json
- Selected viewports: {viewport list with dimensions}

## Navigation Structure
{paste the discovered pages/routes from orientation}

## App Knowledge (skip known patterns)
{paste app knowledge content, or "None loaded"}

## What To Do

### 1. Orient (mission 1 only)
If you are mission 1, start with a brief first-impressions paragraph through the persona's lens before navigating:
- "As a {persona}, my first reaction to this app is..."
- Is the purpose clear? Does the navigation use terms I'd understand?
- Where do I think I should go for my mission?
Write this in the Walkthrough section of your results. Later missions can skip this.

### 2. Navigate With Purpose
Think like the persona: "I want to {mission goal}. Where would I look?"
- Start from the navigation. Which item looks most promising?
- If unsure, try the most logical option. Note if you had to guess — that's a discoverability finding.
- Follow the path naturally. Click what the persona would click.
- If you hit a dead end, try an alternative path. Note the dead end as a finding.

### 3. Arrive & Reason
At each page you visit, stop and reason:
- Does the content match expectations?
- Do the numbers add up? Cross-reference counts and badges with actual content.
- Is the purpose clear? Can I tell what this page is for in 3 seconds?
- Can I figure out what to do? Are the actions I need obvious?
- Look for: raw UUIDs, "undefined"/"null"/"NaN", placeholder text, debug info

Use `playwright-cli -s={session-name} snapshot` to read the accessibility tree (NOT `browser_snapshot` or any MCP tool).

### 4. Try to Use It
Actually attempt the mission task:
- Click buttons, follow flows, interact with forms
- Complete the mission if possible
- Note where you get stuck, confused, or surprised

**Dialogs and permissions:**
- **beforeunload dialogs**: Accept them before navigating away from pages with forms.
- **Browser permission dialogs** (microphone, camera, location, notifications): Dismiss these — they block all interaction.
- **Avoid voice/media input buttons**: Do NOT click microphone icons, voice input buttons, camera buttons, or screen-share buttons. Use text input fields instead.

### 5. Viewport Sweep
At each significant page, test each viewport:
1. Resize browser: `playwright-cli -s={session-name} resize {width} {height}`
2. Wait 0.5s for reflow
3. Take screenshot: `playwright-cli -s={session-name} screenshot {screenshots_dir}/{page-slug}-{viewport-name}-{width}x{height}.png`
4. Visually inspect the screenshot for layout/visual issues
5. Run these JS checks with `playwright-cli -s={session-name} eval '...'`:

Horizontal overflow:
() => {
  const hasOverflow = document.documentElement.scrollWidth > document.documentElement.clientWidth;
  const overflowAmount = document.documentElement.scrollWidth - document.documentElement.clientWidth;
  return { hasOverflow, overflowAmount, scrollWidth: document.documentElement.scrollWidth, clientWidth: document.documentElement.clientWidth };
}

Touch targets:
() => {
  const interactive = document.querySelectorAll('a, button, input, select, textarea, [role="button"], [role="link"], [role="tab"]');
  const small = [];
  for (const el of interactive) {
    const rect = el.getBoundingClientRect();
    if (rect.width > 0 && rect.height > 0 && (rect.width < 44 || rect.height < 44)) {
      small.push({ tag: el.tagName, text: el.textContent?.trim().substring(0, 50), width: Math.round(rect.width), height: Math.round(rect.height), role: el.getAttribute('role') });
    }
  }
  return { count: small.length, elements: small.slice(0, 10) };
}

6. Test interactive elements on Desktop only (buttons respond, dropdowns visible)
7. Collect style fingerprint on Desktop only (once per page):

() => {
  const getStyles = (el) => {
    const s = getComputedStyle(el);
    return { fontSize: s.fontSize, fontWeight: s.fontWeight, color: s.color, backgroundColor: s.backgroundColor, borderRadius: s.borderRadius, padding: s.padding };
  };
  const sample = (selector, limit = 3) =>
    [...document.querySelectorAll(selector)].slice(0, limit).map(el => ({
      text: el.textContent?.trim().substring(0, 40),
      styles: getStyles(el),
      rect: { w: Math.round(el.getBoundingClientRect().width), h: Math.round(el.getBoundingClientRect().height) },
    }));
  return {
    primaryButtons: sample('button[class*="primary"], button[class*="default"], [role="button"]'),
    headings: sample('h1, h2, h3'),
    links: sample('a:not([role="button"])'),
    inputs: sample('input, textarea, select'),
    cards: sample('[class*="card"], [class*="Card"]'),
    badges: sample('[class*="badge"], [class*="Badge"]'),
    iconButtons: sample('button:has(svg):not(:has(span))'),
  };
}

### 6. Record Findings
Check each observation against app knowledge before recording. Classify each finding:
- Category: Broken, Confusing, Inconsistent, Rough, or Observation
- Severity: High, Medium, or Low
- Write descriptions through the persona's lens
- For Broken/Rough: include reproduction steps and a Playwright test suggestion

### 7. Write Results
When done, write your results to the RESULTS FILE path from the Context section above. Use the Write tool with the full absolute path.

Use this EXACT format:

# Mission {n}: {mission goal}

## Verdict
{Accomplished | Partially Accomplished | Failed}

## Walkthrough
{First-person narrative as the persona. Describe the path you took, where you hesitated, what worked, where you got stuck. 2-4 paragraphs.}

## Findings

### Finding: {short title}
- **Category**: {Broken|Confusing|Inconsistent|Rough|Observation}
- **Severity**: {High|Medium|Low}
- **Page**: {page-path}
- **Viewport**: {viewport-name} ({width}x{height}) or "All viewports"
- **Description**: {description through persona's lens}
- **Screenshot**: screenshots/{filename}.png
- **Repro Steps**: {numbered steps, or "N/A" for Confusing/Inconsistent/Observation}
- **Expected**: {what should happen, or "N/A"}
- **Actual**: {what actually happens, or "N/A"}
- **Playwright**: {test code block, or "N/A — design question"}

{Repeat for each finding. If no findings, write "No findings from this mission."}

## Pages Visited
{For each page visited during this mission:}

### {page-path}
- **Context**: {why — e.g., "Navigated here looking for team member list"}
- **Desktop (1440x900)**: PASS | FAIL (Finding: {title})
- **Mobile (375x812)**: PASS | FAIL (Finding: {title})
{etc. for each viewport tested}

## Style Fingerprints
{For each page where a fingerprint was collected:}

### {page-path}
{paste the JSON result from the fingerprint JS}

### 8. Clean Up
Close your browser session when all work is complete:
`playwright-cli -s={session-name} close`
```

### Waiting for Missions

After spawning all mission agents in parallel, wait for them all to complete. Use the `TaskOutput` tool (with `block: true`) for each agent to wait for its result.

As each completes, read its results file to get the verdict and finding count. Give a brief progress update to the user:
- "Mission {n}/{total} done: '{mission name}' — {verdict}, {finding count} findings"

## Phase 5: Wrap Up

After all mission agents have completed:

### 5a. Curiosity Expansion

Spawn one more mission agent for curiosity expansion using a new session (`-s=curiosity`). Build a list of 1-2 pages that no mission agent visited (compare the Pages Visited sections in each results file against the full nav structure from orientation). Pass the list to the agent with the same prompt structure as other missions (including Browser Setup with auth state loading, session name, and the instruction to close the session when done). The agent writes `{run-directory}/curiosity-results.md` (same format, "Curiosity Expansion" as mission name, "N/A" for verdict).

### 5b. Cross-Page Consistency

Read all mission results files and extract the **Style Fingerprints** sections. Compare the JSON values across pages — you're just comparing data, not browsing. Use the methodology from the consistency-checks reference. Look for:
- Button styling differences across pages
- Typography inconsistencies (heading sizes, font weights)
- Input/card/badge style variations
- Color usage differences for the same semantic meaning

Record any inconsistencies — these go directly into the report. This is data comparison, not page inspection, so the leader handles it.

## Phase 6: Report (Assembly Only)

You are **assembling** the report from mission agent output, not writing original analysis. Read the report template reference for the format. Read ALL mission results files (`mission-*-results.md`) and the curiosity results. The mission agents did the hard work — your job is to number findings, stitch walkthroughs together, and add the structural sections (header, summary, matrix).

### 6a. Assign Finding Numbers

Findings come from multiple agents with no sequential numbers. Assign global finding numbers now:
1. Collect all findings from all mission results + curiosity results
2. Order by severity (High → Medium → Low), then by category (Broken → Confusing → Inconsistent → Rough → Observation)
3. Number them sequentially: Finding #1, #2, #3...
4. Add cross-page consistency findings at the end

### 6b. Evaluation Report — `{run-directory}/report.md`

The primary deliverable. Narrative structure:

1. **Header** — metadata table (URL, date, persona name + one-liner, missions tested, viewports)
2. **Executive Summary** — 2-4 sentences. Overall verdict: is this app usable for this persona? Biggest concern?
3. **By the Numbers** — findings matrix (category x severity counts)
4. **Mission Walkthroughs** — paste each mission's walkthrough narrative verbatim from the results files. Add global finding number references (e.g., "Finding #3") where the agent mentioned issues. Don't rewrite the walkthroughs — the mission agents wrote them in the persona's voice already.
5. **All Findings** — paste each finding from the results files with its global number. Reformat to match the report template but preserve the mission agent's descriptions verbatim:
   - Finding number, title, category / severity
   - Page and viewport
   - Description (from mission agent — don't rewrite)
   - Screenshot
   - For Broken/Rough: reproduction steps + Playwright test (from mission agent)
6. **Cross-Page Consistency** — from Phase 5b
7. **What Works Well** — 2-4 positives. Pull these from mission walkthroughs — look for places where agents noted things worked smoothly or were easy to find.

All screenshot paths use `screenshots/filename.png` format (relative to run directory).

### 6c. Page Results Log — `{run-directory}/page-results.md`

Full coverage record assembled from all mission results: every page + viewport combination tested with PASS/FAIL and screenshot references.

### 6d. Present Results

After writing both files, tell the user:
- The run directory path
- Executive summary
- Findings count by category and severity
- Which missions succeeded/failed
- Where to read the full report

### 6e. Update Profile Session Count

Read the app profile, increment the Sessions count, write it back.

## Phase 7: Feedback & Learning

After presenting the report summary, ask the user:

> "Want to review any findings? If I got something wrong — flagged something that's intentional, or misunderstood how a feature works — tell me and I'll update the knowledge guide for next time."

When the user provides feedback:

1. Determine the knowledge file path: `./slh-reports/app-knowledge/{hostname}.md`
2. If the file doesn't exist, create it using the structure from the app-knowledge reference
3. Add learnings to the appropriate section
4. Update the "Last updated" date and increment session count
5. Confirm what was added

Also offer: "Should I update the app profile? Any missions to add, remove, or reword based on what we found?"

If the user wants profile changes, read and update the profile file conversationally.

## Error Handling

- If a mission agent fails or returns an error, log it and tell the user — other agents continue independently
- If a page fails to load inside a mission agent, the agent should screenshot the error, record a Broken finding, and continue
- If auth state fails to load in a mission agent, the agent should report this and attempt to continue (some pages may work without auth)
- Never stop the entire test because of a single mission failure
- After all agents finish, run `playwright-cli kill-all` to clean up any leftover browser processes
