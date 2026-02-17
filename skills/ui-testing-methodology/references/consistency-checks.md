# Cross-Page Consistency Checks

After testing all pages individually, compare common UI elements across pages to find inconsistencies. These are things a human would notice immediately — "why does the button look different on this page?"

## What to Collect

Run this JavaScript snippet on each page (at desktop viewport) to capture a style fingerprint of common elements:

```javascript
() => {
  const getStyles = (el) => {
    const s = getComputedStyle(el);
    return {
      fontSize: s.fontSize,
      fontWeight: s.fontWeight,
      fontFamily: s.fontFamily,
      color: s.color,
      backgroundColor: s.backgroundColor,
      borderRadius: s.borderRadius,
      padding: s.padding,
      lineHeight: s.lineHeight,
      gap: s.gap,
    };
  };

  const sample = (selector, limit = 3) =>
    [...document.querySelectorAll(selector)].slice(0, limit).map(el => ({
      text: el.textContent?.trim().substring(0, 40),
      tag: el.tagName,
      styles: getStyles(el),
      rect: { w: Math.round(el.getBoundingClientRect().width), h: Math.round(el.getBoundingClientRect().height) },
    }));

  return {
    primaryButtons: sample('button[class*="primary"], button[class*="default"], [role="button"]'),
    headings: sample('h1, h2, h3'),
    links: sample('a:not([role="button"])'),
    inputs: sample('input, textarea, select'),
    cards: sample('[class*="card"], [class*="Card"]'),
    badges: sample('[class*="badge"], [class*="Badge"], [class*="tag"], [class*="Tag"]'),
    iconButtons: sample('button:has(svg):not(:has(span))'),
  };
}
```

## What to Compare

After all pages are tested, compare the collected fingerprints. Flag inconsistencies in these categories:

### 1. Button Styling
- Primary/default buttons should have the same background color, border-radius, padding, and font size across all pages
- Icon-only buttons should be consistently sized
- Destructive/danger buttons should use the same red/color treatment everywhere

### 2. Typography
- Same heading level (h1, h2, h3) should use the same font-size and font-weight across pages
- Body text should be consistent
- Link styling (color, underline, hover) should be uniform

### 3. Spacing & Layout
- Page content padding/margins should be consistent
- Card padding and border-radius should match
- Section spacing should follow the same rhythm

### 4. Component Variants
- Badges/tags should use the same sizing and border-radius
- Form inputs should have consistent height, padding, and border treatment
- Avatars should be the same size in the same context (e.g., list items)

### 5. Color Usage
- The same semantic meaning should use the same color (e.g., "success" green should be identical)
- Hover/focus states should be consistent
- Background colors for similar sections should match

### 6. Empty & Loading States
- Empty states should follow the same pattern (icon + message + action)
- Loading indicators should be consistent (all spinners or all skeletons, not mixed)

## How to Report

Inconsistencies go in the issues report as a dedicated section after individual issues. Each inconsistency should include:

- **What varies**: e.g., "Primary button border-radius"
- **Where**: which pages show which variant
- **Evidence**: the specific values (e.g., "4px on /dashboard, 8px on /settings")
- **Severity**: usually MINOR unless it creates confusion about interactive elements (then MODERATE)

Don't flag:
- Intentional differences noted in the app knowledge file
- Page-specific components that only appear once (nothing to compare)
- Negligible sub-pixel differences (1px rounding)
