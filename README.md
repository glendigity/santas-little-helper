# Santa's Little Helper (SLH)

> *"He's a good boy. Your code isn't."*

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that evaluates web apps by exploring them as a real user. Define personas and missions, then SLH navigates your app, finds what's broken/confusing/rough, and writes a narrative QA report with suggested Playwright regression tests.

> **Fair warning**: This was 100% vibe-coded with Claude Opus 4.6. It works, it's useful, but it's early and rough around the edges. Issues, PRs, and feedback very welcome.

Named after the Simpsons' reject greyhound — chaotic, chews everything, digs up things buried in the yard... but he's trying to help.

## What It Does

1. Logs into your app via Playwright CLI
2. Navigates as a chosen persona (e.g., "mid-level manager who uses Jira")
3. Attempts each mission, noting where things break, confuse, or feel rough
4. Tests across viewports (mobile, tablet, desktop) with automated checks
5. Produces a report with findings classified by category/severity and suggested Playwright tests

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- [Playwright CLI](https://github.com/microsoft/playwright-cli) — SLH will prompt you to install if missing:
  ```bash
  # If playwright-cli is already installed
  playwright-cli install --skills

  # Otherwise, use your package manager:
  npx playwright-cli@latest install --skills   # npm
  bunx playwright-cli@latest install --skills   # bun
  pnpx playwright-cli@latest install --skills   # pnpm
  yarn dlx playwright-cli@latest install --skills  # yarn
  ```

## Installation

```
/plugin marketplace add glendigity/santas-little-helper
/plugin install slh
```

## Commands

| Command | Description |
|---------|-------------|
| `/slh-profile [url]` | Define user personas and their mission lists |
| `/slh-test [url] [user] [pass]` | Run a full evaluation session |
| `/slh-retest [url] [user] [pass]` | Re-test previous findings to verify fixes |

## Finding Categories

| Category | Question |
|----------|----------|
| **Broken** | Does it work? |
| **Confusing** | Can I figure it out? |
| **Inconsistent** | Does it match itself? |
| **Rough** | Does it look/feel right? |
| **Observation** | Not sure — needs human judgment |

Each finding gets a severity: **High** (blocks a task), **Medium** (significant friction), or **Low** (noticeable but doesn't impede).

## Why Not Just Write Playwright Tests?

You should — SLH suggests Playwright tests for every finding it discovers. But:

- **Playwright tests** verify things you already know should work. They catch regressions.
- **SLH** finds things you didn't know to test for — confusing navigation, inconsistent data, labels that don't make sense to non-developers.

SLH is also for when you're building fast. The UI changes constantly but your missions stay the same. Playwright tests break when you move a button. SLH just tries to accomplish the goal and tells you what got in the way.

## Why Not Claude in Chrome?

SLH uses `playwright-cli` because it supports named sessions, which lets mission agents run in parallel with independent browser windows. If Claude in Chrome turns out to be better for this, happy to move to it.

## Tips

- Start with `/slh-profile` — good missions are concrete goals ("Find out who reports to you"), not page names ("View the org chart")
- Run `/slh-test` with a subset of missions first to calibrate
- Give feedback after each report — the knowledge base reduces false positives over time
- Commit `.slh/` to git — profiles, knowledge, and orientation are meant to be shared with your team
- Add `slh-reports/` and `.playwright-cli/` to your `.gitignore` — those are generated output
- To change where reports go, create `.slh/config.yml` with `reports_dir: ./your-path`

## Acknowledgements

Inspired by Robbie Baskin's idea for using Claude to take on user missions as a QA tester.

## License

MIT

---

*Fetches bugs so you don't have to.*
