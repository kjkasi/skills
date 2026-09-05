# Cheatsheet

## Decision Rules

| Situation | Action | Why |
|-----------|--------|-----|
| Need to mock something | `mocker.patch('target')` | Auto-cleanup, flat API |
| Need to verify a call happened | `mocker.spy(obj, 'method')` | Preserves real behavior |
| Need to verify a callback was called | `mocker.stub(name='...')` | Zero-config, accepts any args |
| Need to override env vars | `mocker.patch.dict(os.environ, {...})` | Cleanest approach |
| Need strict signature checking | Add `autospec=True` | Catches wrong arg counts |
| Mocking multiple names | `mocker.patch.multiple(...)` | Single call, less boilerplate |
| Need a mock to outlive one test | Use `class_mocker` / `session_mocker` | Scope-aware fixtures |
| Need to undo mock mid-test | `mocker.stop(mock)` | Selective unmocking |
| Need to reset mock state | `mocker.resetall()` | Clears call counts |

## Where to Patch

```
# WRONG: patching where it's defined
@mock.patch('os.remove')  # os.remove is defined in os module

# RIGHT: patching where it's looked up
@mock.patch('mymodule.os.remove')  # mymodule imports os
```

Rule: Patch the name in the module that *uses* it, not where it's *defined*.

## Mock vs Spy vs Stub

| Tool | Changes behavior? | Verifies calls? | Returns configurable? |
|------|-------------------|-----------------|----------------------|
| `mocker.patch()` | Yes | Yes | Yes |
| `mocker.spy()` | No | Yes | No (real code runs) |
| `mocker.stub()` | N/A (no real impl) | Yes | No (accepts anything) |

## Type Annotation Quick Reference

```python
from pytest_mock import MockerFixture

def test_example(mocker: MockerFixture) -> None:
    ...
```

Always annotate `mocker: MockerFixture` — costs nothing, gains IDE support + mypy checking.

## Configuration Quick Reference

```ini
[pytest]
mock_use_standalone_module = true   # Use PyPI mock (default: false)
mock_traceback_monkeypatch = false  # Disable improved tracebacks (default: true)
```

## Common Gotchas

| Gotcha | Fix |
|--------|-----|
| `with mocker.patch(...)` emits warning | Don't use as context manager — call `mocker.patch()` directly |
| Mock accepts wrong arguments | Add `autospec=True` |
| Mock doesn't clean up between tests | Ensure `mocker` is a parameter, not a local variable |
| `spy_return` vs `return_value` confusion | `spy_return` = real return value; `return_value` = mock's configured return |
| `--tb=native` breaks traceback improvement | Expected — monkeypatch is auto-disabled |
