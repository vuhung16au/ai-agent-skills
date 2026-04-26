# PYTHON-POLICY-general.md

## Purpose

Define standards for writing clean, readable, and maintainable Python code across general-purpose scripts, libraries, and data/ML projects. This policy governs agent behavior when generating or modifying Python code.

---

## Scope / When to Use

Apply this policy to all Python code generation, including:
- CLI scripts and automation tools.
- Reusable library modules and packages.
- Data processing pipelines and analysis scripts.
- Backend services and APIs written in Python.
- ML/AI project scaffolding (use domain-specific ML policy for deeper ML rules).

---

## Core Principles

- **Readability is non-negotiable.** Code is read more than it is written.
- **Explicit over implicit.** Follow the Zen of Python.
- **Type annotations are expected** for public APIs, function signatures, and complex data structures.
- **Fail fast.** Validate inputs early and surface errors clearly.
- **Minimal dependencies.** Only add third-party packages when the standard library is insufficient.

---

## Required Behavior

### Project Structure
- Use `src/` layout for installable packages: `src/<package_name>/`.
- Separate entry points (`__main__.py` or CLI scripts) from library logic.
- Place tests in a top-level `tests/` directory mirroring the `src/` structure.
- Include a `pyproject.toml` (preferred) or `setup.cfg` for packaging metadata.
- Include a `requirements.txt` or `pyproject.toml` dependency list; never hardcode dependency installation inside application code.

### Virtual Environments
- Always assume code will run in a virtual environment (`venv`, `conda`, or `uv`).
- Never instruct agents to run `pip install` globally without `--user` or inside an activated environment.
- Include environment setup instructions in the project README.

### Typing
- Annotate all function parameters and return types for public functions.
- Use `Optional[T]` or `T | None` (Python 3.10+) for nullable values.
- Use `TypedDict`, `dataclass`, or `Pydantic` models for structured data instead of plain dicts where clarity matters.
- Avoid `Any` unless interfacing with dynamically typed external systems; document why when used.

### Naming Conventions
- `snake_case` for variables, functions, and module names.
- `PascalCase` for class names.
- `UPPER_SNAKE_CASE` for module-level constants.
- Prefix private attributes and methods with a single underscore (`_`).
- Do not use double underscores (`__`) for name-mangling unless intentional.

### Code Style
- Follow PEP 8. Use a formatter (Black or Ruff) to enforce it automatically.
- Maximum line length: 88 characters (Black default) or as configured in `pyproject.toml`.
- One import per line; group imports as: standard library, third-party, local.
- Do not use wildcard imports (`from module import *`).

### Error Handling
- Catch specific exceptions, not bare `except:` or `except Exception:` without re-raising or logging.
- Use custom exception classes for domain-specific errors.
- Always log or surface exceptions in a way that provides enough context for debugging.
- Avoid swallowing exceptions silently.

### Scripts vs. Libraries
- **Scripts**: Use `if __name__ == "__main__":` guards. Accept arguments via `argparse` or `typer`, not hardcoded values.
- **Libraries**: Do not print to stdout; raise exceptions or return error values. Do not call `sys.exit()` inside library functions.
- **Both**: Do not hardcode paths, credentials, or environment-specific values.

### Data and ML Friendliness
- Use `pathlib.Path` for all file system operations, not `os.path`.
- Prefer generator expressions over list comprehensions for large data sequences.
- Use context managers (`with` statements) for file I/O and resource management.
- For data frames, document expected schema (columns, dtypes) at the point of creation or ingestion.

### Documentation
- Write docstrings for all public modules, classes, and functions.
- Use Google-style or NumPy-style docstrings consistently within a project.
- Include parameter types, return types, and raised exceptions in docstrings when not obvious from type annotations.

### Testing
- Write tests with `pytest`.
- Aim for unit tests on pure functions and integration tests on I/O-heavy code.
- Use `pytest.fixture` for shared setup; avoid relying on global state.
- Mock external services and file I/O in unit tests.

---

## Things to Avoid

- Mutable default arguments (e.g., `def f(items=[])`).
- `print()` statements in library code intended for production.
- Hardcoded credentials, tokens, or secrets anywhere in the code.
- Using `assert` for runtime validation in production code (use explicit `if`/`raise` instead).
- Deeply nested logic; flatten with early returns or helper functions.
- Unused imports or variables.
- Mixing I/O and business logic in the same function.

---

## Output Expectations

Generated Python code must:
- Be syntactically valid and runnable without modification on Python 3.10+.
- Include type annotations on all public functions.
- Follow PEP 8 formatting.
- Include docstrings for public interfaces.
- Not contain hardcoded environment-specific paths or credentials.
- Be testable in isolation where possible.

---

## Optional Checklist

- [ ] All public functions have type annotations.
- [ ] All public functions have docstrings.
- [ ] No wildcard imports.
- [ ] No mutable default arguments.
- [ ] No bare `except:` clauses.
- [ ] File I/O uses `pathlib.Path`.
- [ ] Scripts use `if __name__ == "__main__":`.
- [ ] Tests exist in `tests/` and run with `pytest`.
