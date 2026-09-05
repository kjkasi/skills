# Glossary

**assert introspection** — pytest rewrites assert statements to show intermediate values on failure (Ch 2)

**autouse fixture** — fixture that runs automatically without explicit request via `@pytest.fixture(autouse=True)` (Ch 3)

**capfd** — lower-level capture fixture using file descriptors, catches C-level output (Ch 7)

**caplog** — fixture capturing Python logging module output with level filtering (Ch 8)

**capsys** — fixture capturing stdout/stderr output via `capsys.readouterr()` (Ch 7)

**conftest.py** — project-local plugin file for shared fixtures and hooks, auto-discovered by pytest (Ch 10)

**cross-product** — multiple `@parametrize` decorators combine as Cartesian product (Ch 4)

**dynamic scope** — pass a callable to `scope=` to determine fixture scope at definition time (Ch 3)

**entry point plugin** — installed package with `pytest11` entry point, auto-loaded by pytest (Ch 10)

**factory as fixture** — pattern where fixture returns a callable to generate multiple instances (Ch 3)

**fixture** — function decorated with `@pytest.fixture` providing test setup/teardown and dependency injection (Ch 3)

**fixture caching** — fixtures execute once per scope; repeated requests return the same instance (Ch 3)

**fixture scope** — controls fixture lifetime: `function`, `class`, `module`, `package`, `session` (Ch 3)

**hook** — function named `pytest_*` that pytest calls at specific lifecycle points (Ch 10)

**indirect parametrize** — `indirect=True` routes parametrize values through a named fixture (Ch 4)

**monkeypatch** — fixture for safe attribute/envvar patching with automatic undo (Ch 7)

**parametrize** — `@pytest.mark.parametrize` generates test variants from input data (Ch 4)

**pytest.approx** — helper for floating-point comparisons with tolerance (Ch 1)

**pytest.raises** — context manager for asserting exceptions are raised (Ch 1)

**pytest.warns** — context manager for asserting warnings are raised (Ch 2)

**RaisesExc** — for asserting individual exceptions within exception groups (Ch 2)

**RaisesGroup** — for asserting `ExceptionGroup`/`BaseExceptionGroup` with structure matching (Ch 2)

**recwarn** — fixture capturing warnings for inspection (Ch 7)

**src layout** — project structure with source in `src/package/` and tests in `tests/` (Ch 13)

**test class** — class prefixed with `Test` containing test methods, no subclass required (Ch 1)

**test function** — function prefixed with `test_` using plain assert statements (Ch 1)

**test discovery** — pytest auto-collects `test_*.py`/`*_test.py` files and `test_` prefixed functions (Ch 1)

**tmp_path** — built-in fixture providing unique temporary directory per test (Ch 7)

**tmp_path_factory** — session-scoped factory for creating temporary directories (Ch 7)

**xfail** — `@pytest.mark.xfail` marks expected failures; test still runs (Ch 9)

**yield fixture** — fixture using `yield` instead of `return`; code after yield is teardown (Ch 3)
