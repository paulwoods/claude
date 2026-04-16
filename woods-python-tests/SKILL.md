---
name: woods-python-tests
description: Activates a Python virtual environment and runs the pytest suite for the current project, then displays a clear summary of results. Use this skill whenever the user asks to run tests, run pytest, check tests, verify the test suite, or wants to know if tests pass — even if they say it casually like "run the tests" or "do the tests pass?".
---

# Woods Python Tests

Locate the virtual environment and run the pytest suite for the current Python project.

## Steps

1. **Find the venv.** Look for a `.venv` directory in the current project. On Windows (bash), activate with:
   ```bash
   source .venv/Scripts/activate
   ```
   On Linux/macOS:
   ```bash
   source .venv/bin/activate
   ```
   If no `.venv` exists, fall back to the system `python` / `python3`.

2. **Find the tests.** Look for a `tests/` directory. If none exists, check for `test/` or files matching `test_*.py`. Pass the discovered path to pytest.

3. **Run pytest:**
   ```bash
   python -m pytest <tests-dir> -v 2>&1
   ```

4. **Display a concise summary:**
   ```
   **Tests**
   - Result: X passed / Y failed / Z errors
   - Warnings: [list any warnings briefly, or "none"]
   - Duration: Xs
   ```
   If there are failures, show the failing test names and their error messages so the user can act on them immediately.

## Notes

- Always prefer `python -m pytest` over a bare `pytest` call so the venv's interpreter is used consistently.
- On Windows the venv activate script is at `.venv/Scripts/activate`; on Unix it is `.venv/bin/activate`.
- Allow up to 5 minutes before declaring a hang.

