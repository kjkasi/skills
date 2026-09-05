# Chapter 12: Test Output and Exit Codes

## Core Idea
pytest provides detailed failure reports with assertion introspection, configurable traceback styles, and meaningful exit codes — 0 for success, 1 for failures, 2 for interrupted, 3+ for internal errors.

## Frameworks Introduced
- **Traceback Styles**: `auto` (default), `long`, `short`, `line`, `no` — control failure report verbosity
- **Exit Codes**: 0=passed, 1=failed, 2=interrupted, 3=internal error, 4=usage error, 5=no tests collected
- **Report Sections**: `--report-log` for JSON report, `--junitxml` for CI integration
- **Failure Summary**: Short test summary info at bottom of output

## Key Concepts
- **--tb=STYLE**: `auto` (long for first failure, short for rest), `long`, `short`, `line`, `no`
- **Exit code 0**: All tests passed
- **Exit code 1**: One or more tests failed
- **Exit code 2**: Test execution was interrupted (e.g., Ctrl+C)
- **Exit code 3**: Internal pytest error (bug in pytest or plugin)
- **Exit code 4**: Usage error (bad CLI arguments)
- **Exit code 5**: No tests were collected
- **--junitxml=path**: Generate JUnit XML for CI systems
- **--report-log=path**: Generate JSON log of test results

## Mental Models
- Exit codes are your CI integration signal — 0 means green, anything else means investigate
- `--tb=short` is the sweet spot for most CI — enough info without noise
- Use `--junitxml` for Jenkins/GitHub Actions; use `--report-log` for custom analysis
- Short test summary at bottom of output is the most actionable part

## Anti-patterns
- **Ignoring exit codes in CI**: Always check exit code — non-zero means problems
- **Using --tb=long in CI**: Too verbose; use `--tb=short` or `--tb=line`
- **Not generating test reports**: Use `--junitxml` for CI visibility

## Code Examples
```bash
# Traceback styles
pytest --tb=long      # Full traceback (default for first failure)
pytest --tb=short     # Concise traceback with key info
pytest --tb=line      # One-line per failure
pytest --tb=no        # No traceback, just pass/fail summary
pytest --tb=auto      # Long for first, short for rest (default)

# Exit codes
pytest; echo $?       # 0 = all passed
pytest --co; echo $?  # 5 = no tests collected (if none found)

# CI integration
pytest --junitxml=report.xml
pytest --report-log=report.json

# Custom failure explanation
def pytest_assertrepr_compare(op, left, right):
    if isinstance(left, Model) and op == "==":
        return [f"Model comparison:", f"  left.id={left.id}", f"  right.id={right.id}"]
```

## Key Takeaways
1. Exit codes: 0=pass, 1=fail, 2=interrupted, 3=internal error, 4=usage error, 5=no tests
2. `--tb=short` is ideal for CI — concise failure information
3. `--junitxml` generates CI-compatible XML reports
4. `--report-log` generates JSON for custom analysis
5. Custom assertion output via `pytest_assertrepr_compare` hook
6. Short test summary at output bottom is the most actionable failure info

## Connects To
- **Ch 6**: Usage — CLI options for output control
- **Ch 2**: Assertions — assertion introspection and custom comparisons
- **Ch 10**: Plugins — hooks for custom reporting
