# COLOR-PALETTE-POLICY.md

## Purpose

Define the canonical brand color palette and its application rules across all visual outputs. This policy is the single source of truth for color usage in diagrams, charts, interfaces, design systems, and technical figures.

---

## Scope / When to Use

Apply this policy when generating or modifying:
- TikZ / PGFPlots / LaTeX figures that use custom colors.
- Matplotlib plots and data visualizations.
- Web application UI components, CSS custom properties, and Tailwind theme configs.
- Mobile application design tokens and style definitions.
- Any visual output where color choices must align with the brand system.

This policy is referenced by:
- `figures/FIGURES-POLICY-academic-papers.md` — for branded TikZ/Matplotlib figures.
- `design/INTERACTION-DESIGN-POLICY.md` — for UI component color and affordance rules.
- `web/RESPONSIVE-DESIGN-POLICY.md` — for semantic color tokens in web layouts.
- `latex/LATEX-POLICY-general.md` — for `\definecolor` usage in LaTeX documents.

---

## Core Policy

- Treat these as the default brand colors for diagrams, charts, interfaces, and design systems.
- Preserve the color names exactly when possible: `bookpurple`, `bookred`, `bookblack`, `bookwhite`, `lawpurple`, `warmstone`, `deepcharcoal`, `softivory`.
- Prefer semantic mapping over arbitrary substitution. Use primary colors for brand emphasis and secondary colors for surfaces, text, borders, and supporting accents.
- Maintain accessibility and contrast; do not place low-contrast text on similar backgrounds.
- For light themes, prefer `softivory` backgrounds with `deepcharcoal` or `bookblack` text.
- For dark themes, prefer `deepcharcoal` or `bookblack` backgrounds with `bookwhite` or `softivory` text.
- Use `bookred` sparingly for highlights, warnings, active states, or key callouts.
- Use `bookpurple` as the primary brand accent where one dominant accent color is needed.
- Use `lawpurple` as a secondary accent, not the main competing primary, unless the design specifically needs a warmer secondary emphasis.

---

## Canonical Palette

| Name | RGB | Hex | Recommended Role |
|---|---:|---:|---|
| `bookpurple` | 60, 16, 83 | `#3C1053` | Primary accent, headings, links, key lines |
| `bookred` | 242, 18, 12 | `#F2120C` | Alert accent, highlight, important markers |
| `bookblack` | 0, 0, 0 | `#000000` | Maximum contrast text, dark backgrounds |
| `bookwhite` | 255, 255, 255 | `#FFFFFF` | Inverse text, light contrast element |
| `lawpurple` | 181, 24, 37 | `#B51825` | Secondary accent, tags, emphasis blocks |
| `warmstone` | 145, 139, 131 | `#918B83` | Muted text, borders, neutral support |
| `deepcharcoal` | 48, 44, 42 | `#302C2A` | Primary body text, dark UI surface |
| `softivory` | 242, 239, 235 | `#F2EFEB` | Main light background, cards, canvas |

---

## Usage by Medium

### TikZ / LaTeX

- Keep the original `\definecolor` names as the source of truth.
- Default document background should remain neutral; use `softivory` sparingly as fill backgrounds when needed.
- Use `bookpurple` for theorem boxes, key arrows, structural lines, titles, and emphasis nodes.
- Use `bookred` only for critical annotations, warnings, or standout highlights.
- Use `warmstone` for secondary guides, grids, or less prominent outlines.
- Use `deepcharcoal` or `bookblack` for main text and axes.

Standard color definition block (copy into preamble):

```tex
% Primary Colors
\definecolor{bookpurple}{RGB}{60,16,83}
\definecolor{bookred}{RGB}{242,18,12}
\definecolor{bookblack}{RGB}{0,0,0}
\definecolor{bookwhite}{RGB}{255,255,255}

% Secondary Colors
\definecolor{lawpurple}{RGB}{181,24,37}
\definecolor{warmstone}{RGB}{145,139,131}
\definecolor{deepcharcoal}{RGB}{48,44,42}
\definecolor{softivory}{RGB}{242,239,235}
```

### Matplotlib

- Use `softivory` as the default figure and axes background for light-theme plots.
- Use `deepcharcoal` for axes labels, tick labels, and most text.
- Use `bookpurple` as the first series color.
- Use `lawpurple` as the second series color.
- Use `bookred` for emphasis series, thresholds, alerts, or key reference points.
- Use `warmstone` for grid lines, spines, secondary annotations, or subdued series.
- Avoid default Matplotlib palettes when this policy is active.

Suggested palette order:

```python
palette = [
    "#3C1053",  # bookpurple
    "#B51825",  # lawpurple
    "#F2120C",  # bookred
    "#918B83",  # warmstone
    "#302C2A",  # deepcharcoal
]
```

Suggested style defaults:

```python
plt.rcParams.update({
    "figure.facecolor": "#F2EFEB",
    "axes.facecolor": "#F2EFEB",
    "axes.edgecolor": "#918B83",
    "axes.labelcolor": "#302C2A",
    "text.color": "#302C2A",
    "xtick.color": "#302C2A",
    "ytick.color": "#302C2A",
    "grid.color": "#918B83",
    "savefig.facecolor": "#F2EFEB",
})
```

### Web App

Use the following semantic token mapping:

```css
:root {
  --color-primary: #3C1053;      /* bookpurple */
  --color-primary-2: #B51825;    /* lawpurple */
  --color-accent: #F2120C;       /* bookred */
  --color-bg: #F2EFEB;           /* softivory */
  --color-surface: #FFFFFF;      /* bookwhite */
  --color-text: #302C2A;         /* deepcharcoal */
  --color-text-strong: #000000;  /* bookblack */
  --color-border: #918B83;       /* warmstone */
  --color-inverse: #FFFFFF;      /* bookwhite */
}
```

Rules:

- Default light UI: `softivory` background, `bookwhite` cards, `deepcharcoal` text.
- Primary buttons, active navigation, selected states, and focused highlights should use `bookpurple`.
- Secondary chips, badges, tabs, and supporting emphasis can use `lawpurple`.
- Destructive actions, urgent alerts, or strong highlights can use `bookred`.
- Borders, dividers, disabled states, and quiet metadata should use `warmstone`.
- Avoid introducing unrelated blues, teals, or gradients unless the user explicitly requests them.

### Mobile App

- Follow the same semantic mapping as the web app for consistency.
- Use higher contrast combinations for small UI controls and text.
- Prefer `bookpurple` for the main call to action and active tab state.
- Use `softivory` or `bookwhite` for clean screens with `deepcharcoal` text.
- Reserve `bookred` for destructive actions, notifications, or priority badges.
- Keep `warmstone` for separators, outlines, and secondary labels.
- In dark mode, use `deepcharcoal` as the base surface, with `softivory` or `bookwhite` text and restrained purple/red accents.

---

## Do / Don't

- Do keep the palette restrained and recognizable.
- Do use neutral surfaces and let `bookpurple` carry most brand identity.
- Do use `bookred` intentionally, not as a second default primary.
- Do preserve readability in charts, code blocks, labels, and buttons.
- Don't replace these colors with generic framework defaults.
- Don't overuse saturated accents in every component at once.
- Don't put `warmstone` text on `softivory` when strong readability is required.

---

## Quick Reference

Use the following palette as the default design system across TikZ, Matplotlib, web apps, and mobile apps:

- `bookpurple` `#3C1053` — primary accent
- `bookred` `#F2120C` — highlight or alert accent
- `bookblack` `#000000` and `deepcharcoal` `#302C2A` — text and dark surfaces
- `bookwhite` `#FFFFFF` and `softivory` `#F2EFEB` — light surfaces and inverse text
- `lawpurple` `#B51825` — secondary accent
- `warmstone` `#918B83` — muted borders and supporting neutrals

When converting into framework tokens, preserve these semantic roles unless explicitly overridden by the user.
