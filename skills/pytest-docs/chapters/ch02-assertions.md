# Chapter 2: Assertions

## Core Idea
pytest rewrites assert statements at import time to provide rich failure reports showing intermediate values, set diffs, string diffs, and custom explanations.

## Frameworks Introduced
- **Assertion Rewriting**: pytest's import hook transforms assert AST before execution, injecting introspection data into failure messages
- **pytest_assertrepr_compare hook**: Plugin hook to customize how specific types are compared in failure output
- **Exception Groups (RaisesGroup/RaisesExc)**: Structured testing of `ExceptionGroup` and `BaseExceptionGroup` with nesting awareness

## Key Concepts
- **assert introspection**: pytest rewrites `assert a == b` to show both values and subexpression results
- **pytest.approx**: Tolerance-based float/array comparison — `assert a == pytest.approx(b, rel=1e-6)`
- **pytest.raises**: Context manager capturing exceptions — `with pytest.raises(TypeError) as excinfo:`
- **pytest.warns**: Context manager for expected warnings — `with pytest.warns(DeprecationWarning):`
- **RaisesGroup**: For `ExceptionGroup` assertions with `match`, `check`, `flatten_subgroups`, `allow_unwrapped`
- **RaisesExc**: For asserting individual exceptions within groups with precise matching

## Mental Models
- Think of assertion rewriting as "compile-time" transformation — it happens at import, not at runtime
- Use `pytest.raises(match=r"pattern")` for regex matching on exception messages
- For exception groups, prefer `RaisesGroup` over `group_contains()` — the latter can miss unexpected exceptions

## Anti-patterns
- **group_contains for negative checks**: Cannot ensure no *other* exceptions exist; use `RaisesGroup` instead
- **assert abs(a-b) < tol**: Use `pytest.approx` instead — handles scalars, arrays, NaNs, and dicts
- **Legacy pytest.raises(func, args) form**: Use context manager form for readability

## Code Examples
```python
# Basic assertion with introspection
def test_set_comparison():
    set1 = set("1308")
    set2 = set("8035")
    assert set1 == set2  # Shows extra/missing items

# Custom comparison explanation
def pytest_assertrepr_compare(op, left, right):
    if isinstance(left, Foo) and isinstance(right, Foo) and op == "==":
        return ["Comparing Foo instances:", f"   vals: {left.val} != {right.val}"]

# Exception group assertions
def test_exception_in_group():
    with pytest.RaisesGroup(ValueError, TypeError):
        raise ExceptionGroup("msg", [ValueError("foo"), TypeError("bar")])

# Nested exception group with RaisesExc
def test_nested():
    with pytest.RaisesGroup(pytest.RaisesExc(ValueError, match="foo")):
        raise ExceptionGroup("", (ValueError("foo"),))
```

## Key Takeaways
1. pytest rewrites assert statements to show intermediate values on failure
2. Use `pytest.approx` for float comparisons — it handles scalars, arrays, NaNs
3. Use `pytest.raises` as context manager with optional `match` regex parameter
4. Use `RaisesGroup`/`RaisesExc` for exception groups — safer than `group_contains`
5. Custom comparison output via `pytest_assertrepr_compare` hook in conftest.py
6. Assertion rewriting only applies to test modules; use `register_assert_rewrite` for helpers

## Connects To
- **Ch 1**: Getting Started — basic assert usage
- **Ch 9**: Skipping — `xfail(raises=ExceptionType)` combines marks with exception testing
- **Ch 11**: Configuration — `PYTEST_DONT_REWRITE` docstring disables rewriting per module
