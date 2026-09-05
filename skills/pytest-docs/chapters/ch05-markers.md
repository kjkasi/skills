# Chapter 5: Markers

## Core Idea
Markers tag tests for selection, filtering, and behavior modification — built-in marks handle skip/xfail/slow, custom marks enable project-specific test organization.

## Frameworks Introduced
- **Marker System**: `@pytest.mark.name(args)` decorators applied to tests, classes, or modules for selection and behavior
- **Built-in Marks**: `skip`, `skipIf`, `xfail`, `usefixtures`, `filterwarnings`, `timeout`
- **Marker Selection**: `-m "marker_name"` CLI flag to run only tests with specific markers
- **Marker Registration**: `pytest.ini` or `pyproject.toml` `[tool.pytest.ini_options]` `markers` list to avoid warnings

## Key Concepts
- **@pytest.mark.skip**: Unconditionally skip a test — `@pytest.mark.skip(reason="not yet")`
- **@pytest.mark.skipif**: Conditional skip — `@pytest.mark.skipif(sys.platform == "win32", reason="unix only")`
- **@pytest.mark.xfail**: Expected failure — `@pytest.mark.xfail(reason="bug #123")`
- **@pytest.mark.xfail(raises=Exc)**: xfail only if specific exception raised
- **-m expression**: Filter by marker — `-m "slow and not network"`
- **Marker inheritance**: Class-level markers apply to all methods in the class

## Mental Models
- Markers are "labels with behavior" — `skip` halts execution, `xfail` tolerates failures, custom marks are just labels
- Use `-m` expressions for complex selection: `-m "slow and not flaky"`
- Register custom markers to avoid pytest warnings and get proper documentation

## Anti-patterns
- **Unregistered custom markers**: Causes warnings; always register in `pytest.ini` or `pyproject.toml`
- **Using xfail for bugs without reason**: Always document why something is expected to fail
- **Overusing skip**: Prefer xfail when the test describes correct behavior that's temporarily broken
- **Markers on fixtures**: Markers only apply to tests; use fixture params or conditionals instead

## Code Examples
```python
# Skip unconditionally
@pytest.mark.skip(reason="not yet implemented")
def test_future_feature():
    pass

# Skip conditionally
@pytest.mark.skipif(sys.platform == "win32", reason="posix only")
def test_posix_only():
    pass

# Expected failure
@pytest.mark.xfail(reason="known bug #456")
def test_known_bug():
    assert broken_function() == expected

# xfail only on specific exception
@pytest.mark.xfail(raises=ConnectionError)
def test_network_call():
    return make_network_request()

# Class-level markers
@pytest.mark.slow
class TestHeavySuite:
    def test_one(self):
        pass
    def test_two(self):
        pass

# Register custom markers (pyproject.toml)
# [tool.pytest.ini_options]
# markers = [
#     "slow: marks tests as slow (deselect with '-m \"not slow\"')",
#     "network: marks tests requiring network access",
# ]

# Select by marker
# pytest -m "slow"
# pytest -m "slow and not network"
```

## Key Takeaways
1. `@pytest.mark.skip(reason=...)` unconditionally skips a test
2. `@pytest.mark.skipif(condition, reason=...)` conditionally skips
3. `@pytest.mark.xfail(reason=...)` expects failure — test still runs
4. `@pytest.mark.xfail(raises=Exc)` xfail only on specific exception type
5. Use `-m "expression"` to select/deselect by marker
6. Register custom markers to avoid warnings and document their purpose
7. Class-level markers apply to all test methods in that class

## Connects To
- **Ch 4**: Parametrize — `pytest.param` applies marks to individual parametrize values
- **Ch 9**: Skipping — deeper dive into skip/xfail patterns
- **Ch 11**: Configuration — marker registration in pyproject.toml
