# LATEX-SKILL: TikZ, pgfplots, and groupplots

## Purpose

Teach agents when and how to use **TikZ**, **pgfplots**, and the **groupplots** library together so mathematical booklets, theses, and research papers get high-quality, scalable vector graphics that compile inside LaTeX. Fonts, line weights, and scaling stay consistent with the rest of the document.

---

## Scope / When to Use

Use this skill when generating or revising LaTeX that includes:

- Diagrams, geometry, flowcharts, or custom illustrations drawn from coordinates.
- Plots of mathematical functions, statistics, scatter data, or CSV-driven series.
- Grids of related plots (subplots) that must align axes and spacing.

Apply **[`FIGURES-POLICY-academic-papers.md`](../figures/FIGURES-POLICY-academic-papers.md)** for journal and conference figures (caption stubs, colorblind-safe palettes, explicit sizes, ethics of axes). Use this skill for the **tool choice and minimal working patterns**.

---

## How the Pieces Relate

| Tool | Role | One-line rule |
|------|------|----------------|
| **TikZ** | Foundational graphics language inside LaTeX. | Use **TikZ** to *draw pictures* (shapes, lines, nodes, curves). |
| **pgfplots** | Plotting layer built on TikZ. | Use **pgfplots** to *graph data and math functions* (axes, ticks, `\addplot`). |
| **groupplots** | Library shipped with pgfplots, not a separate package. | Use **groupplots** to *arrange multiple pgfplots into a neat grid*. |

Because graphics are produced at compile time, vector output matches document fonts and weight — appropriate for a professional, textbook-like look.

---

## Core Principles

### 1. TikZ (the foundation)

**TikZ** (*TikZ ist kein Zeichenprogramm* — “TikZ is not a drawing program”) gives coordinate-based drawing: lines, circles, rectangles, paths, and `\node` text.

- **Use for:** Geometry, trees, architecture/block diagrams, neural-network sketches, any schematic where **you** place elements.
- **Mechanism:** Coordinates and `\draw` (e.g. `\draw (0,0) -- (1,1);`, `\draw (0,0) circle (1cm);`).

**Basic example:**

```latex
\usepackage{tikz}

\begin{tikzpicture}
    \draw[->, thick] (-1,0) -- (3,0) node[right] {$x$};
    \draw[->, thick] (0,-1) -- (0,3) node[above] {$y$};
    \draw[fill=blue!20, draw=blue, thick] (0.5, 0.5) rectangle (2.5, 2.5);
    \node at (1.5, 1.5) {Example};
\end{tikzpicture}
```

### 2. pgfplots (graphing)

**pgfplots** adds an `axis` environment: scaling, ticks, legends, and `\addplot` for functions and tables. Prefer it over raw TikZ when the task is *chart-like*.

- **Use for:** \(y = f(x)\), distributions, scatter plots, data from `.csv` or inline coordinates.
- **Mechanism:** `tikzpicture` → `axis` → `\addplot`.

Always set a **compat** line once in the preamble (pin a version for reproducibility, e.g. `1.18`, or follow project policy):

```latex
\pgfplotsset{compat=1.18}
```

**Basic example:**

```latex
\usepackage{pgfplots}
\pgfplotsset{compat=1.18}

\begin{tikzpicture}
    \begin{axis}[
        axis lines = middle,
        xlabel = {$x$},
        ylabel = {$y$},
        domain = -2:2,
        samples = 50
    ]
        \addplot[color=red, thick] {x^2};
        \addlegendentry{$y = x^2$}
    \end{axis}
\end{tikzpicture}
```

### 3. groupplots (multi-panel grids)

**groupplots** is loaded as a **pgfplots library**. It replaces ad-hoc `minipage` / `tabular` alignment for side-by-side axes.

- **Use for:** Compare distributions, function vs. derivative, or any **matrix of related** plots with shared styling.
- **Mechanism:** `\usepgfplotslibrary{groupplots}` → `groupplot` environment → `\nextgroupplot` between panels.

**Basic example:**

```latex
\usepackage{pgfplots}
\usepgfplotslibrary{groupplots}
\pgfplotsset{compat=1.18}

\begin{tikzpicture}
    \begin{groupplot}[
        group style={
            group size=2 by 1,
            horizontal sep=1.5cm
        },
        width=6cm, height=5cm,
        axis lines=middle,
        domain=-pi:pi
    ]
        \nextgroupplot[title={Sine}]
        \addplot[blue, thick, samples=50] {sin(deg(x))};

        \nextgroupplot[title={Cosine}]
        \addplot[red, thick, samples=50] {cos(deg(x))};
    \end{groupplot}
\end{tikzpicture}
```

---

## Summary (decision cheat sheet)

1. Use **TikZ** to *draw pictures*.
2. Use **pgfplots** to *graph data and math functions*.
3. Use **groupplots** to *arrange multiple pgfplots into a neat grid*.

---

## Required Behavior

- Load `tikz` when using low-level drawing; load `pgfplots` when using `axis` / `\addplot`.
- Set `\pgfplotsset{compat=...}` project-wide; do not omit compat for new documents.
- Load groupplots only when needed: `\usepgfplotslibrary{groupplots}` after `pgfplots`.
- For academic figures, follow **[`FIGURES-POLICY-academic-papers.md`](../figures/FIGURES-POLICY-academic-papers.md)** (explicit `width`/`height`, labels with units, colorblind-safe colors, captions/labels).

---

## Things to Avoid

- Using raw TikZ for full numerical axes and dense tick math when `axis` would be clearer.
- Using tables/minipages to align multiple `axis` environments when **groupplots** fits the layout.
- Omitting `\pgfplotsset{compat=...}` on new pgfplots code.
- Treating groupplots as a separate CTAN package — it is part of pgfplots.

---

## Output Expectations

Deliverables should include:

- Correct `\usepackage` / `\usepgfplotslibrary` lines for the chosen stack.
- A compilable `tikzpicture` (or snippet) matching the task (draw vs. single axis vs. groupplot).
- Cross-check against the figures policy when the output is a paper figure (caption, label, sizing).

---

## Related Files

- **[`LATEX-POLICY-general.md`](LATEX-POLICY-general.md)** — preamble order, floats, and general LaTeX standards.
- **[`FIGURES-POLICY-academic-papers.md`](../figures/FIGURES-POLICY-academic-papers.md)** — publication-quality figure rules for agents.
