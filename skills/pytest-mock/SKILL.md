---
name: pytest-mock
description: "Knowledge base from \"pytest-mock Documentation\" by pytest-dev. Use when writing tests with pytest-mock, applying mocker fixture patterns, configuring mock behavior, spying on functions, or referencing the mock/spy/stub API."
---

<!-- argument-hint: [mocker method, topic, or chapter number] -->

# pytest-mock Documentation
**Author**: pytest-dev | **Pages**: ~20 | **Chapters**: 6 | **Generated**: 2026-09-03

## How to Use This Skill

- **Without arguments** — load core mocker fixture patterns and API reference
- **With a topic** — ask about `spy`, `stub`, `patch`, `autospec`, or another indexed topic
- **With chapter** — ask for `ch02`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

### mocker.patch(target, **kwargs)
Replace an object with a `MagicMock` for the test duration. Use `autospec=True` for strict signature checking.
- `mocker.patch('module.path')` — basic patching
- `mocker.patch.object(obj, 'attr')` — patch attribute on an object
- `mocker.patch.multiple('module', a=val, b=val)` — patch several names at once
- `mocker.patch.dict(os.environ, {'KEY': 'val'})` — override dictionaries

### mocker.spy(obj, method_name)
Observe real function calls without replacing the implementation. Returns a `SpyType` with extra attributes:
- `spy.spy_return` — last return value
- `spy.spy_return_list` — all return values (v3.13+)
- `spy.spy_exception` — last exception or `None`
- `mocker.stop(spy)` — selectively stop spying (v3.10+)

### mocker.stub(name)
Create a zero-config mock for callback verification. Accepts any arguments.
- `mocker.stub(name='on_complete')` — readable repr in test output
- `mocker.async_stub(name='...')` — async variant

### Scope-Aware Fixtures
Extend mock lifetime beyond a single test:
- `class_mocker` — all tests in a class
- `module_mocker` — all tests in a module
- `package_mocker` — all tests in a package
- `session_mocker` — all tests in a session

### Where to Patch
Patch the name in the module that *uses* it, not where it's *defined*. This is the #1 source of "my mock doesn't work" bugs.

### Type Annotations
Always annotate `mocker: MockerFixture` — gains IDE autocompletion and mypy checking at zero cost.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-introduction.md) | Introduction & Install | mocker fixture, automatic cleanup |
| [ch02](chapters/ch02-mocker-patch.md) | Mocker Fixture & Patch Methods | patch, patch.object, patch.multiple, patch.dict, stopall, scope fixtures |
| [ch03](chapters/ch03-spy.md) | Spy | mocker.spy, SpyType, spy_return, unspy |
| [ch04](chapters/ch04-stub.md) | Stub | mocker.stub, async_stub |
| [ch05](chapters/ch05-configuration.md) | Configuration | mock_use_standalone_module, mock_traceback_monkeypatch |
| [ch06](chapters/ch06-remarks.md) | Remarks & Type Annotations | MockerFixture, seal, why pytest-mock exists |

## Topic Index

- **autospec** → ch02
- **callback testing** → ch04
- **class_mocker** → ch02
- **configuration** → ch05
- **environment variables** → ch02
- **mocker.patch()** → ch02
- **mocker.patch.dict()** → ch02
- **mocker.patch.multiple()** → ch02
- **mocker.patch.object()** → ch02
- **mocker.spy()** → ch03
- **mocker.stub()** → ch04
- **mocker.stop()** → ch02, ch03
- **module_mocker** → ch02
- **seal** → ch06
- **session_mocker** → ch02
- **spy_return** → ch03
- **type annotations** → ch06
- **unittest.mock** → ch01, ch06
- **where to patch** → ch02

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the pytest-mock documentation only. For hands-on implementation in your codebase,
combine with project-specific test patterns. For topics beyond pytest-mock (e.g., advanced unittest.mock
features, pytest fixtures, test architecture), check related skills or ask the agent directly.
