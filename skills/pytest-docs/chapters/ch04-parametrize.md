# Chapter 4: Parametrize

## Core Idea
`@pytest.mark.parametrize` runs the same test logic against multiple input sets, generating separate test cases for each combination without duplicating code.

## Frameworks Introduced
- **Parametrize Decorator**: `@pytest.mark.parametrize(argnames, argvalues)` generates test variants from input data
- **Indirect Parametrize**: `indirect=True` routes parametrize values through a fixture, enabling fixture-level configuration
- **Param IDs**: Customize test IDs with `ids=` parameter for readable output and `-k` selection
- **pytest.param**: Wrap individual values to apply marks (skip, xfail) per-parameter-set

## Key Concepts
- **argnames**: String or list of parameter names matching test function arguments
- **argvalues**: List of tuples/lists, each producing one test invocation
- **Cross-product**: Multiple parametrize decorators combine as Cartesian product
- **indirect**: Route values through a named fixture instead of directly to test parameters
- **ids**: List of strings or callable to name each parametrized test case
- **pytest.param(value, marks=...)**: Apply per-value marks like `skip` or `xfail`

## Mental Models
- Think of parametrize as "for each value, run this test function with these arguments"
- Use `pytest.param` to mark individual cases: `pytest.param(1, marks=pytest.mark.skip)`
- Cross-product: two `@parametrize` decorators on one test = all combinations
- IDs control test names: use for `-k` filtering and readable failure reports

## Anti-patterns
- **Over-parametrizing**: Too many combinations create slow, hard-to-debug test suites
- **Using parametrize for fixture selection**: Use `indirect=True` or fixture params instead
- **Ignoring IDs**: Auto-generated IDs like `[0]` are unreadable — use `ids=` or `pytest.param`

## Code Examples
```python
# Basic parametrize
@pytest.mark.parametrize("input,expected", [
    ("hello", 5),
    ("", 0),
    ("a b", 3),
])
def test_word_count(input, expected):
    assert word_count(input) == expected

# Multiple parameters (cross-product)
@pytest.mark.parametrize("x", [0, 1])
@pytest.mark.parametrize("y", [2, 3])
def test_multiply(x, y):
    assert x * y in [0, 2, 3]

# Indirect parametrize (route through fixture)
@pytest.fixture
def db_connection(request):
    return create_connection(request.param)

@pytest.mark.parametrize("db_connection", ["mysql", "postgres"], indirect=True)
def test_query(db_connection):
    assert db_connection.execute("SELECT 1") == [(1,)]

# Param with marks
@pytest.mark.parametrize("input,expected", [
    (1, 2),
    pytest.param(2, 5, marks=pytest.mark.xfail(reason="known bug")),
])
def test_increment(input, expected):
    assert increment(input) == expected

# Custom IDs
@pytest.mark.parametrize("input,expected", [
    ("hello", 5),
    ("", 0),
], ids=["non-empty", "empty"])
def test_word_count(input, expected):
    assert word_count(input) == expected
```

## Key Takeaways
1. `@pytest.mark.parametrize("arg", [values])` generates one test per value
2. Multiple decorators combine as Cartesian product (cross-product)
3. Use `indirect=True` to route values through a fixture
4. Use `pytest.param` to apply marks per-value (skip, xfail)
5. Use `ids=` for readable test names and `-k` filtering
6. Combine with fixtures: parametrize test args or parametrize fixture params

## Connects To
- **Ch 3**: Fixtures — fixture parametrization via `params=` vs test parametrization
- **Ch 5**: Markers — `pytest.param` applies marks to individual parametrize values
- **Ch 9**: Skipping — `pytest.param(2, marks=pytest.mark.skip)` for conditional skipping
