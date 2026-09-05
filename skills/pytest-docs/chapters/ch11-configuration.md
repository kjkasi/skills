# Chapter 11: Configuration

## Core Idea
pytest is configured via `pyproject.toml`, `pytest.ini`, `setup.cfg`, or `conftest.py` — settings control test paths, markers, options, and plugin behavior with precedence rules.

## Frameworks Introduced
- **Configuration Files**: `pyproject.toml` (recommended), `pytest.ini`, `setup.cfg`, `conftest.py`
- **Marker Registration**: Declare custom markers to avoid warnings and document purpose
- **ini Options**: `testpaths`, `addopts`, `filterwarnings`, `log_level`, `markers`
- **conftest.py Hierarchy**: Root conftest + directory conftests — deeper conftests override shallower

## Key Concepts
- **pyproject.toml [tool.pytest.ini_options]**: Modern config format — `[tool.pytest.ini_options]` section
- **pytest.ini**: Dedicated pytest config — `[pytest]` section
- **testpaths**: List of directories to search for tests — `testpaths = ["tests"]`
- **addopts**: Default CLI options — `addopts = "-v --tb=short --strict-markers"`
- **markers**: List of custom markers with descriptions
- **filterwarnings**: Suppress or error on specific warnings
- **confcutdir**: Root directory for conftest.py discovery
- **norecursedirs**: Directories to exclude from test collection

## Mental Models
- `pyproject.toml` is the modern standard — prefer it over `pytest.ini`
- `addopts` injects default CLI flags — use it to set project-wide defaults
- conftest.py hierarchy means deeper directories can override parent fixtures/hooks
- `--strict-markers` enforces marker registration — use it to catch typos

## Anti-patterns
- **Not registering custom markers**: Causes warnings; use `--strict-markers` to enforce
- **Hardcoding paths in CLI**: Use `testpaths` in config to set default search paths
- **Inconsistent config across tools**: Use `pyproject.toml` for pytest, ruff, mypy, etc.

## Code Examples
```toml
# pyproject.toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --tb=short --strict-markers -x"
markers = [
    "slow: marks tests as slow (deselect with '-m \"not slow\"')",
    "network: marks tests requiring network access",
]
filterwarnings = [
    "ignore::DeprecationWarning",
    "error::pytest.PytestUnraisableExceptionWarning",
]
log_level = "INFO"
log_format = "%(asctime)s %(levelname)s %(message)s"
```

```ini
# pytest.ini
[pytest]
testpaths = tests
addopts = -v --tb=short
markers =
    slow: marks tests as slow
    network: marks tests requiring network
```

```python
# conftest.py — root level
collect_ignore = ["setup.py"]  # Exclude from collection

# conftest.py — subdirectory (overrides parent for that directory)
@pytest.fixture
def db():
    return connect_db()
```

```bash
# Override config from CLI
pytest -o "addopts=-v --tb=long"   # Override addopts
pytest --override-ini="log_level=DEBUG"
pytest -c /path/to/pytest.ini      # Use different config file
```

## Key Takeaways
1. `pyproject.toml` is the recommended config format — `[tool.pytest.ini_options]`
2. `addopts` sets default CLI flags — use for project-wide consistency
3. `markers` list prevents warnings — register all custom markers
4. `testpaths` sets default test search directories
5. conftest.py hierarchy: deeper conftests override parent hooks/fixtures
6. `-o key=value` overrides config from CLI; `-c path` uses alternate config

## Connects To
- **Ch 10**: Plugins — conftest.py is a project-local plugin
- **Ch 5**: Markers — marker registration in config files
- **Ch 6**: Usage — `addopts` injects default CLI options
