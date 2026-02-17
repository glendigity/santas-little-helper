# Visual Inspection Checklist

This checklist feeds into **Layer 4 (Rough)** of the evaluation methodology. Use it during viewport sweeps to catch visual and layout issues. Not every item applies to every page — skip items that aren't relevant.

## Layout

- [ ] No horizontal scrollbar on the page (unless intentional, like a data table)
- [ ] Grid/flex layout holds together — no collapsed columns or overlapping rows
- [ ] Sidebar and main content properly proportioned for viewport
- [ ] No large unexpected empty areas where content should be
- [ ] Fixed/sticky elements (header, footer, FAB) don't overlap content
- [ ] Content fills its container appropriately — no tiny content in massive containers
- [ ] Modals and dialogs centered and properly sized for viewport
- [ ] Cards and list items have consistent sizing and spacing
- [ ] Page doesn't have an excessively long empty scroll area below content

## Typography

- [ ] All text is readable at current viewport (minimum ~14px on mobile)
- [ ] No text overflows its container without truncation handling (ellipsis or wrap)
- [ ] Headings don't run into adjacent elements
- [ ] Long words or URLs don't break layouts (word-break handling)
- [ ] Line lengths are reasonable for readability (not too wide on desktop)
- [ ] Font weights and sizes create clear visual hierarchy
- [ ] No text-on-text overlap from absolute positioning or negative margins

## Interactive Elements

- [ ] All buttons and links are at least 44x44px on mobile viewports
- [ ] Buttons have visible text or accessible labels
- [ ] Form inputs are tall enough to tap easily on mobile
- [ ] Dropdown menus open within the visible viewport
- [ ] Modals have a visible close mechanism and don't overflow the screen
- [ ] Checkboxes and radio buttons have adequate tap areas
- [ ] Date pickers, color pickers, etc. fit within viewport
- [ ] Tooltips don't get clipped by viewport edges

## Responsive Behavior

- [ ] Navigation transforms appropriately (full menu → hamburger on mobile)
- [ ] Images scale down without distortion or overflow
- [ ] Multi-column layouts reflow to fewer columns on smaller screens
- [ ] Data tables either scroll horizontally with indication or reflow to a mobile layout
- [ ] Form layouts stack vertically on mobile (not side-by-side cramped fields)
- [ ] Padding and margins scale appropriately — not too tight on mobile, not too sparse
- [ ] No content becomes inaccessible at any viewport size
- [ ] Breakpoint transitions are clean — no awkward "in-between" states

## Images and Media

- [ ] Images load and display correctly
- [ ] Images maintain aspect ratio (no stretching or squishing)
- [ ] Avatar images have fallbacks (initials or placeholder)
- [ ] Icons are sized appropriately for the viewport
- [ ] No broken image placeholders visible
- [ ] Charts and graphs are readable at current viewport

## Navigation

- [ ] All navigation links are reachable at every viewport
- [ ] Active/current page is visually indicated in navigation
- [ ] Back/breadcrumb navigation works logically
- [ ] Mobile navigation menu opens and closes correctly
- [ ] Tab bars or secondary navigation is scrollable if it overflows
- [ ] No navigation items hidden without a way to access them

## Content and Data

- [ ] Empty states have appropriate messaging (not blank screens)
- [ ] Loading indicators present for async content
- [ ] Error messages are visible and readable
- [ ] Lists with many items have pagination or virtualization
- [ ] Search/filter controls are accessible and usable
- [ ] Data values are formatted correctly (dates, numbers, currencies)
- [ ] No "undefined", "null", or "[object Object]" displayed as content
