# Chapter 5: Configuration

## Core Idea
pytest-mock can be configured via `pytest.ini` / `pyproject.toml` to use a standalone `mock` package, disable traceback monkeypatching, and customize assertion error reporting.

## Frameworks Introduced
- **mock_use_standalone_module**: Forces pytest-mock to import `mock` from PyPI instead of `unittest.mock`.
  - When to use: When you need features from the standalone `mock` package not in `unittest.mock`.
  - How: Set `mock_use_standalone_module = true` in `[pytest]` section.
- **mock_traceback_monkeypatch**: Controls whether internal traceback entries from `mock` are hidden in assertion errors.
  - When to use: Disable if you encounter issues with traceback formatting.
  - How: Set `mock_traceback_monkeypatch = false` in `[pytest]` section.

## Key Concepts
- **improved assertion reporting**: pytest-mock monkeypatches mock to show better diffs when `assert_called_with()` fails — it uses pytest's advanced assertions to highlight argument differences.
- **traceback suppression**: Internal mock module frames are hidden from error output for cleaner test failures.
- **--tb=native interaction**: The traceback monkeypatch is automatically disabled with `--tb=native` because the underlying mechanism doesn't work with exception chaining on Python 3.5+.

## Mental Models
- Think of configuration as two switches: one for the mock backend (standalone vs. stdlib), one for error presentation (verbose vs. clean tracebacks).

## Anti-patterns
- **Enabling standalone mock unnecessarily**: The stdlib `unittest.mock` is sufficient for most use cases; only switch if you need specific standalone features.
- **Disabling traceback monkeypatching without reason**: The improved reporting is a major benefit — only disable if it causes actual problems.

## Code Examples
```ini
# pytest.ini
[pytest]
mock_use_standalone_module = true
mock_traceback_monkeypatch = false
```
- **What it demonstrates**: Both configuration options in a pytest.ini file.

## Reference Tables

| Option | Default | Effect |
|--------|---------|--------|
| `mock_use_standalone_module` | `false` | Use `mock` from PyPI instead of `unittest.mock` |
| `mock_traceback_monkeypatch` | `true` | Hide internal mock frames in assertion tracebacks |

## Key Takeaways
1. `mock_use_standalone_module = true` switches to the PyPI `mock` package.
2. `mock_traceback_monkeypatch = false` disables improved assertion error formatting.
3. Improved reporting automatically shows argument diffs when `assert_called_with()` fails.
4. Traceback monkeypatch is auto-disabled with `--tb=native`.

## Connects To
- **Ch 1**: Introduction — basic setup doesn't require configuration
- **Ch 2**: Mocker Patch Methods — the patching that configuration affects
