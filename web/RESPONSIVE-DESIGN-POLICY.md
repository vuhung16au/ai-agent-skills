# RESPONSIVE-DESIGN-POLICY.md

## Purpose

Define standards for building responsive, accessible, and mobile-first user interfaces for web and mobile contexts. This policy governs agent behavior when generating layout, styling, and interaction code.

---

## Scope / When to Use

Apply this policy when generating or modifying:
- HTML and CSS/SCSS layouts for web applications.
- Component markup and styling in React, Vue, or similar frameworks.
- Tailwind CSS utility classes for responsive layout.
- Responsive web design for both desktop and mobile viewports.
- UI code intended to work across screen sizes and input methods.

---

## Core Principles

- **Mobile-first.** Design and code for the smallest screen first; enhance for larger screens progressively.
- **Content determines breakpoints.** Do not use device-specific breakpoints; let content flow guide breakpoint choices.
- **Accessibility is a first-class requirement, not an afterthought.**
- **Semantic HTML carries layout intent.** Structure conveys meaning.
- **Touch and pointer input are both valid.** Design for both simultaneously.

---

## Required Behavior

### Layout Strategy
- Build layouts using CSS Flexbox or Grid as the primary layout mechanism.
- Never use `float` for primary layout — reserve it only for text wrapping use cases.
- Use relative units (`%`, `em`, `rem`, `vw`, `vh`, `clamp()`) instead of fixed `px` for fluid layouts.
- Do not use fixed widths on containers that must flex across viewport sizes.
- Use `max-width` on content containers to prevent excessively long line lengths on large screens.

```css
.content {
  width: 100%;
  max-width: 72rem; /* ~1152px */
  margin-inline: auto;
  padding-inline: 1rem;
}
```

### Mobile-First CSS
- Write base styles for mobile (small screen) first.
- Add media queries to enhance the layout for larger viewports.
- Use `min-width` media queries, not `max-width`, for a mobile-first approach.

```css
/* Mobile first */
.grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

/* Tablet and up */
@media (min-width: 48rem) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Desktop */
@media (min-width: 64rem) {
  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

### Tailwind CSS
- Apply responsive prefixes in order: base → `sm:` → `md:` → `lg:` → `xl:`.
- Avoid overriding styles at smaller breakpoints with `max-*` prefixes unless explicitly needed.
- Do not mix Tailwind responsive classes with custom media queries in the same component — pick one approach.

### Typography
- Use `rem` for font sizes, not `px`, to respect user-defined browser font size preferences.
- Set a minimum body font size of `1rem` (16px equivalent).
- Limit line length to 60–80 characters (`ch` units) for readable text blocks.
- Use `clamp()` for fluid typography that scales between minimum and maximum sizes.

```css
h1 {
  font-size: clamp(1.75rem, 4vw, 3rem);
}
```

### Touch Targets
- All interactive elements (buttons, links, form controls) must have a minimum touch target of **44×44 px** (WCAG 2.5.5).
- Do not place interactive elements closer than 8px apart.
- Avoid hover-only interactions — provide equivalent tap/click interactions.

### Images and Media
- Use `max-width: 100%` and `height: auto` on images to prevent overflow.
- Use the `<picture>` element or `srcset` attribute for art-directed or resolution-switched images.
- Specify `width` and `height` attributes on `<img>` elements to prevent layout shift (CLS).
- Use `aspect-ratio` CSS property to reserve space for media before it loads.

### Semantic HTML
- Use `<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`, `<section>`, `<article>` to structure content.
- Use `<button>` for actions, `<a>` for navigation. Do not use `<div>` as a click target.
- Use `<ul>`/`<ol>` for navigation and lists of items.
- Use heading hierarchy (`h1`–`h6`) correctly without skipping levels.

### Accessibility
- All form inputs must have visible, associated `<label>` elements.
- Maintain a minimum color contrast ratio of 4.5:1 for normal text (WCAG AA).
- Do not convey information through color alone.
- Ensure keyboard navigation follows a logical tab order.
- Use `focus-visible` styles so keyboard users can see the focus indicator.
- Add `aria-label` to icon-only buttons and interactive elements without visible text.
- **For brand-consistent color choices**, apply the semantic CSS custom properties defined in [`design/COLOR-PALETTE-POLICY.md`](../design/COLOR-PALETTE-POLICY.md). The palette is pre-validated for accessible contrast on recommended pairings.

### Navigation Patterns
- On mobile, use hamburger menus or bottom navigation only when the navigation exceeds what fits comfortably.
- Hamburger menus must be keyboard-accessible and closable with the Escape key.
- Active navigation state must be visually and programmatically indicated.

---

## Things to Avoid

- Fixed `px` widths on fluid containers.
- Hiding content exclusively via `display: none` without providing an accessible alternative.
- Using `!important` to work around responsive layout issues.
- Targeting specific device models or screen resolutions with media queries.
- Touch targets smaller than 44×44 px.
- Relying on hover states for any primary interaction.
- Using `vw` units for font sizes without a fallback (`clamp()` is safer).
- Generating layouts that break at intermediate screen widths.

---

## Output Expectations

Generated responsive UI code must:
- Follow mobile-first CSS ordering.
- Use relative units for layout dimensions.
- Include semantic HTML elements.
- Have interactive elements with sufficient touch target size.
- Include accessible labels and ARIA attributes where needed.
- Not break at common viewport widths (320px, 768px, 1024px, 1440px).

---

## Optional Checklist

- [ ] Base styles written for mobile; enhanced for larger screens.
- [ ] `min-width` media queries used (not `max-width`).
- [ ] Fluid units (`rem`, `%`, `clamp()`) used for fonts and layout.
- [ ] All interactive elements have 44×44 px minimum touch target.
- [ ] Semantic HTML elements used throughout.
- [ ] Color contrast meets WCAG AA.
- [ ] Keyboard navigation is functional.
- [ ] Images include `width`, `height`, and `alt` attributes.
- [ ] No hover-only interactions.
