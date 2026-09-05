# Chapter 3: Spy

## Core Idea
`mocker.spy()` observes real function/method calls without replacing the original implementation — it tracks calls, return values, and exceptions while the real code still runs.

## Frameworks Introduced
- **mocker.spy(obj, method_name)**: Wraps a method to track calls while preserving original behavior.
  - When to use: You want to verify a method was called (and with what args) without changing its behavior.
  - How: `spy = mocker.spy(obj, 'method'); result = obj.method(args); spy.assert_called_once_with(args)`
- **mocker.spy with duplicate_iterators**: When `duplicate_iterators=True`, `spy_return_iter` uses `itertools.tee` to duplicate iterator return values.
  - When to use: Spying on functions that return iterators.

## Key Concepts
- **SpyType**: The object returned by `mocker.spy()` — subclasses `MagicMock` with extra attributes.
- **spy_return**: The last returned value of the spied function.
- **spy_return_iter**: Duplicate of the last returned value if it was an iterator (requires `duplicate_iterators=True`).
- **spy_return_list**: A list of all returned values across calls (new in 3.13).
- **spy_exception**: The last exception raised by the spied function, or `None`.
- **unspy**: Selectively stop spying with `mocker.stop(spy)` (since v3.10).

## Mental Models
- Think of spy as a transparent wrapper — the real code runs, but you get call metadata for free.
- Use spy when you need to *observe* behavior (was it called? what did it return?) without *changing* it.
- Use mock when you need to *control* behavior (return value, side effect, raise exception).

## Anti-patterns
- **Using spy when you need to change return values**: Spy runs the real code; use `mocker.patch()` instead.
- **Confusing spy_return with return_value**: In pre-2.0 versions these were named `return_value` and `side_effect`, but were renamed to avoid conflicts with `unittest.mock`.

## Code Examples
```python
def test_spy_method(mocker):
    class Foo:
        def bar(self, v):
            return v * 2

    foo = Foo()
    spy = mocker.spy(foo, 'bar')
    assert foo.bar(21) == 42

    spy.assert_called_once_with(21)
    assert spy.spy_return == 42

def test_spy_function(mocker):
    import mymodule
    spy = mocker.spy(mymodule, "myfunction")
    assert mymodule.myfunction() == 42
    assert spy.call_count == 1
    assert spy.spy_return == 42
```
- **What it demonstrates**: Spying on both methods and functions; accessing call metadata.

## Worked Example
```python
def test_unspy(mocker):
    class Foo:
        def bar(self):
            return 42

    spy = mocker.spy(Foo, "bar")
    foo = Foo()

    # Spy is active — tracks calls
    assert foo.bar() == 42
    assert spy.call_count == 1

    # Stop spying — real method still works, but no tracking
    mocker.stop(spy)
    assert foo.bar() == 42
    assert spy.call_count == 1  # count frozen after stop
```
This demonstrates selective unspy — useful when you want to verify a call happened, then let subsequent calls run untracked.

## Key Takeaways
1. `mocker.spy()` preserves the original implementation — it observes, not replaces.
2. Access return values via `spy.spy_return` and exceptions via `spy.spy_exception`.
3. `spy_return_list` (v3.13+) captures all return values across multiple calls.
4. Use `mocker.stop(spy)` to selectively stop spying mid-test.
5. Spy works with regular methods, class methods, static methods, and `async def` functions.

## Connects To
- **Ch 2**: Mocker Patch Methods — when you need to *control* behavior instead of observe
- **Ch 4**: Stub — when you need a simple callback mock
