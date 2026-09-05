# Chapter 6: Usage and CLI

## Core Idea
pytest's CLI provides powerful test selection (`-k`, `-m`, `-x`), verbose output (`-v`, `-vv`), failure focus (`--tb=short`), and debugging (`--pdb`, `-s`) with minimal configuration.

## Frameworks Introduced
- **Test Selection**: `-k` expression, `-m` marker, `-x` stop-first, `--lf` last-failed, `--ff` failed-first
- **Output Control**: `-v`/`-vv` verbosity, `--tb=short/long/line/no` traceback style, `-q` quiet
- **Debugging**: `--pdb` drop into debugger on failure, `-s` disable capture, `--breakpoint`
- **Collection**: `--collect-only` preview tests without running, `--ignore`/`--deselect` exclude paths

## Key Concepts
- **-k EXPRESSION**: Filter tests by name — `-k "test_method or test_class"`
- **-m MARKER**: Filter by marker — `-m "not slow"`
- **-x / --exitfirst**: Stop after first failure
- **--lf / --last-failed**: Re-run only last failures
- **--ff / --failed-first**: Run failures first, then rest
- **--tb=STYLE**: Traceback format — `short`, `long`, `line`, `no`, `auto`
- **-s**: Disable stdout/stderr capture (print visible)
- **--pdb**: Interactive debugger on failure
- **-n NUM**: Parallel execution via pytest-xdist

## Mental Models
- Start with `pytest -x -v` for development; switch to `pytest --lf` for rerunning failures
- Use `-k` for name-based filtering: `-k "login and not slow"`
- `-s` is essential for debugging — without it, print output is hidden
- `--collect-only` is your friend for understanding what pytest will run

## Anti-patterns
- **Running all tests during development**: Use `-x` or `--lf` to save time
- **Ignoring captured output**: Use `-s` when debugging; check `--capture=no` for permanent uncapure
- **Not using --tb=short**: Long tracebacks are noisy; short gives essential info

## Code Examples
```bash
# Basic run with verbose output
pytest -v

# Stop on first failure
pytest -x

# Run only tests matching name pattern
pytest -k "login or auth"

# Run only unmarked-slow tests
pytest -m "not slow"

# Re-run only last failures
pytest --lf

# Short traceback, no capture
pytest -x --tb=short -s

# Debug failures with pdb
pytest --pdb

# Preview what would run
pytest --collect-only

# Parallel execution (requires pytest-xdist)
pytest -n auto

# Run specific file with specific test
pytest tests/test_auth.py::TestLogin::test_valid_credentials
```

## Key Takeaways
1. `pytest -x` stops on first failure — use during development
2. `-k "expression"` filters by test name with boolean logic
3. `-m "marker"` filters by marker; combine with boolean: `-m "slow and not network"`
4. `--lf` reruns only last failures — efficient for iteration
5. `-s` disables capture — essential for debugging with print/pdb
6. `--tb=short` gives concise tracebacks; `--tb=long` for full context
7. `--collect-only` shows what tests would run without executing them

## Connects To
- **Ch 5**: Markers — `-m` selection works with marker system
- **Ch 8**: Logging/Capture — `-s` and `--capture` control output capture
- **Ch 12**: Test Output — failure reporting and exit codes
