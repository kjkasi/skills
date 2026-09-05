# Chapter 1: Introduction & Install

## Core Idea
pytest-mock provides a `mocker` fixture that wraps Python's `unittest.mock` patching API, eliminating context manager nesting and decorator complexity in tests.

## Frameworks Introduced
- **mocker fixture**: A pytest fixture that provides a thin wrapper around `mock.patch` with automatic cleanup after each test.
  - When to use: Any test that needs to mock, stub, or spy on objects.
  - How: Add `mocker` as a test function parameter; call `mocker.patch()`, `mocker.spy()`, or `mocker.stub()`.

## Key Concepts
- **mocker**: The central fixture; acts as a namespace for all patching/spying/stubbing operations.
- **unittest.mock**: Python's built-in mocking library that pytest-mock wraps.
- **patch**: Replace an object with a mock temporarily during a test.
- **automatic cleanup**: All mocks are undone when the test ends, no manual teardown needed.

## Mental Models
- Think of `mocker` as a test-scoped context manager that never needs explicit `with` blocks.
- Use `mocker` when you have more than one mock to apply — it avoids decorator stacking and parameter ordering issues.

## Anti-patterns
- **Using mocker as a context manager**: `with mocker.patch(...)` emits a warning and defeats the plugin's purpose.
- **Patching where the object is used vs. where it's defined**: Always patch the name in the module where it's looked up, not where it's defined.

## Code Examples
```python
import os

class UnixFS:
    @staticmethod
    def rm(filename):
        os.remove(filename)

def test_unix_fs(mocker):
    mocker.patch('os.remove')
    UnixFS.rm('file')
    os.remove.assert_called_once_with('file')
```
- **What it demonstrates**: Basic mock patching with automatic cleanup — no `with` or `@` needed.

## Worked Example
```python
# Before pytest-mock (nested context managers):
def test_before():
    with mock.patch('os.remove'):
        with mock.patch('os.listdir'):
            # test code with awkward nesting
            pass

# After pytest-mock (flat, readable):
def test_after(mocker):
    mocker.patch('os.remove')
    mocker.patch('os.listdir')
    # test code flows naturally
```
The `mocker` fixture flattens the test structure, making it easier to read and maintain.

## Key Takeaways
1. Install with `pip install pytest-mock` — no configuration required.
2. Add `mocker` as a parameter to any test function to access mocking utilities.
3. All mocks are automatically cleaned up after each test — no manual teardown.
4. The `mocker` fixture works identically to `mock.patch` API but without context managers or decorators.

## Connects To
- **Ch 2**: Mocker Patch Methods — the core patching API details
- **Ch 5**: Configuration — customizing pytest-mock behavior
