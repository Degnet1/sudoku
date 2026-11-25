# SEN4013 V&V Project: Full Plan and Working Notes

This is the detailed companion to `CLAUDE.md`. `CLAUDE.md` holds the rules, roles, incident log and status; this file holds **how** we do each step, how to set up, and what we have learned about the code. Claude Code loads this file automatically (via `@PLAN.md` in `CLAUDE.md`).

Readable summary for people: the shared doc "SEN4013 V&V Project: Plan and Progress" (Defne shares the link).

---

## 1. Teammate onboarding (do this first)

Goal: you can run the program and the tests on your own laptop, and you get the same numbers as the baseline.

1. Accept the GitHub invite to `Degnet1/sudoku` (Defne sends it once she has your username).
2. Clone and set up:

   **Windows (PowerShell)**
   ```powershell
   git clone https://github.com/Degnet1/sudoku.git
   cd sudoku
   python -m venv venv
   venv\Scripts\Activate.ps1
   pip install pytest pytest-cov hypothesis
   ```
   If PowerShell refuses to run `Activate.ps1`, run once: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.

   **Mac**
   ```bash
   git clone https://github.com/Degnet1/sudoku.git
   cd sudoku
   python3 -m venv venv
   source venv/bin/activate
   pip install pytest pytest-cov hypothesis
   ```
   You should see `(venv)` at the start of the terminal line.
3. Set your git identity for this repo (use your GitHub no-reply address if you want your email private: GitHub → Settings → Emails):
   ```bash
   git config user.name "Your Name"
   git config user.email "<id>+<username>@users.noreply.github.com"
   ```
4. Run the program from the **project root** (the README command does not work, see INC-01):
   ```bash
   python -m sudoku.sudoku_game -a generate -l hard -p 1
   python -m sudoku.sudoku_game -a solve -f files/sudoku_hard.txt -p 1
   ```
   Expect a puzzle, then a solved grid ending `Finished in … in 620 tries`.
5. Run the existing tests **from inside `tests/`** (INC-02):
   ```powershell
   cd tests
   $env:PYTHONPATH=".."        # Mac: export PYTHONPATH=..
   python -m pytest
   ```
   Expect **16 passed**. If you get something else, tell the group: a difference between machines is itself a finding.
6. Put your own tests in your role's folder (`tests/functional/`, `tests/structural/`, `tests/property/`, `tests/regression/`). Commit often, `git pull` before you start each session, and push when done.

**Do not** commit `venv/`, coverage output or the course PDF (`.gitignore` handles this). **Do not** change `baseline/`: it is our "before" evidence.

---

## 2. The steps in detail

### Step 0: Kick-off (25–26 Nov, all) — in progress
Goal: everyone knows their role and the rules before writing any test.
- Confirm roles, one testing approach each (see Roles in `CLAUDE.md`). Owners are still TBD.
- Split Part 1 (35 pts) the same way, so each person writes about the tools they use:
  - A: introduction, unit test frameworks (pytest, JUnit)
  - B: coverage tools, static analysis (mypy, Pylint), symbolic execution
  - C: property-based/fuzz testing, mutation testing, CI, AI trends, conclusion
- Agree on the two templates (test case, incident) so the report reads as one team's work. Both are in `CLAUDE.md`.
- One shared fork; everyone works in it. Done: `Degnet1/sudoku`.

### Step 1: Setup and baseline (25 Nov, Defne) — done
Goal: the program and tests run, and the starting point is recorded.
- Forked, cloned, venv with pytest 9.2.0, pytest-cov 7.1.0, hypothesis 6.170.17 (Python 3.14.2).
- Program runs with `-m`; README command fails (INC-01).
- Tests: 9 failed / 7 passed from the root (INC-02); 16 passed from `tests/`.
- Baseline coverage: 82% overall, 47% `sudoku_game.py`. Everything is in `baseline/` (see `baseline/README.md`).
- Test folders created and pushed (commit `127ec3b`).
- **Still open:** add Ezgi and Elif as collaborators; each runs onboarding (section 1) once.

### Step 2: Requirements (27–29 Nov, Role A leads, all review)
Goal: a numbered list of what the software should do. Every test points back to one of these. This list is the "Validation" half of the report.
- Write requirements from three sources:
  - **Sudoku rules**, e.g. R1: a solution has 1–9 exactly once in every row, column and 3×3 box.
  - **README**, e.g. R5: the generator offers easy, medium and hard levels.
  - **Command-line options**, e.g. R9: `-a` accepts `solve` or `generate`.
- Add **"expected" requirements the code may not meet**, e.g. "an invalid puzzle is rejected with a clear message", "a generated puzzle has exactly one solution". These produce the validation findings.
- Make a **traceability table**: requirement → test case IDs. Fill it in as tests are written.
- Done when: about 12–20 numbered requirements everyone agrees on.

### Step 3: Design and run tests (29 Nov–5 Dec, all in parallel)
Goal: the core of Part 2. Each person writes, runs and records their tests, and logs every problem in the incident log the moment it is found. **Write Part 1 sections in parallel.**

**Role A: functional (black-box), in this order**
1. File input (equivalence partitions and boundaries): valid puzzle · 8 / 10 rows · 8 / 10 digits in a row · a letter · digits separated by spaces · an empty file.
2. Solver behaviour: easy / hard / "world's hardest" puzzles · an already-solved grid · an unsolvable puzzle · invalid starting numbers (two 5s in a row, a grid of only 1s).
3. Command-line decision table: every combination of `-a` (solve / generate / invalid), `-l` (easy / medium / hard / invalid), `-f` (exists / missing) and `-p` (true / false / odd values). Call this **combinatorial testing** in the report.

**Role B: structural (white-box), in this order**
1. Open the baseline HTML report (`baseline/coverage_html/index.html`) and list every uncovered branch.
2. `sudoku_game.py` (47%): test `main()` and the option handling.
3. The "too many tries" branch: monkeypatch `solver.MAX_ATTEMPTS` to e.g. 10 inside the test so it is reached instantly.
4. The "no solution" branch: the solver returns `None`; check what the program prints then.
5. Generator branches: the per-level masking in `_add_row_mask`, with a fixed seed.
6. Re-run coverage after each batch and record the improvement.

**Role C: property-based and regression, in this order**
1. Regression tests first, one per known bug (INC-01 … INC-06), so there are results early. Mark unfixed bugs with `@pytest.mark.xfail(strict=True)`: the test documents the bug now and fails loudly once it is fixed, so we remember to remove the marker.
2. Hypothesis properties:
   - every solution follows the Sudoku rules
   - the solver never changes the original numbers
   - every generated puzzle can be solved
   - each level hides the right number of cells per row: easy 3, medium 4–5, hard 5–8
3. Validation investigation: do generated puzzles have exactly one solution? Do "hard" puzzles really need more attempts than "easy"?

Done when: each role has a set of tests run and recorded, and the incident log is up to date.

### Step 4: Fix, re-test, wrap up (5–7 Dec, all)
- Decide which bugs to fix. Fix one or two simple ones (candidates: INC-02 test paths, INC-06 spaces). Re-run **all** tests to show the fixes work and nothing else broke: regression testing in action. The matching `xfail` tests should now pass (remove the marker).
- Leave the rest as unresolved incidents, with an explanation.
- Final coverage run: before vs after, per module (compare with `baseline/`).
- Each person writes their results section, including **why** each failure happened, not just that it failed.
- Recommendations together, e.g. validate starting numbers, return the solved grid instead of printing it, guarantee unique solutions.

### Step 5: Report, Part 1, video (6–9 Dec)
- Finish Part 1 as one PDF and edit it so it reads in one voice.
- Assemble the report (Part 3) using the outline in `CLAUDE.md`. The appendix **Student AI Declaration is required**: list every AI tool and purpose (e.g. Claude for project selection, planning, setup help, explaining code).
- Video: 15 minutes in total, at most 5 per person, in English. A: project and functional results. B: coverage with a short demo. C: properties, bugs, recommendations. **Record by 8 Dec: without a presentation the project grade is zero.**

### Step 6: Submit (10 Dec)
Re-run all tests one last time, check the video length, then upload **three separate files** on Teams: Part 1 (PDF), report (PDF), video. 11 Dec is buffer; late work loses 10% per day.

---

## 3. How the code works (read before testing)

Found by reading the code on 25 Nov. Line numbers refer to upstream commit `d1b37f1`.

**Solver (`sudoku/solver.py`)**
- Works through the grid cell by cell. For each empty cell (0) it computes the allowed numbers: not already in that row, column or 3×3 box (`_get_candidates`).
  - one allowed number → writes it in and continues;
  - several → makes one copy of the grid per number and puts the copies in a priority queue ("guesses");
  - none → dead end, that copy is discarded.
- Each grid taken off the queue counts as one **try**. `MAX_ATTEMPTS = 1000000` (line 17); past it, it raises `CustomException("TooManyTries")`.
- It **prints** the solved grid and **returns only the try count**, or `None` if the queue empties with no solution.
- It **only ever looks at empty cells**. Given numbers are never checked against the rules. This is the root cause of INC-03 (grid of 1s has no empty cells → "solved" in 1 try) and INC-04 (impossible givens → guesses until 1,000,000 tries).
- **The upstream "speed-up" bug (line 86):** `unknowns_count = len({e for r in grid_rows for e in r if e == 0})` builds a *set* of zeros, so its length is always 1 (or 0). The queue priority is therefore constant and the intended "fewest unknowns first" ordering no longer happens. Explains why the claimed improvement could not be measured.

**Generator (`sudoku/generator.py`)**
- Does **not** use the solver. Builds a full grid row by row with random allowed numbers; if it gets stuck it throws the grid away and starts again. The "tries" printed after generating count these restarts.
- Then hides cells per row in `_add_row_mask`: easy exactly 3; medium 4–5; hard 5–7 plus one extra random number that is hidden only if still visible, so 5–8.
- Never checks that the puzzle has a **unique** solution.
- Uses `random.sample` / `random.randint`: call `random.seed(...)` for repeatable tests.

**Command line and file input (`sudoku/sudoku_game.py`)**
- `-a` is required, choices `solve` / `generate`; `-l` defaults to `easy`; `-f` defaults to `files/sudoku_easy.txt` (a relative path, so it depends on the current folder); `-p` defaults to `false`.
- `-p`: only `no`, `false`, `f`, `n`, `0` (any case) mean False. **Any other value, e.g. `banana`, means True** (line 108). Relevant for Role A's "odd values".
- `_validate_row` (line 37): `row.join(row.split())` joins the pieces using the row itself as separator. With no spaces there is one piece, so it happens to work; with spaces the row text gets inserted between digits and `int()` fails → `InvalidGridItem`. This is INC-06; the intended code is almost certainly `"".join(row.split())`.
- `_validate_grid_shape` (line 46) rejects anything not 9×9 with `InvalidGridShape`.
- `main()` always prints `Finished in … in {runs_count} tries`; when the solver returns `None` this becomes "in None tries" with no error (INC-05).

---

## 4. Testing tips

- **Capture printed output** with pytest's `capsys` fixture (the solver and generator print their grids).
- **Fixed seed** for anything using the generator: `random.seed(42)`.
- **Monkeypatch** limits instead of waiting: `monkeypatch.setattr(solver, "MAX_ATTEMPTS", 10)`.
- **Open bugs** → `@pytest.mark.xfail(strict=True, reason="INC-0x")`.
- **CLI tests**: either call `main()` with `monkeypatch.setattr(sys, "argv", [...])`, or run `python -m sudoku.sudoku_game ...` via `subprocess` from the project root.
- **Test data files**: build paths from the test file's own location (`pathlib.Path(__file__).parent`), not from the current folder, so our tests do not repeat INC-02.
- **Coverage after each batch** (from `tests/`): `python -m pytest --cov=sudoku --cov-branch --cov-report=term-missing`. Note: our new tests in subfolders are collected too.
- Use the templates in `CLAUDE.md` for every test case and incident.

---

## 5. Why these approaches (points for report section 3.1)

The brief says functional and structural testing are only examples; we may choose any approaches covered in the course and must justify the choice. We use three. Write the justification in our own words; these are the points:

- **Functional:** the program is driven entirely by files and command options, so input/output testing covers what users do; most known bugs were found this way. The CLI decision table is also **combinatorial testing**.
- **Structural:** we measured a real gap (`sudoku_game.py` at 47%), giving a concrete before/after.
- **Property-based:** Sudoku has a perfect oracle (rules can be checked without knowing the answer), so many random puzzles can be checked automatically; it suits the random generator.
- **Regression:** six known bugs and planned fixes in Step 4 need tests that prove the fixes work and nothing else broke.
