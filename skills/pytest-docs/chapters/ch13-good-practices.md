# Chapter 13: Good Practices

## Core Idea
Organize tests with `src` layout, use virtual environments, keep test files independent, leverage conftest.py for shared fixtures, and follow the testing pyramid for maintainable test suites.

## Frameworks Introduced
- **src Layout**: Source code in `src/mypackage/`, tests in `tests/` — prevents accidental imports of uninstalled code
- **Test Independence**: Each test should be runnable in isolation — no shared mutable state
- **conftest.py as Fixture Hub**: Shared fixtures live in conftest.py, not test files
- **Testing Pyramid**: Many unit tests, fewer integration tests, minimal end-to-end tests

## Key Concepts
- **src layout**: `src/package_name/` for source, `tests/` for tests — cleaner separation
- **Virtual environments**: Always use venvs — `python -m venv .venv && source .venv/bin/activate`
- **Test naming**: `test_<what>_<scenario>_<expected>` — descriptive, consistent
- **Fixture placement**: Shared fixtures in conftest.py; test-specific fixtures in test files
- **Test isolation**: Each test sets up its own state — no dependency on test execution order
- **Parametrize over loops**: Use `@pytest.mark.parametrize` instead of `for` loops in tests

## Mental Models
- src layout prevents "my tests pass locally but fail in CI" — tests import installed package
- conftest.py is your "test infrastructure" — fixtures, hooks, configuration
- Each test is a mini documentation of behavior — name it clearly
- Test order independence = reliable test suite — always clean up or isolate

## Anti-patterns
- **Tests depending on execution order**: Each test must work independently
- **for loops in tests**: Use parametrize — it gives individual test IDs and reporting
- **Shared mutable fixtures**: Fixtures return new instances per test by default — use this
- **Putting everything in one test file**: Split by feature/concern for maintainability

## Code Examples
```
# Recommended project layout
myproject/
├── src/
│   └── mypackage/
│       ├── __init__.py
│       └── core.py
├── tests/
│   ├── conftest.py          # Shared fixtures
│   ├── test_core.py
│   └── integration/
│       ├── conftest.py      # Integration-specific fixtures
│       └── test_api.py
├── pyproject.toml           # pytest config here
└── README.md
```

```python
# conftest.py — shared fixtures
@pytest.fixture
def app():
    return create_app(config={"TESTING": True})

@pytest.fixture
def client(app):
    return app.test_client()

# tests/test_core.py
def test_addition():
    assert add(1, 2) == 3

@pytest.mark.parametrize("input,expected", [
    (1, 2),
    (0, 1),
    (-1, 0),
])
def test_increment(input, expected):
    assert increment(input) == expected
```

## Key Takeaways
1. Use `src` layout — prevents accidental imports of uninstalled code
2. Always use virtual environments for isolated dependencies
3. Shared fixtures go in conftest.py, not test files
4. Use `@pytest.mark.parametrize` instead of for-loops in tests
5. Each test should be independently runnable — no shared state dependency
6. Name tests descriptively: `test_<what>_<scenario>_<expected>`

## Connects To
- **Ch 3**: Fixtures — conftest.py as fixture repository
- **Ch 10**: Plugins — conftest.py is a project-local plugin
- **Ch 11**: Configuration — pyproject.toml for project config
