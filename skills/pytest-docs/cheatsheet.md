# Quick Reference

## Selection Rules

| What | CLI | Example |
|------|-----|---------|
| By name pattern | `-k "expr"` | `-k "login and not slow"` |
| By marker | `-m "marker"` | `-m "not network"` |
| Stop first fail | `-x` | `-x` |
| Last failures only | `--lf` | `--lf` |
| Failed first | `--ff` | `--ff` |
| Specific file::test | path::test | `tests/test_auth.py::TestLogin::test_valid` |
| No tests collected | exit code 5 | check `--co` output |

## Output Control

| Flag | Effect |
|------|--------|
| `-v` / `-vv` | Verbose / very verbose |
| `-q` | Quiet mode |
| `-s` | No capture (see print output) |
| `--tb=short` | Concise tracebacks |
| `--tb=long` | Full tracebacks |
| `--tb=line` | One line per failure |
| `--tb=no` | No tracebacks |
| `--co` | Collect only, don't run |

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | All tests passed |
| 1 | One or more tests failed |
| 2 | Interrupted (Ctrl+C) |
| 3 | Internal pytest error |
| 4 | Usage error (bad CLI) |
| 5 | No tests collected |

## Fixtures Quick Reference

| Fixture | Purpose | Scope |
|---------|---------|-------|
| `tmp_path` | Temporary directory (pathlib.Path) | function |
| `tmp_path_factory` | Temp dir factory for broader scopes | session |
| `monkeypatch` | Patch attributes/envvars, auto-undo | function |
| `capsys` | Capture stdout/stderr | function |
| `capfd` | Lower-level fd capture | function |
| `caplog` | Capture logging output | function |
| `recwarn` | Capture warnings | function |
| `request` | Test context introspection | function |

## Markers Quick Reference

| Marker | Purpose |
|--------|---------|
| `@pytest.mark.skip(reason=...)` | Unconditional skip |
| `@pytest.mark.skipif(cond, reason=...)` | Conditional skip |
| `@pytest.mark.xfail(reason=...)` | Expected failure |
| `@pytest.mark.xfail(raises=Exc)` | xfail on specific exception |
| `@pytest.mark.xfail(strict=True)` | Fail if unexpectedly passes |
| `@pytest.mark.parametrize(args, vals)` | Data-driven tests |
| `@pytest.fixture(autouse=True)` | Auto-run fixture |

## Configuration (pyproject.toml)

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --tb=short --strict-markers -x"
markers = [
    "slow: marks tests as slow",
    "network: marks tests requiring network",
]
```

## Common Patterns

```python
# Exception assertion
with pytest.raises(ValueError, match=r"invalid .* value"):
    parse("bad input")

# Float comparison
assert result == pytest.approx(3.14, rel=1e-3)

# Parametrize
@pytest.mark.parametrize("input,expected", [
    ("hello", 5),
    ("", 0),
])
def test_count(input, expected):
    assert word_count(input) == expected

# Skip specific parametrize value
@pytest.mark.parametrize("n", [
    1, 2,
    pytest.param(3, marks=pytest.mark.skip(reason="bug")),
])
def test_process(n):
    process(n)

# Fixture with cleanup
@pytest.fixture
def db():
    conn = connect()
    yield conn
    conn.close()

# Custom marker with data
@pytest.mark.env("production")
def test_api():
    pass

# Access marker in fixture
def check_env(request):
    marker = request.node.get_closest_marker("env")
    if marker and marker.args[0] != os.environ.get("ENV"):
        pytest.skip("wrong environment")
```
