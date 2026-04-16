# AI Review Prompt
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
17. Compatibility: Ensure the code runs on Python 3.10+ and is compatible with both Windows and Ubuntu.
18. Tests: Propose at least 3 test cases using `pytest` for the main functions.
19. README: Generate a documentation block with: description, dependencies, installation instructions, and usage example.
20. Final summary: At the end, present a table with: what was changed, why, and the expected impact.

Additional context: [briefly describe what the script does and where it runs, e.g. "Python script that connects to the DWH on Azure SQL Server and generates a sales report. Runs on Ubuntu Server."]

Here is the code to improve:

[PASTE YOUR CODE HERE]
