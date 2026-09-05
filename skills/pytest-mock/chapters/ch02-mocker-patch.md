# Chapter 2: Mocker Fixture & Patch Methods

## Core Idea
The `mocker` fixture exposes the full `mock.patch` API as methods, plus convenience access to mock objects (`Mock`, `MagicMock`, `ANY`, `call`, etc.) and scope-aware fixtures for class/module/package/session-level mocking.

## Frameworks Introduced
- **mocker.patch()**: Replaces an object with a `MagicMock`. Supports `new`, `autospec`, `spec`, `side_effect`, `return_value`.
  - When to use: Mocking a function, method, or attribute in a module.
  - How: `mocker.patch('module.path.to.name', **kwargs)`
- **mocker.patch.object()**: Patches a specific attribute on an object.
  - When to use: Mocking a method on a class instance or a module attribute.
  - How: `mocker.patch.object(obj, 'attr', **kwargs)`
- **mocker.patch.multiple()**: Patches multiple names in a single call.
  - When to use: When you need to mock several things in the same module.
  - How: `mocker.patch.multiple('module', name1=mock1, name2=mock2)`
- **mocker.patch.dict()**: Patches a dictionary (e.g., `os.environ`).
  - When to use: Setting environment variables or config dicts for a test.
  - How: `mocker.patch.dict(os.environ, {'KEY': 'value'})`
- **mocker.stopall()**: Stops all active mocks.
  - When to use: When you need to undo mocking mid-test.
- **mocker.stop()**: Stops a specific mock.
  - When to use: Selective unmocking (e.g., spy → stop spy → verify original behavior).
- **mocker.resetall()**: Resets call counts and state on all mocks.
  - When to use: When you need a clean mock state within a single test.

## Key Concepts
- **autospec**: Creates mocks that match the original object's signature; raises `TypeError` for invalid calls.
- **scope-aware fixtures**: `class_mocker`, `module_mocker`, `package_mocker`, `session_mocker` — apply mocks at broader scopes.
- **convenience names**: `Mock`, `MagicMock`, `PropertyMock`, `AsyncMock`, `ANY`, `DEFAULT`, `call`, `sentinel`, `mock_open`, `seal` are directly accessible from `mocker`.

## Mental Models
- `mocker.patch()` is equivalent to `mock.patch()` but auto-cleanup happens at test end, not context exit.
- Use `autospec=True` when you want strict signature validation — it catches typos and wrong argument counts.
- Scope-aware fixtures (`class_mocker`, etc.) are for when you want a mock to persist across multiple tests.

## Anti-patterns
- **Forgetting autospec**: Without it, mocks accept any arguments, hiding bugs.
- **Patching built-ins directly**: `mocker.patch('builtins.open')` works but patching at the usage site is clearer.
- **Using mocker as context manager**: Emits a warning; defeats the plugin's purpose.

## Code Examples
```python
def test_patch_methods(mocker):
    # Basic patch
    mocker.patch('os.remove')

    # Patch with autospec
    mocker.patch.object(os, 'listdir', autospec=True)

    # Patch dict (e.g., environment variables)
    mocker.patch.dict(os.environ, {'MY_VAR': 'test_value'})

    # Multiple patches in one call
    mocker.patch.multiple('os',
        remove=mocker.DEFAULT,
        listdir=mocker.DEFAULT,
    )

    # Stop a specific mock mid-test
    mock_remove = mocker.patch('os.remove')
    # ... do work ...
    mocker.stop(mock_remove)
```
- **What it demonstrates**: The range of patching methods available through `mocker`.

## Reference Tables

| Method | Use Case | Returns |
|--------|----------|---------|
| `mocker.patch(target)` | Mock a function/attribute by path | `MagicMock` |
| `mocker.patch.object(obj, name)` | Mock an attribute on an object | `MagicMock` |
| `mocker.patch.multiple(target, **kwargs)` | Mock multiple names at once | `dict` of mocks |
| `mocker.patch.dict(d, values)` | Patch a dictionary | `dict` context |
| `mocker.stopall()` | Stop all active mocks | `None` |
| `mocker.stop(mock)` | Stop a specific mock | `None` |
| `mocker.resetall()` | Reset all mock states | `None` |

## Worked Example
```python
import os
from pytest_mock import MockerFixture

def test_env_var_override(mocker: MockerFixture) -> None:
    # Save original and override
    mocker.patch.dict(os.environ, {'DATABASE_URL': 'sqlite:///:memory:'})

    # Code under test reads os.environ['DATABASE_URL']
    result = get_database_url()
    assert result == 'sqlite:///:memory:'

    # Mock is automatically cleaned up after test
```

## Key Takeaways
1. `mocker.patch()` mirrors `mock.patch` API — same arguments, same behavior, but auto-cleanup.
2. Use `autospec=True` to get strict signature checking on mocks.
3. `mocker.patch.dict()` is the cleanest way to override environment variables.
4. Scope-aware fixtures (`class_mocker`, `session_mocker`) extend mock lifetime beyond a single test.
5. `mocker.stop()` and `mocker.stopall()` let you undo mocking mid-test when needed.

## Connects To
- **Ch 1**: Introduction — how the mocker fixture fits into pytest
- **Ch 3**: Spy — observing real behavior without replacing it
- **Ch 4**: Stub — simple callback mocking
- **Ch 5**: Configuration — customizing mock behavior
