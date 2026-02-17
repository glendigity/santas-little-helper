# Viewport Presets

## Definitions

| Name | Width | Height | Device Reference |
|------|-------|--------|-----------------|
| Mobile | 375 | 812 | iPhone 12/13/14 standard |
| Mobile Landscape | 812 | 375 | Phone rotated |
| Tablet | 768 | 1024 | iPad portrait |
| Tablet Landscape | 1024 | 768 | iPad landscape |
| Desktop | 1440 | 900 | Standard laptop/desktop |
| Large Desktop | 1920 | 1080 | Full HD monitor |

## What to Watch For at Each Size

### Mobile (375x812)

This is where most issues surface. The screen is narrow and touch is the input method.

- Navigation must collapse to a hamburger or bottom bar
- All interactive elements need 44px+ touch targets
- Horizontal overflow is the most common failure — tables, wide forms, and fixed-width elements often break
- Modals and dropdowns must fit within 375px width
- Text input fields should be full-width, not cramped side-by-side
- Font sizes below 14px become hard to read
- Sidebars should be hidden behind a toggle, not squeezed beside content

### Mobile Landscape (812x375)

Short and wide — a challenging combination often overlooked.

- Fixed headers/footers eat a huge portion of the 375px height
- Modals may not fit vertically
- Keyboard-triggered layouts (form focus) may hide content
- Horizontal space is generous so layout doesn't usually break horizontally

### Tablet (768x1024)

The awkward middle ground. Apps designed for "mobile or desktop" often look worst here.

- 768px is a common CSS breakpoint — layouts may flicker between mobile and desktop
- Sidebars may be visible but too narrow to be useful
- Grid layouts with 3+ columns may need to drop to 2
- Touch targets still matter — tablet users touch the screen
- Modals can be wider but shouldn't stretch to full width

### Tablet Landscape (1024x768)

Often where desktop layouts start to work, but with touch-sized constraints.

- 1024px triggers many desktop-first layouts
- Height is limited (768px) so tall modals/menus may overflow
- Sidebar + content layout usually works but may feel tight
- This width often exposes issues with "min-width" assumptions

### Desktop (1440x900)

The primary development viewport. Usually the best-tested, so issues are rarer.

- Check for excessive whitespace or overly wide content areas
- Verify hover states exist for interactive elements
- Ensure dropdowns and tooltips appear in logical positions
- Check that text doesn't stretch to unreadable line lengths on wide containers
- Look for alignment issues in complex layouts (dashboards, data tables)

### Large Desktop (1920x1080)

Tests maximum width behavior.

- Content should have a max-width or centered layout — not stretch edge to edge
- Tables and data grids should handle extra width gracefully
- Navigation should remain usable (items not too spread out)
- Check for visual "emptiness" if layouts don't scale up

## Common Breakpoint Problems

| Breakpoint | Common Issue |
|-----------|-------------|
| < 640px | Content overflow, cramped forms, hidden elements |
| 640-768px | Layout ambiguity — mobile or tablet? Inconsistent behavior |
| 768-1024px | Sidebar visibility flicker, grid column confusion |
| 1024-1280px | Desktop layout with insufficient width for all columns |
| > 1440px | No max-width constraint, content stretches too wide |
