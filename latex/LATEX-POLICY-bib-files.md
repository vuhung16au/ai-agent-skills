# LATEX-POLICY-bib-files.md

## Purpose

Define standards for `.bib` bibliography files used in LaTeX academic papers. This policy governs how bibliography files are structured, named, and validated.

---

## Scope / When to Use

Apply this policy when:
- Creating or editing `.bib` files for any LaTeX document.
- Adding new references to an existing `.bib` file.
- Reviewing or auditing bibliography entries in a LaTeX project.

---

## Core Rules

### 1. Every `.tex` File Must Have a Dedicated `.bib` File

- Each LaTeX manuscript must reference its own `.bib` file — do not share a single `.bib` across multiple unrelated papers.
- The `.bib` file must be named to match or relate clearly to the paper (e.g., `references.bib`, `paper-title.bib`).
- Declare the bibliography file with `\addbibresource{references.bib}` (BibLaTeX) or `\bibliography{references}` (legacy BibTeX).
- The `.bib` file must live in the same directory as the main `.tex` file, or in a clearly documented `bib/` subdirectory.

### 2. Required Fields for Every Entry

Every `.bib` entry — regardless of type — **must** include all of the following non-blank fields:

| Field      | Notes |
|------------|-------|
| `author`   | Full author names; use `and` to separate multiple authors. |
| `title`    | Full title; wrap proper nouns and acronyms in `{}` to protect case. |
| `year`     | Publication year as a four-digit integer. |
| `url`      | Publicly accessible URL to the work. |
| `doi`      | Digital Object Identifier, if one exists. Omit only if the work genuinely has no DOI (e.g., informal tech reports, preprints without one). |
| `abstract` | A copy of the work's abstract; used for quick review and traceability. |

**Never leave any of these fields empty or omit them.** If a DOI does not exist, add a comment explaining why:

### Optional but Encouraged Fields

| Field      | Notes |
|------------|-------|
| `keywords` | Comma-separated terms describing the work's topic, method, or domain. Especially valuable in systematic reviews for filtering and grouping entries by theme. Example: `keywords = {intelligent tutoring systems, K-12 education, systematic review, AI in education}`. |

Add `keywords` whenever the source paper provides them, or when you can assign meaningful thematic tags. Omit only when the entry genuinely resists categorisation (e.g., a broad reference work).

```bibtex
% No DOI assigned — informal technical report
```

### 3. Entry Key Convention

- Use the format `firstauthorlastnameYYYY` (all lowercase, no spaces), e.g., `popay2006`, `ahmad2025`.
- If two works share the same key, append a letter: `smith2023a`, `smith2023b`.

### 4. Case Protection

Wrap proper nouns, acronyms, and mixed-case terms in `{}` to prevent BibTeX/BibLaTeX from lowercasing them:

```bibtex
title = {A systematic review of {AI}-driven intelligent tutoring systems ({ITS}) in {K-12} education},
```

### 5. Author Formatting

- List all authors; do not truncate with "et al." inside the `.bib` file.
- Separate authors with ` and ` (not commas at the top level).
- Use `{Lastname, Firstname}` or `Firstname Lastname` consistently throughout the file.

---

## Canonical Entry Examples

### Journal Article

```bibtex
@article{ahmad2025,
  author   = {Ahmad, M. F. and others},
  title    = {A systematic review of {AI}-driven intelligent tutoring systems ({ITS}) in {K-12} education},
  journal  = {Frontiers in Education},
  year     = {2025},
  url      = {https://pmc.ncbi.nlm.nih.gov/articles/PMC12078640/},
  doi      = {10.3389/feduc.2025.0000000},
  abstract = {The use of artificial intelligence in education ({AIEd}) has grown exponentially in the last decade...},
  keywords = {intelligent tutoring systems, K-12 education, systematic review, AI in education}
}
```

### Technical Report

```bibtex
@techreport{popay2006,
  author      = {Popay, Jennie and Roberts, Helen and Sowden, Amanda and Petticrew, Mark and Arai, Lisa and Rodgers, Mark and Britten, Nicky and Roen, Katharine and Duffy, Steven},
  title       = {Guidance on the Conduct of Narrative Synthesis in Systematic Reviews: A Product from the {ESRC} Methods Programme},
  institution = {Economic and Social Research Council},
  year        = {2006},
  address     = {Lancaster, UK},
  url         = {https://www.lancaster.ac.uk/media/lancaster-university/content-assets/documents/fhm/dhr/chir/NSsynthesisguidanceVersion1-April2006.pdf},
  % No DOI assigned — informal technical report
  abstract    = {This guidance document provides a framework for conducting narrative synthesis...},
  keywords    = {narrative synthesis, systematic review, qualitative evidence, ESRC}
}
```

---

## Things to Avoid

- Leaving `author`, `title`, `year`, `url`, `doi`, or `abstract` blank or absent.
- Using a shared `.bib` file across multiple unrelated papers.
- Hardcoding "et al." in the `author` field.
- Using inconsistent entry key formats (e.g., mixing `Smith2023` and `smith_2023`).
- Unprotected acronyms or proper nouns in titles that will be silently lowercased.

---

## Validation Checklist

Before submitting or committing a `.bib` file:

- [ ] Every entry has a non-blank `author` field.
- [ ] Every entry has a non-blank `title` field with proper `{}` case protection.
- [ ] Every entry has a `year` field (four-digit integer).
- [ ] Every entry has a `url` field pointing to a live, accessible source.
- [ ] Every entry has a `doi` field, or a comment explaining its absence.
- [ ] Every entry has a non-blank `abstract` field.
- [ ] Entries have a `keywords` field where meaningful thematic tags can be assigned.
- [ ] Entry keys follow the `firstauthorlastnameYYYY` convention.
- [ ] No authors are truncated with "et al."
- [ ] The `.bib` file is dedicated to this paper/project only.
