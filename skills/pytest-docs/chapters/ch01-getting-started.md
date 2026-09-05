# Chapter 1: Getting Started

## Core Idea
pytest is installed via pip, discovers tests automatically by naming conventions, and uses plain assert statements with rich introspection on failure.

## Frameworks Introduced
- **Test Discovery**: pytest collects `test_*.py` and `*_test.py` files, finds functions/methods prefixed with `test_`, runs them automatically
- **Assertion Introspection**: pytest rewrites assert statements to show intermediate values on failure, eliminating the need for `self.assertEqual` style methods

## Key Concepts
- **Test function**: A function prefixed with `test_` that uses plain `assert` statements
- **Test class**: A class prefixed with `Test` containing test methods (no subclass required)
- **pytest.raises**: Context manager for asserting exceptions are raised
- **pytest.approx**: Helper for floating-point comparisons with tolerance
- **tmp_path fixture**: Built-in fixture providing a unique temporary directory per test

## Mental Models
- Use `test_*.py` naming convention so pytest auto-discovers your tests. No test runner config needed.
- Think of pytest as "just write functions with assert" — the framework handles everything else.
- Use `pytest.raises` as a context manager: `with pytest.raises(Exception): code()`

## Anti-patterns
- **Returning bool from tests**: pytest ignores return values; always use `assert` — returning `True/False` silently passes
- **Subclassing TestCase**: Unnecessary in pytest; use plain classes prefixed with `Test` for grouping only
- **Using self.assert* methods**: Plain `assert` gives better introspection and shorter code

## Code Examples
```python
# Basic test function
def func(x):
    return x + 1

def test_answer():
    assert func(3) == 5  # Fails with rich diff

# Testing exceptions
import pytest
def test_zero_division():
    with pytest.raises(ZeroDivisionError):
        1 / 0

# Approximate float comparison
def test_floats():
    assert (0.1 + 0.2) == pytest.approx(0.3)

# Temporary directory
def test_needsfiles(tmp_path):
    d = tmp_path / "sub"
    d.mkdir()
    p = d / "hello.txt"
    p.write_text("content")
    assert p.read_text() == "content"
```

## Key Takeaways
1. Install with `pip install -U pytest`, verify with `pytest --version`
2. Name test files `test_*.py`, test functions `test_*`, test classes `Test*`
3. Use plain `assert` — pytest shows detailed introspection on failure
4. Use `pytest.raises` for exception testing, `pytest.approx` for float comparison
5. Use `tmp_path` for temporary files, `--fixtures` to list all available fixtures

## Connects To
- **Ch 2**: Assertions — deeper introspection and custom comparison hooks
- **Ch 3**: Fixtures — `tmp_path` is one of many built-in fixtures
- **Ch 6**: Usage — CLI options for test selection and reporting
