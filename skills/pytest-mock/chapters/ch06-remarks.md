# Chapter 6: Remarks & Type Annotations

## Core Idea
pytest-mock is fully type-annotated for static type checking, and exists because the standard `mock.patch` API doesn't scale well beyond one or two patches in a test.

## Frameworks Introduced
- **MockerFixture type annotation**: Use `pytest_mock.MockerFixture` to annotate `mocker` parameters for static type checking.
  - When to use: Always, if you use `mypy` or another type checker.
  - How: `def test_foo(mocker: MockerFixture) -> None: ...`
- **seal()**: Prevents further attribute access on a mock, catching accidental usage after mock setup.
  - When to use: When you want strict control over which attributes are accessed on a mock.
  - How: `mocker.patch('module.MockClass'); mock.seal(module.MockClass)`

## Key Concepts
- **type annotations**: `mocker` returns `pytest_mock.MockerFixture` — annotate test parameters for IDE support and type checking.
- **mypy support**: The only tested type checker; others may work but aren't officially supported.
- **why pytest-mock exists**: Standard `mock.patch` has three usage patterns (context manager, decorator, direct), and they don't compose well:
  - Context managers → excessive nesting
  - Decorators → parameter ordering issues, can't mix with pytest parametrize
  - `ExitStack` → more complex than necessary
- **pytest-mock solution**: Single `mocker` fixture with flat API, no nesting or decoration needed.

## Mental Models
- Type annotate `mocker: MockerFixture` as a habit — it costs nothing and improves IDE support.
- The plugin exists because `mock.patch` patterns fight pytest's fixture model; `mocker` aligns mocking with pytest's philosophy.

## Anti-patterns
- **Not using type annotations**: Loses IDE autocompletion and static checking benefits.
- **Mixing `mock.patch` decorators with pytest fixtures**: The parameter ordering becomes confusing; use `mocker` instead.

## Code Examples
```python
from pytest_mock import MockerFixture

def test_foo(mocker: MockerFixture) -> None:
    mocker.patch('os.remove')
    # IDE knows all mocker methods; mypy checks types
```
- **What it demonstrates**: Proper type annotation for the mocker fixture.

## Worked Example
```python
# The problem pytest-mock solves:
import mock

def test_before():
    with mock.patch('os.remove'):
        with mock.patch('os.listdir'):
            with mock.patch('shutil.copy'):
                # Deep nesting, hard to read
                pass

# pytest-mock's solution:
def test_after(mocker: MockerFixture) -> None:
    mocker.patch('os.remove')
    mocker.patch('os.listdir')
    mocker.patch('shutil.copy')
    # Flat, readable, pytest-native
```

## Key Takeaways
1. Always annotate `mocker: MockerFixture` for type safety and IDE support.
2. pytest-mock exists because `mock.patch` context managers and decorators don't scale.
3. The `mocker` fixture eliminates nesting, parameter ordering issues, and fixture conflicts.
4. `mypy` is the only officially tested type checker.

## Connects To
- **Ch 1**: Introduction — the mocker fixture in context
- **Ch 2**: Mocker Patch Methods — the API that makes patching flat
