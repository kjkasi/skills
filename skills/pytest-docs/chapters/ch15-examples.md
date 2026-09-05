# Chapter 15: Working Examples

## Core Idea
The pytest documentation includes practical examples covering markers, parametrize, non-python tests, test collection customization, and real-world reporting patterns.

## Frameworks Introduced
- **Marker Examples**: Custom markers with parameters, class-level markers, marker inheritance
- **Parametrize Patterns**: Cross-product testing, indirect parametrize, parameter IDs
- **Non-python Tests**: Testing non-Python code with `--runpytest=run-python" or custom collectors
- **Collection Customization**: `conftest.py` hooks to modify test collection, `pytest_collect_modifyitems`

## Key Concepts
- **Custom markers with args**: `@pytest.mark.env("production")` — access via `request.node.get_closest_marker("env")`
- **Parametrize cross-product**: Multiple `@parametrize` decorators combine as Cartesian product
- **conftest collection hooks**: `pytest_collect_file`, `pytest_pycollect_makemodule` for custom collection
- **Reporting demo**: Detailed assertion introspection examples for sets, dicts, strings

## Mental Models
- Markers can carry data — use `mark.args` and `mark.kwargs` to pass configuration
- Cross-product parametrize: two decorators = all combinations (like nested for-loops)
- Custom collectors let pytest test anything — not just Python functions

## Anti-patterns
- **Not leveraging marker args**: Markers can carry data — use this for test configuration
- **Ignoring collection hooks**: They're powerful for custom test discovery patterns

## Code Examples
```python
# Markers with arguments
@pytest.mark.env("production")
@pytest.mark.timeout(30)
def test_api_call():
    pass

# Access marker args in fixture
@pytest.fixture(autouse=True)
def check_env(request):
    marker = request.node.get_closest_marker("env")
    if marker:
        env = marker.args[0]
        if env != os.environ.get("TEST_ENV"):
            pytest.skip(f"not for {env}")

# Cross-product parametrize
@pytest.mark.parametrize("x", [0, 1])
@pytest.mark.parametrize("y", [2, 3])
def test_multiply(x, y):
    assert x * y in [0, 2, 3]

# Custom collection hook
def pytest_collect_file(parent, file_path):
    if file_path.suffix == ".rst" and file_path.name.startswith("test_"):
        return RSTFile.from_parent(parent, path=file_path)
```

## Key Takeaways
1. Markers can carry data via args/kwargs — access with `get_closest_marker`
2. Cross-product: multiple `@parametrize` = all combinations
3. Custom collectors let pytest test any file type
4. conftest hooks modify collection dynamically
5. Real examples: markers, parametrize, non-python, collection, reporting

## Connects To
- **Ch 5**: Markers — custom markers with arguments
- **Ch 4**: Parametrize — cross-product and indirect patterns
- **Ch 10**: Plugins — collection hooks as plugin extension points
