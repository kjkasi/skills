# Chapter 9: Skipping and Xfail

## Core Idea
pytest provides `skip`/`skipIf` for unconditional/conditional test skipping, and `xfail` for expected failures — both support reason documentation and integrate with marker selection.

## Frameworks Introduced
- **Skip Mechanism**: `pytest.skip()` function and `@pytest.mark.skip`/`@pytest.mark.skipif` decorators
- **Xfail Mechanism**: `@pytest.mark.xfail` for expected failures with optional `raises` parameter
- **Skip in fixtures**: `pytest.skip()` called within fixtures to skip tests requesting that fixture
- **Parametrize integration**: `pytest.param(..., marks=pytest.mark.skip)` skips specific parametrize values

## Key Concepts
- **pytest.skip(reason)**: Programmatic skip — can be called anywhere in test/fixture code
- **@pytest.mark.skip(reason=...)**: Unconditional skip decorator
- **@pytest.mark.skipif(condition, reason=...)**: Conditional skip — skip when condition is True
- **@pytest.mark.xfail(reason=...)**: Expected failure — test still runs, failure not reported as error
- **@pytest.mark.xfail(raises=Exc)**: xfail only if specific exception type raised
- **@pytest.mark.xfail(strict=True)**: Fails if test unexpectedly passes (useful for tracking bug fixes)
- **--runxfail**: CLI flag to run xfail tests as normal (ignore xfail marker)

## Mental Models
- `skip` = "don't run this test at all" — for environments/conditions where test is irrelevant
- `xfail` = "run it, but don't worry if it fails" — for known bugs or incomplete features
- Use `strict=True` on xfail to detect when a bug is fixed (test becomes passing = unexpected)
- Skip in fixtures to propagate skip to all tests using that fixture

## Anti-patterns
- **Skipping without reason**: Always provide a reason — helps future developers understand why
- **Using xfail for tests that should pass**: xfail documents known failures; fix the code instead
- **Forgetting strict=True on xfail**: Without it, you won't know when a bug is fixed

## Code Examples
```python
# Unconditional skip
@pytest.mark.skip(reason="not yet implemented")
def test_future():
    pass

# Conditional skip
@pytest.mark.skipif(sys.platform == "win32", reason="posix only")
def test_posix():
    pass

# Programmatic skip
def test_condition(config):
    if not config.get("feature_enabled"):
        pytest.skip("feature not enabled")

# Expected failure
@pytest.mark.xfail(reason="known bug #789")
def test_known_bug():
    assert buggy_function() == expected

# xfail with strict mode
@pytest.mark.xfail(strict=True, reason="bug #789")
def test_fixed_bug():
    # If this starts passing, test will FAIL — bug is fixed
    assert fixed_function() == expected

# xfail only on specific exception
@pytest.mark.xfail(raises=ConnectionError, reason="network issues")
def test_network():
    return unreliable_network_call()

# Skip specific parametrize values
@pytest.mark.parametrize("input", [
    1,
    2,
    pytest.param(3, marks=pytest.mark.skip(reason="bug #321")),
])
def test_process(input):
    process(input)

# Skip in fixture
@pytest.fixture
def db():
    if not HAS_DATABASE:
        pytest.skip("database not available")
    return connect_db()
```

## Key Takeaways
1. `pytest.skip(reason)` programmatically skips — works in tests and fixtures
2. `@pytest.mark.skipif(condition, reason=...)` conditionally skips
3. `@pytest.mark.xfail(reason=...)` marks expected failures — test still runs
4. `strict=True` on xfail makes unexpected passes fail the test
5. `pytest.param(value, marks=pytest.mark.skip)` skips specific parametrize values
6. `--runxfail` CLI flag runs xfail tests as normal for debugging

## Connects To
- **Ch 5**: Markers — skip/xfail are built-in markers
- **Ch 4**: Parametrize — skip individual parametrize values with `pytest.param`
- **Ch 6**: Usage — `--runxfail` flag for debugging expected failures
