# Patterns

## Pattern: Auto-cleanup Mocking
**When to use**: Any test that needs to replace an object with a mock.
**How**: Add `mocker` parameter; call `mocker.patch('target')`. Mocks are automatically undone at test end.
**Trade-offs**: Simpler than context managers/decorators; mock lifetime tied to test scope.

## Pattern: Spy Before Assert
**When to use**: Verifying a method was called without changing its behavior.
**How**: `spy = mocker.spy(obj, 'method'); obj.method(args); spy.assert_called_once_with(args)`
**Trade-offs**: Real code runs (integration-like); can't control return values.

## Pattern: Callback Stub
**When to use**: Testing that a callback/hook was invoked with correct arguments.
**How**: `stub = mocker.stub(name='descriptive_name'); function_under_test(stub); stub.assert_called_once_with(expected)`
**Trade-offs**: Zero config; no behavior control — use full mock if callback needs to return values.

## Pattern: Environment Variable Override
**When to use**: Tests that depend on `os.environ` or similar dicts.
**How**: `mocker.patch.dict(os.environ, {'KEY': 'value'})`
**Trade-offs**: Cleaner than `@mock.patch.dict` decorator; scoped to test lifetime.

## Pattern: Strict Signature Mocking
**When to use**: Preventing silent bugs from wrong argument counts/types.
**How**: `mocker.patch('module.func', autospec=True)`
**Trade-offs**: Catches typos and signature mismatches; may fail if original signature is dynamic.

## Pattern: Selective Unmocking
**When to use**: Mocking a method, doing work, then unmocking to test real behavior.
**How**: `mock = mocker.patch('target'); ... ; mocker.stop(mock); ...`
**Trade-offs**: Fine-grained control; more complex than simple auto-cleanup.

## Pattern: Multiple Patches in One Call
**When to use**: Patching several things in the same module.
**How**: `mocker.patch.multiple('module', name1=val1, name2=val2)`
**Trade-offs**: Reduces boilerplate; harder to capture individual mock references (use `mocker.DEFAULT`).

## Pattern: Scope-Extended Mocking
**When to use**: A mock that should persist across multiple tests (class, module, session).
**How**: Use `class_mocker`, `module_mocker`, `package_mocker`, or `session_mocker` instead of `mocker`.
**Trade-offs**: Shared state across tests; cleanup happens at scope boundary, not per-test.
