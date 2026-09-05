# Glossary

**ANY** — A sentinel that matches any single argument in mock call assertions (Ch 2)

**AsyncMock** — A mock subclass designed for mocking `async def` functions (Ch 2)

**autospec** — Creates mocks that match the original object's signature, raising `TypeError` for invalid calls (Ch 2)

**call** — A helper object representing a mock call; used in `assert_called_with()` comparisons (Ch 2)

**class_mocker** — A fixture-scoped mock that persists across all tests in a class (Ch 2)

**DEFAULT** — A sentinel indicating the default mock behavior should be used in `patch.multiple()` (Ch 2)

**MagicMock** — The default mock type; supports magic methods like `__len__`, `__iter__`, `__enter__` (Ch 2)

**Mock** — The base mock class; does not support magic methods by default (Ch 2)

**mock_use_standalone_module** — Config option to use the PyPI `mock` package instead of `unittest.mock` (Ch 5)

**mock_traceback_monkeypatch** — Config option controlling whether internal mock frames are hidden in tracebacks (Ch 5)

**mock_open** — A helper that creates a mock for `open()` to test file I/O (Ch 2)

**mocker** — The central pytest fixture providing patching, spying, and stubbing utilities (Ch 1)

**mocker.patch()** — Replaces an object with a `MagicMock` for the duration of a test (Ch 2)

**mocker.patch.object()** — Patches a specific attribute on an existing object (Ch 2)

**mocker.patch.multiple()** — Patches multiple names in a single call (Ch 2)

**mocker.patch.dict()** — Patches a dictionary, commonly used for `os.environ` (Ch 2)

**mocker.spy()** — Observes real function calls without replacing the implementation (Ch 3)

**mocker.stub()** — Creates a minimal mock that accepts any arguments; ideal for callback testing (Ch 4)

**mocker.stopall()** — Stops all active mocks in the current test (Ch 2)

**mocker.stop()** — Stops a specific mock or spy (Ch 2)

**mocker.resetall()** — Resets call counts and state on all mocks (Ch 2)

**module_mocker** — A fixture-scoped mock that persists across all tests in a module (Ch 2)

**package_mocker** — A fixture-scoped mock that persists across all tests in a package (Ch 2)

**PropertyMock** — A mock subclass for mocking properties on classes (Ch 2)

**seal()** — Prevents further attribute access on a mock, catching accidental usage (Ch 6)

**sentinel** — A unique object used as a marker or placeholder in mock call assertions (Ch 2)

**session_mocker** — A fixture-scoped mock that persists across all tests in a session (Ch 2)

**spy_return** — The last returned value of a spied function (Ch 3)

**spy_return_iter** — Iterator duplicate of the last return value (requires `duplicate_iterators=True`) (Ch 3)

**spy_return_list** — A list of all returned values across calls (new in v3.13) (Ch 3)

**spy_exception** — The last exception raised by a spied function, or `None` (Ch 3)

**SpyType** — The type returned by `mocker.spy()`, subclassing `MagicMock` with extra attributes (Ch 3)

**stub** — A zero-configuration mock for verifying callback invocations (Ch 4)

**unittest.mock** — Python's built-in mocking library that pytest-mock wraps (Ch 1)
