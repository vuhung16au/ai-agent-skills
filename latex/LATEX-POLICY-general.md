# LATEX-POLICY-general.md

## Purpose

Define standards for writing clean, maintainable, and typographically correct LaTeX documents. This policy governs agent behavior when generating LaTeX source files, preambles, macros, figure/table code, and bibliography configurations.

---

## Scope / When to Use

Apply this policy when generating or modifying LaTeX documents across the following contexts:

### Academic Writing (Proposals and Papers)
- Writing research proposals and academic papers intended for journal or preprint submission.
- Key submission target: **arXiv** ([arxiv.org](https://arxiv.org/)) and peer-reviewed journals.
- arXiv submission rules apply: keep the LaTeX source **as simple as possible**.
  - Writing everything in a **single, large `.tex` file** is acceptable and is actually the recommended approach for arXiv submissions — it eliminates include/input path issues and simplifies the upload package.
  - Avoid deep folder structures or complex multi-file layouts when targeting arXiv.
- arXiv compiler environment (as of 2025):
  - **TeX Live 2025** by default; **TeX Live 2023** available as an alternative.
  - **pdflatex** is the standard engine for PDF-mode LaTeX on arXiv.
  - **xelatex** is also supported if explicitly selected during submission.
  - **lualatex** is NOT currently supported on arXiv — do not use it for arXiv submissions.

### Personal Math Books and Booklets
- Writing books or booklets for personal use (e.g., Year 11/12 maths, university-level mathematics).
- These documents are **not submitted to any journal or preprint server** — no arXiv constraints apply.
- Prefer **lualatex** for these documents: better Unicode support, native font handling via `fontspec`, and modern engine features.
- Multi-file structure (separate chapters, a `macros.tex`, etc.) is appropriate and encouraged here.

### General Applicability
- Document preambles and package configurations.
- Macros and custom commands.
- Figure, table, and listing environments.
- Bibliography and citation management.
- Class files and custom style files.

---

## Compiler and Engine Selection

| Context | Engine | Notes |
|---|---|---|
| arXiv / journal submission | `pdflatex` | Default and most compatible. Use unless xelatex is specifically required. |
| arXiv / journal submission | `xelatex` | Supported on arXiv; select explicitly during submission. |
| Personal books / booklets | `lualatex` | Preferred for math-heavy personal documents; not available on arXiv. |

- Use `\usepackage[utf8]{inputenc}` and `\usepackage[T1]{fontenc}` with **pdflatex**.
- Use `\usepackage{fontspec}` (and drop `inputenc`/`fontenc`) with **xelatex** or **lualatex**.
- Never use `lualatex` for documents targeting arXiv.

---

## Build Environment

Docker-based builds are preferred for reproducibility and OS independence.

**Primary OS:** macOS (default development machine).

**Preferred Docker images:**

| Image | Use Case |
|---|---|
| `texlive/texlive` | Full TeX Live environment; general purpose. |
| `dallay/texlive:2025` | Pinned TeX Live 2025; matches arXiv's default compiler environment. |

**Example Docker build command:**

```bash
docker run --rm -v "$(pwd)":/workspace -w /workspace dallay/texlive:2025 \
  pdflatex main.tex
```

- Always mount the working directory into the container rather than copying files.
- Use the pinned `dallay/texlive:2025` image when verifying arXiv compatibility.
- Use `texlive/texlive` (latest) for personal lualatex documents where the newest engine features matter.

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
- Choose the encoding/font setup based on the target engine:
  - **pdflatex** (arXiv/journals): use `inputenc` with `utf8` and `fontenc` with `T1`.
  - **xelatex / lualatex** (personal documents): use `fontspec` instead; omit `inputenc` and `fontenc`.
- Always load `hyperref` last among packages unless another package explicitly requires otherwise.
- Use `\usepackage{microtype}` for improved typography on final documents.

**pdflatex preamble (arXiv / journal submissions):**

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

**lualatex preamble (personal math books/booklets):**

```latex
\documentclass[12pt, a4paper]{book}
\usepackage{fontspec}
\usepackage{geometry}
\usepackage{amsmath, amssymb, unicode-math}
\usepackage{graphicx}
\usepackage{booktabs}
\usepackage{hyperref}
\usepackage{microtype}
```

### Macros and Commands
- Define reusable macros with `\newcommand{\macroname}[num-args]{definition}`.
- Use `\renewcommand` only when overriding an existing definition intentionally; add a comment explaining why.
- Parameterize repeated notation: paper titles, author names, dataset names, frequently-used math symbols.
- **arXiv / single-file documents:** place all macro definitions in a clearly marked block near the top of the `.tex` file (after the preamble, before `\begin{document}`). Do not use `\input{macros}` in single-file arXiv submissions.
- **Personal books / multi-file projects:** keep macro definitions in a separate file (e.g., `macros.tex`) and include it with `\input{macros}`.
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
- **For branded figures using custom colors**, use the `\definecolor` block from [`design/COLOR-PALETTE-POLICY.md`](../design/COLOR-PALETTE-POLICY.md) as the source of truth for color names and RGB values.

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

> All `.bib` file rules — required fields, entry key conventions, case protection, canonical examples, and the validation checklist — are defined in **[`LATEX-POLICY-bib-files.md`](LATEX-POLICY-bib-files.md)**. Follow that policy for everything related to bibliography file content.

The LaTeX-side wiring for bibliography:
- Load BibLaTeX with Biber: `\usepackage[backend=biber, style=authoryear]{biblatex}` (adjust `style` to the venue's requirements).
- Declare the bibliography file: `\addbibresource{references.bib}`.
- Cite with `\cite{}`, `\parencite{}`, or `\textcite{}` depending on context.
- Print the bibliography with `\printbibliography`.
- Use BibTeX (legacy `\bibliography{}` / `\bibliographystyle{}`) only when the journal template explicitly requires it.

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
- Overriding LaTeX spacing manually with `\vspace`, `\hspace`, `\\` except as a last resort.
- Violating any `.bib` file rule — see [`LATEX-POLICY-bib-files.md`](LATEX-POLICY-bib-files.md) for the full list.
- **Using `lualatex` for arXiv submissions** — it is not supported on arXiv's build servers.
- **Using `\input{}` / `\include{}` for multi-file splits in arXiv submissions** — a single `.tex` file is simpler and less error-prone when uploading to arXiv.
- Building LaTeX locally without Docker when reproducibility across machines matters — always prefer the Docker workflow.

---

## Output Expectations

Generated LaTeX code must:
- Compile without errors on a standard TeX Live installation (via Docker when applicable).
- Use `\newcommand` for all custom macros.
- Use `\label` and `\ref` for all cross-references.
- Use `booktabs` for tables.
- Load `hyperref` last in the preamble.
- Use BibLaTeX/Biber for bibliography (or BibTeX only when the journal template mandates it); satisfy all `.bib` content rules per [`LATEX-POLICY-bib-files.md`](LATEX-POLICY-bib-files.md).
- Not hardcode cross-reference numbers.
- Include `\caption` and `\label` on all float environments.
- For arXiv submissions: be contained in a **single `.tex` file** with no `\input{}`/`\include{}` splits.
- Use the engine appropriate to the context (`pdflatex` for arXiv, `lualatex` for personal books).

---

## Optional Checklist

### All Documents
- [ ] Document class and options explicitly specified.
- [ ] Correct engine selected for the target context (pdflatex / xelatex / lualatex).
- [ ] `hyperref` loaded last in preamble.
- [ ] `microtype` included for final documents.
- [ ] All sections, figures, tables, and equations have `\label`.
- [ ] All cross-references use `\ref{}`, `\eqref{}`, or `\autoref{}`.
- [ ] Figures use `\includegraphics` with vector format.
- [ ] Tables use `booktabs`; no vertical lines.
- [ ] Non-breaking spaces before `\ref` and `\cite`.
- [ ] All `.bib` file rules satisfied — run through the checklist in [`LATEX-POLICY-bib-files.md`](LATEX-POLICY-bib-files.md).

### arXiv / Journal Submissions
- [ ] Single `.tex` file (no `\input{}` / `\include{}` splits).
- [ ] Engine is `pdflatex` (or `xelatex` if explicitly required).
- [ ] All macro definitions placed in a block before `\begin{document}`.
- [ ] Document compiles cleanly with `dallay/texlive:2025` Docker image.
- [ ] Bibliography wired correctly (BibLaTeX/Biber or BibTeX per journal); `.bib` content validated per [`LATEX-POLICY-bib-files.md`](LATEX-POLICY-bib-files.md).

### Personal Books / Booklets
- [ ] Engine is `lualatex`.
- [ ] `fontspec` loaded; `inputenc`/`fontenc` omitted.
- [ ] Macros in a separate `macros.tex` file, included with `\input{macros}`.
- [ ] Multi-chapter structure with `\include{}` per chapter where appropriate.
- [ ] Document compiles cleanly with `texlive/texlive` Docker image.
