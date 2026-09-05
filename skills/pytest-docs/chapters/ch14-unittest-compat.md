# Chapter 14: Unittest Compatibility

## Core Idea
pytest can run unittest.TestCase tests natively, supports xunit-style setup/teardown methods, and allows gradual migration from unittest to pytest by running both side-by-side.

## Frameworks Introduced
- **Unittest Runner**: pytest discovers and runs `unittest.TestCase` subclasses without changes
- **xunit-style Setup**: `setup_method`/`teardown_method`, `setup_class`/`teardown_class`, `setup_module`/`teardown_module`
- **setup/teardown functions**: Module-level `setup_function`/`teardown_function` for non-class tests
- **Mixin pattern**: Use unittest mixins with pytest fixtures via `@pytest.fixture(autouse=True)`

## Key Concepts
- **unittest.TestCase**: pytest discovers these automatically — no runner change needed
- **setup_method(self)**: Called before each test method in a class
- **teardown_method(self)**: Called after each test method
- **setup_class(cls)**: Called once before all tests in class (classmethod)
- **teardown_class(cls)**: Called once after all tests in class
- **setup_module / teardown_module**: Module-level setup/teardown
- **pytest fixtures + unittest**: Use `@pytest.fixture(autouse=True)` in TestCase subclasses

## Mental Models
- pytest is a superset of unittest — it runs unittest tests plus more
- Migrate gradually: run existing unittest suite with pytest, then convert file by file
- xunit-style setup/teardown is legacy — prefer pytest fixtures for new code
- `setup_method` ≠ `setUp` — pytest uses different method names

## Anti-patterns
- **Rewriting all tests at once**: Migrate incrementally — pytest runs unittest natively
- **Using setUp/tearDown in new code**: Use pytest fixtures instead — more composable
- **Mixing unittest assertions with pytest**: Use plain `assert` — better introspection

## Code Examples
```python
# unittest.TestCase — runs natively in pytest
class TestMyStuff(unittest.TestCase):
    def setUp(self):
        self.data = [1, 2, 3]

    def test_length(self):
        assert len(self.data) == 3

# xunit-style setup/teardown
class TestExample:
    def setup_method(self):
        self.data = []

    def teardown_method(self):
        self.data = None

    def test_append(self):
        self.data.append(1)
        assert self.data == [1]

# Module-level setup
def setup_module():
    global db
    db = connect_database()

def teardown_module():
    db.close()

# Mixing fixtures with TestCase
class TestWithFixtures(unittest.TestCase):
    @pytest.fixture(autouse=True)
    def setup_fixtures(self, tmp_path, monkeypatch):
        self.tmp_path = tmp_path
        self.monkeypatch = monkeypatch

    def test_something(self):
        assert self.tmp_path.exists()
```

## Key Takeaways
1. pytest runs unittest.TestCase tests natively — no changes needed
2. `setup_method`/`teardown_method` for per-test setup in classes
3. `setup_class`/`teardown_class` for per-class setup (classmethod)
4. `setup_module`/`teardown_module` for per-module setup
5. Prefer pytest fixtures over xunit-style setup for new code
6. Migrate gradually: pytest runs both unittest and pytest-style tests

## Connects To
- **Ch 3**: Fixtures — prefer fixtures over xunit-style setup
- **Ch 1**: Getting Started — pytest discovers tests automatically
- **Ch 13**: Good Practices — modern test patterns
