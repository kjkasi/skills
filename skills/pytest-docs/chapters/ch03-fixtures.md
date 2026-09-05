# Chapter 3: Fixtures

## Core Idea
Fixtures are pytest's dependency injection system — functions decorated with `@pytest.fixture` that provide test setup/teardown, can request other fixtures, and are cached per-scope for isolation.

## Frameworks Introduced
- **Fixture Request Pattern**: Tests declare dependencies as function arguments; pytest resolves and injects matching fixtures
- **Scope System**: `function` (default), `class`, `module`, `package`, `session` — controls fixture lifetime and sharing
- **Yield Fixtures**: Use `yield` instead of `return` to define teardown code that runs after the test
- **Autouse Fixtures**: `autouse=True` makes fixtures run automatically without explicit request
- **Factory as Fixture**: Return a callable from a fixture to generate multiple instances

## Key Concepts
- **Fixture scope**: Determines lifetime — `function`=per-test, `module`=per-file, `session`=per-run
- **Fixture caching**: Fixtures are executed once per scope; repeated requests get the same instance
- **Fixture composition**: Fixtures can request other fixtures, building dependency trees
- **request object**: Provides access to test context — `request.module`, `request.node`, `request.param`
- **addfinalizer**: Alternative to yield for teardown — `request.addfinalizer(cleanup_func)`
- **Dynamic scope**: Pass a callable to `scope=` to determine scope at definition time

## Mental Models
- Think of fixtures as "setup functions that pytest calls automatically based on parameter names"
- Use scope to balance speed vs isolation: `session` for expensive setup, `function` for test isolation
- Yield fixtures are generators: code before `yield` is setup, code after is teardown (reverse order)
- Fixture dependencies form a DAG — pytest resolves execution order automatically

## Anti-patterns
- **Monolithic setup fixtures**: Split into single-purpose fixtures for better reuse and error isolation
- **session fixture using module fixture**: Broader scope cannot depend on narrower scope
- **Shared mutable state across tests**: Each test gets its own fixture instance by default — leverage this for isolation
- **Forgetting teardown order**: Teardown runs in reverse of setup; last fixture to set up tears down first

## Code Examples
```python
# Basic fixture with teardown
@pytest.fixture
def SMTP():
    server = smtplib.SMTP("smtp.gmail.com", 587, timeout=5)
    yield server
    server.quit()

# Scoped fixture (shared across module)
@pytest.fixture(scope="module")
def smtp_connection():
    return smtplib.SMTP("smtp.gmail.com", 587, timeout=5)

# Autouse fixture
@pytest.fixture(autouse=True)
def append_first(order, first_entry):
    return order.append(first_entry)

# Factory as fixture
@pytest.fixture
def make_customer_record():
    created_records = []
    def _make_customer_record(name):
        record = Customer(name=name, orders=[])
        created_records.append(record)
        return record
    yield _make_customer_record
    for record in created_records:
        record.destroy()

# Parametrized fixture
@pytest.fixture(params=["smtp.gmail.com", "mail.python.org"])
def smtp_connection(request):
    return smtplib.SMTP(request.param, 587, timeout=5)

# Dynamic scope
def determine_scope(fixture_name, config):
    if config.getoption("--keep-containers", None):
        return "session"
    return "function"

@pytest.fixture(scope=determine_scope)
def docker_container():
    yield spawn_container()
```

## Key Takeaways
1. Fixtures are requested by parameter name — pytest auto-discovers and injects them
2. Use `yield` for teardown code — it runs in reverse order of setup
3. Scope controls lifetime: `function` (default), `class`, `module`, `package`, `session`
4. Fixtures can request other fixtures — build composable dependency trees
5. Use `autouse=True` for fixtures that should always run
6. Factory pattern: return a callable from a fixture to create multiple instances
7. Safe teardown: keep fixtures single-purpose so failures don't skip cleanup

## Connects To
- **Ch 7**: Builtin Fixtures — `tmp_path`, `monkeypatch`, `capsys` are all fixtures
- **Ch 4**: Parametrize — `@pytest.mark.parametrize` vs fixture parametrization
- **Ch 13**: Good Practices — conftest.py as fixture repository
