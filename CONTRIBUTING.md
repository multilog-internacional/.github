# Contributing Guide — Multilog Internacional

Welcome to the development standards repository for **Multilog Internacional**.  
This document defines how to write, review, and improve Python code within the organization.

---

## Table of Contents

1. [General Philosophy](#general-philosophy)
2. [Stack & Environments](#stack--environments)
3. [Project Structure](#project-structure)
4. [Git Workflow](#git-workflow)
5. [AI Review Prompt](#ai-review-prompt)
6. [PR Checklist](#pr-checklist)

---

## General Philosophy

- Code is written **once** but read **many times**. Prioritize readability.
- If something is not documented, it does not exist.
- Scripts that "only I understand" are technical debt.
- Everything that goes to production goes through review.

---

## Stack & Environments

| Tool             | Minimum Version | Notes                                      |
|------------------|-----------------|--------------------------------------------|
| Python           | 3.12+           | Compatible with Ubuntu and Windows         |
| GitHub           | —               | Organization: `multilog-internacional`     |
| Virtual env      | conda / venv    | Always use an isolated environment         |

---

## Project Structure

```
my-project/
├── data/                 # Input, output, or reference data (do not commit sensitive data)
├── env_setup/            # Scripts and instructions to set up the environment
├── notebooks/            # Jupyter notebooks for exploration and analysis
├── src/                  # Main source code (modules and scripts)
├── pyproject.toml        # Project configuration and dependencies
├── setup.py              # Package installation
├── LICENSE               # Project license
└── README.md             # Description, installation, and usage
```

---

## Git Workflow

```
main
 └── dev
      └── feature/descriptive-name     ← work here
      └── fix/bug-description
```

1. Always branch off `dev`, never from `main`
2. Commits must be in English, never Spanish except very special cases
3. Commit messages should describe **what** and **why**, not **how**
4. Open a PR toward `dev` when you're done
5. At least one review required before merging

**Examples of good commits:**
```
feat: add weekly sales report by branch
fix: correct division by zero error in margin calculation
refactor: extract connection logic into a separate module
```

---

## AI Review Prompt

Before opening a PR, run your code through this prompt on [Claude](https://claude.ai).  
Copy the full prompt, paste your code where indicated, and send the message.

```
You are a Python expert. Analyze and improve the following code following these 20 rules:

1.  PEP 8: Apply style conventions (naming, spacing, 4-space indentation).
2.  Descriptive names: Rename variables, functions, and classes with clear, descriptive names in English.
3.  Type hints: Add type annotations to all functions (parameters and return values).
4.  Docstrings: Write docstrings for all functions, classes, and modules (Google style format).
5.  Error handling: Wrap critical operations in try/except blocks with useful, specific error messages.
6.  Logging: Replace all print() calls with logging at appropriate levels (DEBUG, INFO, WARNING, ERROR).
7.  Constants: Move all hardcoded values (URLs, paths, credentials, magic numbers) to UPPER_CASE constants or a config file.
8.  Small functions: Break functions longer than 20 lines into smaller functions with a single responsibility.
9.  DRY (Don't Repeat Yourself): Eliminate duplicated code and extract repeated logic into reusable functions.
10. Imports: Sort and clean imports (stdlib first, then third-party, then local). Remove unused imports.
11. Context managers: Use `with` for file handling, database connections, and external resources.
12. List/dict comprehensions: Replace simple loops with comprehensions where readable and appropriate.
13. Pathlib: Replace string path concatenations with `pathlib.Path`.
14. Input validation: Add input validation at the beginning of functions for received parameters.
15. Security: Identify hardcoded credentials or sensitive data and suggest moving them to environment variables using `os.environ` or `python-dotenv`.
16. Efficiency: Identify costly operations inside loops and suggest optimizations (vectorization, caching, etc.).
17. Compatibility: Ensure the code runs on Python 3.12+ and is compatible with both Windows and Ubuntu.
18. Tests: Propose at least 3 test cases using `pytest` for the main functions.
19. README: Generate a documentation block with: description, dependencies, installation instructions, and usage example.
20. Final summary: At the end, present a table with: what was changed, why, and the expected impact.

Additional context: [briefly describe what the script does and where it runs, e.g. "Python script that connects to the DWH on Azure SQL Server and generates a sales report. Runs on Ubuntu Server."]

Here is the code to improve:

[PASTE YOUR CODE HERE]
```

> **Tip:** The more context you give Claude about what the script does and where it runs, the better the suggestions will be.

---

## PR Checklist

Check everything before requesting a review:

- [ ] Ran the AI prompt and applied the relevant improvements
- [ ] Code has type hints and docstrings
- [ ] No hardcoded credentials or sensitive data
- [ ] At least one basic test exists in `tests/`
- [ ] The project `README.md` is up to date
- [ ] Code runs without errors on my local machine
- [ ] Branch name follows the convention (`feature/`, `fix/`, `refactor/`)
- [ ] The PR has a clear description of what it does and why

---

*Questions or suggestions about this guide? Open an issue in this repository.*
