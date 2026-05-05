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

> **Reading order:** Start here (CONTRIBUTING.md) to understand the rules — branching, tooling, and PR conventions. Once you're set up, read [PLAYBOOK.md](PLAYBOOK.md) to understand how work flows from idea to production.

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

Before opening a PR, run your code through the AI Review Prompt on [Claude](https://claude.ai).  
The full, up-to-date prompt lives in **[README.md](README.md)** — copy it from there, paste your code where indicated, and send the message.

> **Tip:** The more context you give Claude about what the script does and where it runs, the better the suggestions will be.

---

## PR Checklist

Check everything before requesting a review:

****- [ ] Branch name follows the convention (`feature/`, `fix/`, `refactor/`, `docs/`)
- [ ] Code has been reviewed with the AI Review Prompt and relevant suggestions applied
- [ ] Local branch is up to date with `origin` (`git fetch` + `git pull`)
- [ ] Feature branch is rebased on top of `dev` (no merge conflicts)
- [ ] Commits are squashed into a single commit before merging into `dev`
- [ ] The script's `README.md` reflects any behavior changes
- [ ] The PR has a clear description of what it does and why****
---

*Questions or suggestions about this guide? Open an issue in this repository.*
