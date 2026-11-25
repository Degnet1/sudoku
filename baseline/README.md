# Baseline (before any of our tests)

Recorded by Defne Doğan on **2026-11-25**, Step 1 of the plan in `CLAUDE.md`.

| | |
|---|---|
| Code under test | upstream commit `d1b37f1` (2025-11-02), unchanged |
| Python | 3.14.2 (Windows 11, virtual environment) |
| Tools | pytest 9.2.0, pytest-cov 7.1.0 (coverage 7.17.5), hypothesis 6.170.17 |

## Files

| File | What it shows |
|---|---|
| `01_readme_command_INC-01.txt` | README run command fails with `ModuleNotFoundError` (INC-01) |
| `02_generate_hard.txt` | `python -m sudoku.sudoku_game -a generate -l hard -p 1` works |
| `03_solve_hard.txt` | Solving `files/sudoku_hard.txt` works: 620 tries |
| `04_tests_from_root_INC-02.txt` | Existing tests from project root: **9 failed, 7 passed** (INC-02) |
| `05_tests_from_tests_folder.txt` | Existing tests from `tests/` with `PYTHONPATH=..`: **16 passed** |
| `06_baseline_coverage.txt` | Branch coverage, terminal report |
| `coverage_html/index.html` | Same coverage as HTML, uncovered lines in red |

## Baseline branch coverage

| Module | Stmts | Miss | Branches | Partial | Cover | Missing |
|---|---|---|---|---|---|---|
| `action_type.py` | 4 | 0 | 0 | 0 | 100% | |
| `exceptions.py` | 1 | 0 | 0 | 0 | 100% | |
| `generator.py` | 67 | 1 | 28 | 2 | 97% | 51->64, 121 |
| `grid.py` | 27 | 2 | 0 | 0 | 93% | 80-83 |
| `level.py` | 5 | 0 | 0 | 0 | 100% | |
| `printer.py` | 4 | 1 | 2 | 1 | 67% | 16 |
| `solver.py` | 50 | 2 | 20 | 2 | 94% | 123, 136 |
| `sudoku_game.py` | 63 | 32 | 12 | 2 | **47%** | 38-39, 47, 71-113, 120-135, 139 |
| **Total** | 221 | 38 | 62 | 7 | **82%** | |

How to reproduce: see "How to run things" in `CLAUDE.md`.
