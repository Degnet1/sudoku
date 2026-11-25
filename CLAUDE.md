# SEN4013 V&V Project: Sudoku Generator and Solver

This file gives the context, rules, plan and current status of our SEN4013 Software Verification and Validation group project. Keep it in the root of our fork so that teammates and any AI assistant (for example Claude Code in VS Code) start from the same information. Update the **Status** section as work progresses.

## Project at a glance

| | |
|---|---|
| Course | SEN4013 Software Verification and Validation |
| Software under test | Sudoku Generator and Solver, https://github.com/datahappy1/sudoku (upstream last commit 2 Nov 2025) |
| Group | Defne Doğan (2202877), Ezgi Doğanözer (2202046), Elif Ceylan (2366191) |
| Final deadline | **11 December 2026, 17:30**: Part 1 PDF, Part 3 report PDF and Part 4 video, as three separate files on Teams |
| Our target | Submit on **10 December**; late work loses 10% per day |
| Grading | Part 1 tools review 35 · Part 2 V&V activities 35 · Part 3 report 20 · Part 4 video 10 |

**What the software does:** a Python command-line program that generates Sudoku puzzles at three levels (easy, medium, hard) and solves puzzles read from a text file of 9 lines × 9 digits, where 0 is an empty cell.

**Our test oracle:** a solved grid is correct when every row, column and 3×3 box contains 1–9 exactly once, and the original given numbers are unchanged. We can judge any output without knowing the answer in advance.

## Rules for working on this project

### Course AI policy (brief, section 7)
- AI may support the work: summarising documentation, brainstorming test ideas, improving report language.
- The **V&V approach, the test case design and the analysis of results must reflect our own understanding.** Review and validate any AI output before relying on it.
- We are fully responsible for correctness, originality and academic integrity.
- We must submit a **Student AI Declaration** listing every AI tool used and its purpose.

### Instructions for AI assistants working in this repo
- **Explain why**, not just what: why a test is designed a certain way, why something broke.
- Give **one clear recommendation**, not a list of options. If unsure, say so instead of guessing.
- **Before making any change:** read the relevant files first, state the plan, and **wait for approval**.
- **Never install packages, delete files or change the program's design without asking first.**
- Help us understand and review tests; do not write the whole test suite for us. Test designs must be ours (AI policy above).
- Do not modify the code under `sudoku/` unless we have explicitly decided to fix a specific incident (see Step 4).
- Respond in English.

## How to run things

All commands run from the **project root** with the virtual environment active.

```bash
# Setup (once)
python -m venv venv
source venv/bin/activate          # Windows PowerShell: venv\Scripts\Activate.ps1
pip install pytest pytest-cov hypothesis

# Run the program. Use -m: the README's "python sudoku\sudoku_game.py" fails (INC-01)
python -m sudoku.sudoku_game -a generate -l hard -p 1
python -m sudoku.sudoku_game -a solve -f files/sudoku_hard.txt -p 1
```

**Existing tests** pass only when run from inside `tests/` (INC-02):

```bash
cd tests
PYTHONPATH=.. python -m pytest                     # Windows: $env:PYTHONPATH=".."; python -m pytest
PYTHONPATH=.. python -m pytest --cov=sudoku --cov-branch --cov-report=term-missing --cov-report=html
```

Command-line options: `-a` action (`solve` / `generate`), `-l` level (`easy` / `medium` / `hard`), `-f` puzzle file (for solve), `-p` prettify output (default `false`).

## Code facts that matter for testing

| Module | What it does | Notes for testers |
|---|---|---|
| `sudoku/sudoku_game.py` | Command-line entry point, option handling, reads the puzzle file | Only ~47% covered at baseline |
| `sudoku/solver.py` | `Solver.solve_sudoku()` | **Prints the solved grid and returns only the number of tries** (or `None` if no solution). Capture output with pytest's `capsys`. Stops with `TooManyTries` after `MAX_ATTEMPTS = 1000000`: monkeypatch it to a small number in tests |
| `sudoku/generator.py` | Builds a full grid, then hides cells per level | Uses `random`: call `random.seed(...)` in tests for repeatable results. Hidden cells per row: easy 3, medium 4–5, hard 5–8 |
| `sudoku/grid.py` | Grid helpers (rows, columns, 3×3 squares) | Has a rarely reached `IndexError` branch |
| `sudoku/exceptions.py` | `InvalidGridItem`, `InvalidGridShape`, `TooManyTries` | |
| `sudoku/level.py`, `action_type.py`, `printer.py` | Enums and output formatting | |

Size: ~25 functions in 7 modules with logic, 37 decision points, 62 branch outcomes measured by coverage. Most complex functions: `_add_row_mask` (generator), `_solve_grid` (solver), `run_sudoku_solver` (game), `get_square_from_position` (grid).

**Baseline (measured 2026-11-25 by Defne, Python 3.14.2, commit `d1b37f1`):** 16 existing tests, all passing when run from `tests/` (9 fail from the root); **82% branch coverage overall (221 statements, 62 branch outcomes), 47% for `sudoku_game.py`**. Full per-module numbers, raw outputs and the HTML report are in `baseline/` (see `baseline/README.md`). Do not overwrite that folder: it is the "before" evidence.

## Incident log

Template: **ID · description · steps to reproduce · expected vs actual · cause · status (open / resolved) · regression test**

| ID | Description | Cause | Status |
|---|---|---|---|
| INC-01 | README run command `python sudoku\sudoku_game.py …` fails with `ModuleNotFoundError: No module named 'sudoku'` | Running the file directly puts `sudoku/` on the import path, not the project root | Open (workaround: `python -m sudoku.sudoku_game`). Reproduced 2026-11-25, `baseline/01_…` |
| INC-02 | 9 of 16 existing tests fail with `FileNotFoundError` when run from the project root | Tests build paths from `os.getcwd()` (`test_functional_solve.py:29` → `files/`, `test_end_to_end.py:25` → `temp/`); those folders exist only inside `tests/` | Open (candidate fix). Reproduced 2026-11-25, `baseline/04_…` |
| INC-03 | A grid of only 1s is reported as "solved" in 1 try | Given numbers are never checked against Sudoku rules | Open |
| INC-04 | A puzzle with two 5s in one row runs 1,000,000 attempts, then crashes with `TooManyTries` | Same root cause as INC-03 | Open |
| INC-05 | An unsolvable puzzle prints "Finished in … in None tries" with no error message | Solver returns `None`; the caller doesn't handle it | Open |
| INC-06 | A valid puzzle written with spaces (`0 0 0 5 …`) is rejected as `InvalidGridItem` | Likely typo: `row.join(row.split())` where `"".join(...)` was meant | Open (candidate fix) |
| Note | The latest upstream commit claims a solver speed-up (list → set in the unknowns count), but tries were identical on three puzzles, and the count is now always 1 | Claimed improvement not measurable | Discuss in report |

INC-03 to INC-06 were found during project selection. Each must be **re-reproduced on our own machines** before it goes in the report.

## Roles

Each person owns one testing approach end to end: tests, report section, share of Part 1, and 5 minutes of video.

| | Role A: Functional (black-box) | Role B: Structural (white-box) | Role C: Property-based + Regression |
|---|---|---|---|
| Owner | _TBD_ | _TBD_ | _TBD_ |
| Techniques | Equivalence partitioning, boundary values, decision table for CLI options | Branch coverage (pytest-cov), monkeypatching, fixed random seeds | Hypothesis properties; regression tests (`xfail(strict=True)` for open bugs) |
| Main targets | File input; solver behaviour on valid/invalid/unsolvable puzzles; CLI combinations | `sudoku_game.py` main and options; `TooManyTries` and no-solution branches; generator masking | Solutions obey the rules; givens preserved; generated puzzles solvable; per-level hidden counts; unique-solution investigation |
| Also owns | Requirements list + traceability table | Test environment; coverage before/after | Incident log |
| Part 1 share | Introduction, unit test frameworks | Coverage tools, static analysis, symbolic execution | Property-based/fuzz, mutation testing, CI, AI trends, conclusion |
| Test folder | `tests/functional/` | `tests/structural/` | `tests/property/`, `tests/regression/` |

Test case template: **ID · requirement · input · expected result · actual result · pass/fail**

## Plan

- **Step 0: Kick-off** (25–26 Nov). Confirm roles, split Part 1, agree on templates, share the fork.
- **Step 1: Setup and baseline** (26–27 Nov, Defne). Fork and clone, venv, run program and tests from both locations, save baseline coverage (terminal output + `htmlcov` copy + date + Python version), log INC-01 and INC-02, create and push test folders.
- **Step 2: Requirements** (27–29 Nov, Role A leads). 12–20 numbered requirements from Sudoku rules, README and CLI options, including "expected" ones the code may not meet (invalid input rejected clearly; unique solution). Traceability table requirement → test IDs.
- **Step 3: Design and run tests** (29 Nov–5 Dec, all in parallel). Each role works in the order listed in Roles. Log incidents as found. **Write Part 1 sections in parallel.**
- **Step 4: Fix, re-test, wrap up** (5–7 Dec). Fix one or two simple incidents (e.g. INC-02, INC-06), re-run everything, final coverage before/after, results written with *why* each failure happened, joint recommendations.
- **Step 5: Report, Part 1, video** (6–9 Dec). Merge Part 1 into one PDF. Assemble the report. Record the video (≤5 min each, English) **by 8 Dec**: no video means a project grade of zero.
- **Step 6: Submit** (10 Dec). Re-run all tests, check video length, upload 3 separate files. 11 Dec is buffer.

### Report outline (Part 3)
1. Introduction: purpose
2. Validation: 2.1 objectives and scope of the software; 2.2 requirements and traceability (A)
3. Verification: 3.1 test strategy and why these approaches; 3.2 test environment (B)
4. Test results: 4.1 functional (A); 4.2 structural (B); 4.3 property-based and regression (C)
5. Test incidents: 5.1 resolved; 5.2 unresolved (C)
6. Recommendations (all)
7. Appendix: Student AI Declaration

## Status

_Update this section as work progresses: date, who, what._

- 2026-10-16: Group and topic submitted (Sudoku).
- 2026-11-25: Plan agreed. Defne starting Step 1.
- 2026-11-25 (Defne): Step 1 done. Fork `Degnet1/sudoku` (remotes: `origin` = fork, `upstream` = datahappy1). venv with pytest, pytest-cov, hypothesis. Program runs via `-m`; INC-01 and INC-02 reproduced; baseline saved in `baseline/`; test folders created. Still to do: add Ezgi and Elif as collaborators; each teammate clones and runs the tests once on their own machine.
