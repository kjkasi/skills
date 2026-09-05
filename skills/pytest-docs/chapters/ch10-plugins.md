# Chapter 10: Plugins

## Core Idea
pytest's plugin system extends functionality through hooks — plugins are discovered automatically from `sys.path`, installed packages, and `conftest.py` files, with 1300+ community plugins available.

## Frameworks Introduced
- **Plugin Architecture**: Hooks-based extension system — plugins implement `pytest_*` and `pytest_runtest_*` hooks
- **Plugin Discovery**: Auto-discovered from installed packages, `PYTEST_PLUGINS` env var, and `conftest.py`
- **Plugin Loading**: `-p name` to load specific plugin, `-p no:name` to disable, `PYTEST_ADDOPTS` env var
- **conftest.py**: Project-local plugin file — hooks and fixtures automatically discovered

## Key Concepts
- **Hook**: Function named `pytest_*` that pytest calls at specific points — `pytest_collection_modifyitems`, `pytest_runtest_makereport`
- **Plugin**: Module implementing one or more hooks
- **conftest.py**: Special file for project-local fixtures and hooks — auto-discovered
- **Entry point plugins**: Installed packages with `pytest11` entry point — auto-loaded
- **Plugin order**: Internal → third-party → conftest.py (conftest wins on conflicts)
- **-p name / -p no:name**: Force-load or disable specific plugins
- **--co / --collect-only**: Useful hook `pytest_collection_modifyitems` to modify test collection

## Mental Models
- conftest.py is your "project plugin" — fixtures and hooks go here
- Plugin hooks are "event handlers" — pytest fires events, plugins respond
- Plugin order matters: later plugins can override earlier ones
- Use `-p no:devtools` to disable a plugin that's causing issues

## Anti-patterns
- **Putting fixtures in test files**: Use conftest.py for shared fixtures — they're auto-discovered
- **Ignoring plugin conflicts**: Check `--co` output to see what's loaded
- **Not disabling broken plugins**: Use `-p no:pluginname` to isolate issues

## Code Examples
```python
# conftest.py — project-local plugin
def pytest_collection_modifyitems(items):
    """Modify collected items — e.g., add markers based on path."""
    for item in items:
        if "slow" in str(item.fspath):
            item.add_marker(pytest.mark.slow)

# Custom hook for reporting
def pytest_runtest_makereport(item, call):
    """Called after each test phase — customize reporting."""
    if call.when == "call" and call.excinfo is not None:
        # Test failed during call phase
        item.add_report_section("call", "extra", "Custom info")

# Fixture in conftest.py (auto-discovered)
@pytest.fixture
def api_client():
    return APIClient(base_url="http://localhost:8000")
```

```bash
# Plugin management
pytest -p no:devtools          # Disable a plugin
pytest -p myplugin             # Force-load a plugin
pytest --co                    # See collected tests (useful for debugging plugins)
pip list | grep pytest         # List installed pytest plugins

# Popular plugins
# pytest-xdist: parallel execution (-n auto)
# pytest-cov: coverage reporting (--cov=src)
# pytest-mock: unittest.mock wrapper
# pytest-django: Django integration
# pytest-asyncio: async test support
```

## Key Takeaways
1. conftest.py is pytest's project-local plugin — fixtures + hooks auto-discovered
2. Plugins implement `pytest_*` hooks called at specific points in the test lifecycle
3. `-p no:name` disables a plugin; `-p name` forces loading
4. Plugin order: internal → third-party → conftest (later overrides earlier)
5. 1300+ community plugins — check `pytest --co` to see what's loaded
6. Entry point plugins (`pytest11`) are auto-discovered from installed packages

## Connects To
- **Ch 3**: Fixtures — conftest.py is the primary fixture repository
- **Ch 11**: Configuration — pytest.ini/pyproject.toml for plugin configuration
- **Ch 2**: Assertions — `pytest_assertrepr_compare` is a hook for custom comparison output
