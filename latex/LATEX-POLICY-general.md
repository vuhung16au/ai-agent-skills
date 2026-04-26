# LATEX-POLICY-general.md

## Purpose

Define standards for writing clean, maintainable, and typographically correct LaTeX documents. This policy governs agent behavior when generating LaTeX source files, preambles, macros, figure/table code, and bibliography configurations.

---

## Scope / When to Use

Apply this policy when generating or modifying:
- LaTeX manuscripts, reports, theses, or articles.
- Document preambles and package configurations.
- Macros and custom commands.
- Figure, table, and listing environments.
- Bibliography and citation management.
- Class files and custom style files.

---

## Core Principles

- **Structure before styling.** Document structure and semantic markup come first; visual adjustments come last.
- **Package discipline.** Load only what is needed. Avoid conflicting packages.
- **Macros over repetition.** Repeated patterns belong in macros, not copy-pasted markup.
- **References are automatic.** Labels and cross-references must be consistent and generated, not hardcoded.
- **Reproducibility.** The document must compile cleanly from source with standard TeX Live.

---

## Required Behavior

### Document Class and Preamble
- Specify the document class explicitly with relevant options: `\documentclass[12pt, a4paper]{article}`.
- Load packages in a deliberate order: encoding → fonts → layout → math → figures → tables → bibliography → hyperref last.
- Load `inputenc` with `utf8` and `fontenc` with `T1` for most documents (or use LuaLaTeX/XeLaTeX with `fontspec` for Unicode workflows).
- Always load `hyperref` last among packages unless another package explicitly requires otherwise.
- Use `\usepackage{microtype}` for improved typography on final documents.

```latex
\documentclass[12pt, a4paper]{article}
\usepackage[utf8]{inputenc}
\usepackage[T1]{fontenc}
\usepackage{lmodern}
\usepackage{geometry}
\usepackage{amsmath, amssymb}
\usepackage{graphicx}
\usepackage{booktabs}
\usepackage{hyperref}
\usepackage{microtype}
```

### Macros and Commands
- Define reusable macros with `\newcommand{\macroname}[num-args]{definition}`.
- Use `\renewcommand` only when overriding an existing definition intentionally; add a comment explaining why.
- Parameterize repeated notation: paper titles, author names, dataset names, frequently-used math symbols.
- Keep macro definitions in a separate file (e.g., `macros.tex`) and include it with `\input{macros}`.
- Do not use `\def` for new macros — use `\newcommand` to get argument count checking and name clash detection.

### Document Structure
- Use semantic sectioning commands: `\section`, `\subsection`, `\subsubsection`.
- Do not skip heading levels.
- Use `\label{sec:identifier}` on every section, figure, table, and equation that may be referenced.
- Use `\ref{}`, `\eqref{}`, and `\autoref{}` (from `hyperref`) for all cross-references — never hardcode numbers.
- Place `\label` immediately after the heading command or inside the float environment after `\caption`.

### Mathematics
- Use `equation` for displayed numbered equations, `align` for multi-line aligned equations.
- Use `\[...\]` for unnumbered displayed math, not `$$...$$`.
- Do not use inline `$` for display-style math that should be on its own line.
- Define math operators with `\DeclareMathOperator` rather than using `\mathrm` inline.
- Use `\text{}` inside math mode for upright text.

### Figures
- Always place figures in a `figure` float environment with `\caption` and `\label`.
- Use `\includegraphics[width=\linewidth]{filename}` (or `\columnwidth` for two-column layouts).
- Include figures as PDF (vector) files for line art and plots; PNG at 300 dpi minimum for photographs.
- Do not hardcode the file extension in `\includegraphics` — let LaTeX find the right format.
- Place `\label` after `\caption` within the `figure` environment.
- Captions must be informative: describe the figure's content and key takeaway.

```latex
\begin{figure}[htbp]
  \centering
  \includegraphics[width=\linewidth]{plots/accuracy-comparison}
  \caption{Accuracy over training epochs for Model A and Model B. Model A reaches peak accuracy faster but converges to a similar final value.}
  \label{fig:accuracy-comparison}
\end{figure}
```

### Tables
- Use the `booktabs` package for publication-quality tables (`\toprule`, `\midrule`, `\bottomrule`).
- Never use vertical lines in data tables — they are almost always chartjunk.
- Define column alignment explicitly: `l`, `c`, `r`, or `S` (siunitx) for numeric columns.
- Use `\siunitx` for aligned numeric columns with units.
- Always provide a `\caption` and `\label` in the `table` float environment.

```latex
\begin{table}[htbp]
  \centering
  \caption{Comparison of model performance metrics.}
  \label{tab:model-comparison}
  \begin{tabular}{l S[table-format=2.1] S[table-format=2.1]}
    \toprule
    Model & {Accuracy (\%)} & {F1 Score (\%)} \\
    \midrule
    Baseline & 78.3 & 74.1 \\
    Proposed & 85.7 & 82.4 \\
    \bottomrule
  \end{tabular}
\end{table}
```

### Bibliography
- Use BibLaTeX with Biber as the default bibliography system for new documents.
- Load with `\usepackage[backend=biber, style=authoryear]{biblatex}` (or the appropriate citation style for the venue).
- Use `\addbibresource{references.bib}` to specify the bibliography file.
- Cite with `\cite{}`, `\parencite{}`, or `\textcite{}` depending on the citation context.
- Print the bibliography with `\printbibliography`.
- Keep `.bib` entries consistent: always include `doi` or `url`, `year`, and the full title.
- Do not use BibTeX (legacy) in new documents unless the journal template requires it.

### Typography Awareness
- Use en-dashes (`--`) for ranges (pp. 1--10) and em-dashes (`---`) for parenthetical remarks.
- Use `\,` before units in math: `$5\,\text{ms}$`.
- Use `~` for non-breaking spaces before citations and references: `Figure~\ref{fig:results}`.
- Use `\ldots` for ellipsis, not `...`.
- Do not manually add spacing before or after punctuation — LaTeX handles it.

---

## Things to Avoid

- Loading packages without knowing what they do or whether they conflict with existing packages.
- Using `\def` for new commands.
- Hardcoding cross-reference numbers instead of using `\ref{}`.
- Using `$$...$$` for display math.
- Including raster images (PNG/JPEG) for plots and diagrams where vector formats are possible.
- Vertical lines in tables.
- Missing `\label` on any section, figure, table, or equation that could be referenced.
- Using BibTeX instead of BibLaTeX/Biber for new documents.
- Overriding LaTeX spacing manually with `\vspace`, `\hspace`, `\\` except as a last resort.

---

## Output Expectations

Generated LaTeX code must:
- Compile without errors on a standard TeX Live installation.
- Use `\newcommand` for all custom macros.
- Use `\label` and `\ref` for all cross-references.
- Use `booktabs` for tables.
- Load `hyperref` last in the preamble.
- Use BibLaTeX/Biber for bibliography.
- Not hardcode cross-reference numbers.
- Include `\caption` and `\label` on all float environments.

---

## Optional Checklist

- [ ] Document class and options explicitly specified.
- [ ] `hyperref` loaded last in preamble.
- [ ] `microtype` included for final documents.
- [ ] Custom macros defined with `\newcommand` in a `macros.tex` file.
- [ ] All sections, figures, tables, and equations have `\label`.
- [ ] All cross-references use `\ref{}`, `\eqref{}`, or `\autoref{}`.
- [ ] Figures use `\includegraphics` with vector format.
- [ ] Tables use `booktabs`; no vertical lines.
- [ ] Bibliography uses BibLaTeX/Biber.
- [ ] Non-breaking spaces before `\ref` and `\cite`.
