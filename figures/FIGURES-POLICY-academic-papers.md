# FIGURES-POLICY-academic-papers.md

## Purpose

Define how AI coding agents should generate figure code for academic papers. This policy governs TikZ, PGF, PGFPlots, and related LaTeX figure workflows to produce journal-ready, reproducible, and honest visualizations.

---

## Scope / When to Use

Apply this policy whenever generating:
- TikZ, PGFPlots, or PGF figure source code for academic manuscripts.
- Standalone LaTeX figure files intended for journal or conference submission.
- Scripts or snippets that produce figures embedded in LaTeX documents.

---

## Core Principles

These principles are adapted from the "Ten Simple Rules for Better Figures" framework and translated into actionable agent behavior.

### 1. Know Your Audience
- Generate figures appropriate for a technical, academic readership unless told otherwise.
- Avoid decorative elements that add no information value.
- Default to a style consistent with major peer-reviewed venues (IEEE, ACM, Springer, Elsevier, PLOS).

### 2. Identify the Message
- Every figure must convey one clear message.
- If asked to generate a figure, confirm or infer the key takeaway before writing code.
- Do not add secondary data series, annotations, or visual elements that dilute the primary message.

### 3. Adapt to the Support Medium
- Default to single-column width unless the user specifies otherwise (`\columnwidth` in two-column formats).
- For two-column journals, generate figures scaled to fit a single column (approx. 88 mm) or full page width (approx. 180 mm).
- Always generate vector output (PDF via LaTeX, not rasterized PNG) unless explicitly asked for a raster format.

### 4. Captions Are Not Optional
- Always include a `\caption{}` command stub or placeholder text.
- Caption text should summarize the figure's message, not just label it.
- Remind the user to complete the caption if placeholder text is used.

### 5. Do Not Trust Defaults
- Never use PGFPlots or TikZ library defaults without review.
- Explicitly set axis label font sizes, tick label sizes, line widths, and mark sizes.
- Set `width` and `height` explicitly on every `tikzpicture` or `axis` environment.
- Use `\pgfplotsset{compat=newest}` or a pinned version to ensure reproducibility.

### 6. Use Color Effectively
- Default to a colorblind-safe palette (e.g., Wong, Tol, or ColorBrewer qualitative schemes).
- Never rely on color alone to encode meaning — use line style, marker shape, or pattern as well.
- Limit the number of distinct colors to what is strictly necessary (typically ≤ 5 for line plots).
- Avoid overly saturated or neon colors.
- Use grayscale-safe choices when color reproduction cannot be guaranteed.

### 7. Do Not Mislead the Reader
- Always start y-axes at zero for bar charts unless there is a documented, explicitly stated reason not to.
- Never truncate axes in a way that exaggerates differences without a clear visual indicator (e.g., axis break).
- Do not manipulate aspect ratios to inflate or minimize trends.
- Do not add trendlines, smoothing, or interpolation unless the user explicitly requests it and it is statistically appropriate.

### 8. Avoid Chartjunk
- Do not generate grid lines unless they aid readability (`grid=major` only when useful).
- Remove unnecessary tick marks, borders, and background fills.
- Do not use 3D effects, drop shadows, or gradients on 2D data.
- Prefer clean axis styles: `axis lines=left` or `axis lines=box` as appropriate.

### 9. Message Trumps Beauty
- Prioritize clarity over visual complexity.
- If a simple line plot communicates the result, do not upgrade it to a stacked area chart.
- Resist adding visual polish that complicates the code and reduces maintainability.

### 10. Get the Right Tool
- Use PGFPlots for data-driven charts (line, bar, scatter, histogram, heatmap).
- Use TikZ for diagrams, architecture drawings, flowcharts, and annotated schematics.
- Use `standalone` document class for figures to be compiled independently.
- Use `\input{}` or `\includegraphics{}` of a compiled PDF for embedding into the main document.

---

## Required Behavior

- Set `\pgfplotsset{compat=newest}` or a pinned compat value at the top of every PGFPlots figure.
- Explicitly declare `width`, `height`, font sizes, and line widths.
- Use named color definitions (`\definecolor`) for all non-default colors.
- Include axis labels with units on every quantitative axis.
- Include a legend only when there are multiple data series; place it where it does not obscure data.
- Generate figures that compile without errors on a standard TeX Live installation.
- Structure the code so that data values are easy to find and update (prefer `\addplot coordinates` or table-based input over hardcoded paths).
- Add short inline comments in the code to identify key configurable parameters.

---

## Things to Avoid

- Hardcoding absolute font sizes in `pt` without reference to the document's base font.
- Using `\textwidth` without confirming single- vs. two-column layout with the user.
- Generating figures that require non-standard or private LaTeX packages.
- Omitting the `\caption` and `\label` commands in `figure` environments.
- Using default PGFPlots color cycle without explicitly verifying it is appropriate.
- Generating raster images when vector output is achievable.
- Adding explanatory text inside the figure that belongs in the caption.

---

## Output Expectations

A correct output from this policy includes:
- A compilable LaTeX snippet or standalone `.tex` file.
- Explicit width/height settings.
- Colorblind-safe colors defined by name.
- Axis labels with units.
- A `\caption{}` and `\label{}` stub.
- Inline comments marking configurable parameters.
- No reliance on proprietary packages or private data files.

---

## Example Structure 1 

```latex
\documentclass[tikz, border=4pt]{standalone}
\usepackage{pgfplots}
\pgfplotsset{compat=1.18}

% --- Color definitions (Wong colorblind-safe palette) ---
\definecolor{wongBlue}{RGB}{0,114,178}
\definecolor{wongOrange}{RGB}{230,159,0}

\begin{document}
\begin{tikzpicture}
  \begin{axis}[
    width=88mm,          % single-column width
    height=60mm,
    xlabel={Time (s)},
    ylabel={Accuracy (\%)},
    xmin=0, xmax=100,
    ymin=0, ymax=100,
    xtick distance=25,
    ytick distance=25,
    tick label style={font=\small},
    label style={font=\small},
    legend style={font=\small, at={(0.97,0.05)}, anchor=south east},
    grid=major,
    grid style={dotted, gray!40},
  ]
    \addplot[color=wongBlue, thick, mark=o, mark size=1.5pt]
      coordinates { (0,50) (25,65) (50,78) (75,88) (100,92) };
    \addlegendentry{Model A}

    \addplot[color=wongOrange, thick, mark=square, mark size=1.5pt]
      coordinates { (0,48) (25,60) (50,72) (75,82) (100,87) };
    \addlegendentry{Model B}
  \end{axis}
\end{tikzpicture}
\end{document}
% Caption (add to figure environment in main document):
% \caption{Accuracy over training time for Model A and Model B. Model A converges faster but reaches a similar final accuracy to Model B.}
% \label{fig:accuracy-comparison}
```

## Example Structure 2

TBC 

## Example Structure 10 

TBC