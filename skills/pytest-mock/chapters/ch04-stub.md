# Chapter 4: Stub

## Core Idea
`mocker.stub()` creates a mock that accepts any arguments — ideal for testing callbacks, event handlers, and hook functions where you only care that it was called with the right args.

## Frameworks Introduced
- **mocker.stub(name)**: Creates a `MagicMock` that accepts any arguments. Optional `name` appears in `repr` for debugging.
  - When to use: Testing callback invocations where the stub's behavior doesn't matter.
  - How: `stub = mocker.stub(name='on_complete'); do_operation(stub); stub.assert_called_once_with(expected_args)`
- **mocker.async_stub(name)**: Same as `stub()` but creates an async-compatible stub.
  - When to use: Callbacks that are `async def` functions.

## Key Concepts
- **stub**: A minimal mock — no return value setup needed, just call verification.
- **name parameter**: Makes the stub's `repr` readable in test output (e.g., `<Mock name='on_complete' id='...'>`).
- **async_stub**: Async variant for coroutine callbacks.

## Mental Models
- A stub is a "listen-only" mock — it records what was passed to it but doesn't respond.
- Use a stub when the callback's implementation is irrelevant to the test; you just need to verify it was invoked.

## Anti-patterns
- **Using stub when you need to control return value**: Use `mocker.patch()` or `mocker.MagicMock(return_value=...)` instead.
- **Overusing stubs for complex callbacks**: If the callback needs to return different values based on input, a full mock is more appropriate.

## Code Examples
```python
def test_stub(mocker):
    def foo(on_something):
        on_something('foo', 'bar')

    stub = mocker.stub(name='on_something_stub')
    foo(stub)
    stub.assert_called_once_with('foo', 'bar')

def test_async_stub(mocker):
    async def process(callback):
        await callback('data')

    stub = mocker.async_stub(name='async_callback')
    # Use in async test context
```
- **What it demonstrates**: Creating named stubs for callback verification.

## Worked Example
```python
def test_event_handler(mocker):
    class EventEmitter:
        def __init__(self):
            self.handlers = []

        def on(self, event, handler):
            self.handlers.append((event, handler))

        def emit(self, event, data):
            for evt, handler in self.handlers:
                if evt == event:
                    handler(data)

    emitter = EventEmitter()
    on_user_created = mocker.stub(name='on_user_created')
    emitter.on('user_created', on_user_created)

    emitter.emit('user_created', {'id': 1, 'name': 'Alice'})

    on_user_created.assert_called_once_with({'id': 1, 'name': 'Alice'})
```

## Key Takeaways
1. `mocker.stub()` creates a zero-configuration mock for callback verification.
2. Always pass a `name` parameter for readable test output.
3. Use `mocker.async_stub()` for async callback testing.
4. Stubs accept any arguments — they're for verification, not behavior control.

## Connects To
- **Ch 2**: Mocker Patch Methods — for mocks that need return values or side effects
- **Ch 3**: Spy — for observing real function behavior
