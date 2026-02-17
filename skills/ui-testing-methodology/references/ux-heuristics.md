# UX Evaluation Heuristics

Practical prompts for evaluating usability during mission-based exploration. These aren't abstract principles — they're questions to ask yourself at each page and interaction.

## 1. Discoverability

*Can the target user find what they need?*

- **Is the path obvious?** If the mission is "find who reports to you," is there a clear nav item or link that says something like "My Team" or "Org Chart"? Or do you have to guess which of 8 sidebar items might contain that information?
- **Does the label match the expectation?** The target user thinks in their own vocabulary. If they'd call it "My Projects" but the app calls it "Initiatives," that's a discoverability gap.
- **How many clicks to get there?** A core task buried 3 levels deep under a non-obvious parent is a problem. Count the clicks for each mission.
- **Is search useful?** If you can't find something through navigation, does search help? Does it return relevant results for natural queries?
- **Are related things near each other?** If I'm looking at a team, can I easily get to the team's projects, members, and goals from there?

## 2. Information Architecture

*Does the structure make sense?*

- **Does the navigation hierarchy match the user's mental model?** A corporate user expects "People → Teams → Projects" grouping. If projects are under Settings, that's confusing.
- **Are there dead ends?** Pages with no onward links, no breadcrumbs, no way to navigate except the browser back button.
- **Is the depth appropriate?** Too flat (everything on one page) is overwhelming. Too deep (5 clicks to reach content) is frustrating.
- **Are similar things grouped together?** All settings in one place, all content in another — not scattered.
- **Does the breadcrumb/title tell you where you are?** If you landed on this page from a link, would you know where you are in the app?

## 3. Page Purpose & Clarity

*Does each page communicate what it's for?*

- **Can you tell what this page is for in 3 seconds?** A clear heading, relevant content, and logical layout should make the purpose obvious.
- **Is there too much competing for attention?** Multiple CTAs, dense data, and no visual hierarchy make it hard to know what to do first.
- **Are empty states helpful?** An empty page should explain why it's empty and what to do about it ("No projects yet. Create your first project."), not just show a blank table.
- **Do numbers and summaries agree?** If a card says "5 members" but the detail page shows 4, something is wrong.
- **Is the most important action prominent?** The primary CTA should be the most visually dominant actionable element.

## 4. Feedback & State

*Does the app tell you what's happening?*

- **Do actions have feedback?** After clicking "Save," is there a success message, a toast, a visual change? Or does nothing visible happen?
- **Are loading states clear?** If data takes time, is there a spinner, skeleton, or progress indicator? Or does the page just sit blank?
- **Are errors helpful?** When something goes wrong, does the error message tell you what happened and what to do about it? Or is it a generic "Something went wrong"?
- **Is the current state visible?** Can you tell if something is selected, active, enabled, or disabled? Are toggles clear about on/off?
- **Do destructive actions warn you?** Delete, remove, leave — these should have confirmation. Is it there?

## 5. Error Recovery

*Can the user recover from mistakes?*

- **Is there an undo?** After an action, can you reverse it? Or is it permanent with no warning?
- **Do forms preserve input?** If a form submission fails, is the data still there? Or does the user have to re-enter everything?
- **Are validation messages inline?** Do you see the error next to the field, or a generic message at the top that doesn't tell you which field is wrong?
- **Can you go back?** After a multi-step flow, can you go back to a previous step? Or are you stuck going forward?
- **Is there a way out?** Modals have close buttons, flows have cancel/back, wizards have exit. Can you always bail out?

## 6. Consistency & Predictability

*Does the app behave the same way throughout?*

- **Do similar actions work the same way?** If clicking a row opens a detail page in one list, it should do the same in other lists.
- **Are the same terms used everywhere?** If it's called "Workspace" in the nav, it shouldn't be called "Organization" on the settings page.
- **Do icons mean the same thing?** A pencil icon should always mean "edit," not sometimes "edit" and sometimes "view."
- **Are interactive patterns consistent?** If one dropdown opens on click, they all should. Not some on click, some on hover.
- **Does the visual language hold?** Same colors for same meanings, same spacing rhythms, same component styles across pages.

## 7. Efficiency

*Can experienced users work quickly?*

- **Are there shortcuts for common tasks?** Keyboard shortcuts, quick actions, recent items.
- **Is there unnecessary friction?** Confirmation dialogs for non-destructive actions, required fields that don't need to be required, extra clicks that could be eliminated.
- **Can you bulk-operate?** If a user needs to do the same thing to 10 items, can they select all and act, or must they do it one by one?
- **Does the app remember context?** Filters, sort orders, last-visited pages — does it remember or reset?

---

## How to Use These

During mission exploration, don't run through every question on every page. Instead:

1. **At each new page**: Quickly scan sections 1-3 (can I find things, does the structure make sense, is the purpose clear?)
2. **When interacting**: Check sections 4-5 (does it give feedback, can I recover from mistakes?)
3. **Across pages**: Check section 6 (is it consistent?)
4. **After completing a mission**: Reflect on section 7 (was this efficient?)

Flag findings using the appropriate category from the finding-categories reference.
