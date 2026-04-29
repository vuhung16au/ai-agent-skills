# AI Agent Skills

A curated library of reusable **SKILL.md**-style files and **policy files** for AI coding agents — including GitHub Copilot, GitHub Codex, Claude, and Cursor. Covers practical software engineering, academic/research workflows, and design/development best practices.

---

## What Are Skills and Policies?

| Term | Meaning |
|------|---------|
| **SKILL** | A focused instruction file teaching an agent how to use a specific technology, library, or workflow (e.g., LangChain, LangGraph). |
| **POLICY** | A set of rules and standards an agent must follow when generating code or content in a given domain (e.g., Python style, C# naming, LaTeX and bibliography files, color palettes, responsive design). |

Both file types are written in Markdown and designed to be:
- Pasted directly into `AGENTS.md`, `.github/copilot-instructions.md`, or system prompts.
- Referenced as `SKILL.md` files in supported agent workflows.
- Used as reusable prompt assets during code generation sessions.

---

## Intended Use

| Agent / Tool | How to Use |
|---|---|
| **GitHub Copilot** | Paste into `.github/copilot-instructions.md` or reference in workspace settings. |
| **Claude** | Include as a system prompt or user-turn context block. |
| **GitHub Codex** | Reference in `AGENTS.md` at the repo root or subdirectory level. |
| **Cursor** | Add to `.cursorrules` or use as a prompt prefix in chat. |
| **General agents** | Paste the relevant file content into any system prompt or instruction block. |

---

## Folder Structure

```
ai-agent-skills/
├── README.md
├── ai/
│   ├── LANGCHAIN-SKILL.md
│   ├── LANGGRAPH-SKILL.md
│   └── ML-POLICY-general.md
├── design/
│   ├── COLOR-PALETTE-POLICY.md
│   └── INTERACTION-DESIGN-POLICY.md
├── desktop/
│   └── ELECTRONJS-POLICY.md
├── dotnet/
│   ├── CSharp-POLICY.md
│   ├── DOTNET-BLAZOR-POLICY.md
│   ├── DOTNET-MAUI-POLICY.md
│   └── DOTNET-RAZOR-POLICY.md
├── figures/
│   └── FIGURES-POLICY-academic-papers.md
├── java/
│   └── SPRINGBOOT-BACKEND-API-POLICY.md
├── latex/
│   ├── LATEX-POLICY-bib-files.md
│   ├── LATEX-POLICY-general.md
│   └── LATEX-SKILL-tikz-pgfplots-groupplots.md
├── python/
│   └── PYTHON-POLICY-general.md
├── testing/
│   └── PLAYWRIGHT-TESTING-POLICY.md
└── web/
    ├── NEXTJS-TYPESCRIPT-POLICY.md
    └── RESPONSIVE-DESIGN-POLICY.md
```

---

## Naming Conventions

- **SKILL files**: `<TECHNOLOGY>-SKILL.md` — for tool-specific how-to guidance.
- **POLICY files**: `<TECHNOLOGY>-POLICY[-scope].md` — for standards and rules agents must follow.
- Use **UPPERCASE-WITH-HYPHENS** for the primary identifier portion.
- Use **kebab-case** for any scope qualifiers (e.g., `-general`, `-academic-papers`, `-bib-files`).
- Category folders use lowercase (e.g., `python/`, `dotnet/`, `web/`).

---

## Contribution Guidelines

1. **One concern per file.** Each file should address a single technology, tool, or domain.
2. **Follow the standard structure**: Title → Purpose → Scope → Core Principles → Required Behavior → Things to Avoid → Output Expectations.
3. **Be actionable.** Prefer explicit rules over vague recommendations.
4. **No placeholders.** All content must be useful on day one — no lorem ipsum, no stubs.
5. **Add your file to the correct category folder.** Create a new category folder if none of the existing ones fit.
6. **Update this README** when adding a new category folder or a file worth highlighting.
7. **Keep tone professional and direct.** These files are consumed by agents, not by marketing teams.

---

## License

See [LICENSE](./LICENSE).
