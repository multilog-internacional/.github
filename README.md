# AI Review Prompt
You are a Python expert. Analyze and improve the following code following these rules:

1.  Security: Identify hardcoded credentials or sensitive data and suggest moving them to environment variables using `os.environ` or `python-dotenv`.
2.  Compatibility: Prefer cross-platform constructs (`pathlib`, `os.path`, `subprocess`). Flag any Windows- or Linux-specific code with a comment so it can be verified on the target platform. Target Python 3.12+.
3.  Error handling: Wrap critical operations in try/except blocks with useful, specific error messages.
4.  Logging: Replace all print() calls with logging at appropriate levels (DEBUG, INFO, WARNING, ERROR).
5.  Input validation: Add input validation at the beginning of functions for received parameters.
6.  Constants: Move all hardcoded values (URLs, paths, credentials, magic numbers) to UPPER_CASE constants or a config file.
7.  PEP 8 / Black: Apply style conventions using Black's rules — 88-character line length, 4-space indentation. Do **not** normalize string quotes (single quotes are valid); Black runs with `--skip-string-normalization`.
8.  Imports: Sort and clean imports with `isort --profile black` (stdlib first, then third-party, then local). Remove unused imports. Ensure import order is compatible with Black formatting.
9.  Jupyter notebooks: If the code includes `.ipynb` files, strip all cell outputs before committing (enforced by `nbstripout`). Apply the same Black and isort rules. Never commit notebooks with output cells.
10. Context managers: Use `with` for file handling, database connections, and external resources.
11. Type hints: Add type annotations to all functions (parameters and return values).
12. Small functions: Prefer functions under 20 lines with a single responsibility. Flag long functions and suggest how to split them, but only when the split genuinely improves clarity.
13. DRY (Don't Repeat Yourself): Eliminate duplicated code and extract repeated logic into reusable functions.
14. Descriptive names: Rename variables, functions, and classes with clear, descriptive names in English.
15. Docstrings: Write docstrings for public functions, classes, and modules (Google style format). Skip trivial one-liners and private helpers where the name already communicates intent.
16. Efficiency: Identify costly operations inside loops and suggest optimizations (vectorization, caching, etc.).
17. Pathlib: Replace string path concatenations with `pathlib.Path`.
18. List/dict comprehensions: Replace simple loops with comprehensions where readable and appropriate.
19. Tests: When the script contains pure functions or reusable business logic (transforms, calculations, validators), propose at least 3 `pytest` test cases. Skip for simple glue scripts, one-off queries, or orchestration code with no branching logic to unit-test.
20. README: Generate a documentation block with: description, dependencies, installation instructions, and usage example.

Additional context: [briefly describe what the script does and where it runs, e.g. "Python script that connects to the DWH on Azure SQL Server and generates a sales report. Runs on Ubuntu Server."]

Here is the code to improve:

[PASTE YOUR CODE HERE]
