---
name: pytest-docs
description: "Knowledge base from \"pytest Documentation\" by pytest-dev team. Use when writing tests with pytest, applying fixtures/parametrize/markers, configuring test suites, debugging failures, or referencing pytest APIs and patterns."
---

<!-- argument-hint: [topic, fixture name, or chapter number] -->

# pytest Documentation
**Author**: pytest-dev team | **Pages**: ~2000 | **Chapters**: 15 | **Generated**: 2026-09-03

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `fixtures`, `parametrize`, `markers`, or another indexed topic
- **With chapter** — ask for `ch03`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

### Test Discovery & Writing
- **Discovery**: pytest collects `test_*.py` and `*_test.py`, finds `test_` prefixed functions/classes automatically
- **Assertion**: Use plain `assert` — pytest rewrites it to show intermediate values on failure
- **pytest.raises**: `with pytest.raises(Exc, match=r"regex"): code()` — context manager for exception testing
- **pytest.approx**: `assert a == pytest.approx(b)` — float comparison with tolerance, works with scalars/arrays/NaNs

### Fixtures (The Core Abstraction)
- **Request pattern**: Test declares dependencies as function arguments; pytest injects matching fixtures
- **Scope**: `function` (default), `class`, `module`, `package`, `session` — controls lifetime and sharing
- **Yield fixtures**: `yield` in fixture = setup before, teardown after (runs in reverse order)
- **Autouse**: `@pytest.fixture(autouse=True)` — runs without explicit request
- **Factory pattern**: Return a callable from a fixture to generate multiple instances
- **Composition**: Fixtures can request other fixtures — pytest resolves dependency DAG automatically
- **Scope rule**: Broader scope can't depend on narrower scope (session can't use module fixture)

### Parametrize
- **@pytest.mark.parametrize("arg", [values])**: One test per value — each gets individual ID/reporting
- **Cross-product**: Multiple decorators = Cartesian product of all combinations
- **indirect=True**: Route values through a fixture instead of directly to test parameters
- **pytest.param**: `pytest.param(val, marks=pytest.mark.skip)` — apply marks per-value

### Markers
- **skip/skipif**: `@pytest.mark.skip(reason=...)` / `@pytest.mark.skipif(cond, reason=...)`
- **xfail**: `@pytest.mark.xfail(reason=...)` — expected failure, test still runs
- **xfail(raises=Exc)**: xfail only on specific exception type
- **strict=True**: Fails if test unexpectedly passes (tracks bug fixes)
- **Selection**: `-m "marker"` with boolean: `-m "slow and not network"`
- **Register custom markers** in pyproject.toml to avoid warnings

### Builtin Fixtures
- **tmp_path**: `pathlib.Path` per-test temp dir — auto-cleaned
- **monkeypatch**: `setattr`/`setenv`/`delattr` — auto-undone after test
- **capsys**: `capsys.readouterr()` captures stdout/stderr
- **caplog**: `caplog.text` / `caplog.records` for logging capture
- **recwarn**: Warning capture; `pytest.warns` is context-manager alternative

### Configuration
- **pyproject.toml**: `[tool.pytest.ini_options]` — `testpaths`, `addopts`, `markers`, `filterwarnings`
- **addopts**: Default CLI flags: `addopts = "-v --tb=short --strict-markers -x"`
- **conftest.py**: Project-local plugin — fixtures + hooks auto-discovered
- **Exit codes**: 0=pass, 1=fail, 2=interrupted, 3=internal error, 4=usage error, 5=no tests

### CLI Essentials
- `-x` stop on first failure | `--lf` rerun last failures | `-s` no capture (see prints)
- `--tb=short` concise tracebacks | `-v` verbose | `--co` collect only
- `-k "expr"` name filtering | `-m "marker"` marker filtering
- `--junitxml=report.xml` CI reports | `--pdb` debugger on failure

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-getting-started.md) | Getting Started | discovery, assert, pytest.raises, pytest.approx |
| [ch02](chapters/ch02-assertions.md) | Assertions | introspection, RaisesGroup, RaisesExc, approx |
| [ch03](chapters/ch03-fixtures.md) | Fixtures | request, scope, yield, autouse, factory, composition |
| [ch04](chapters/ch04-parametrize.md) | Parametrize | parametrize, cross-product, indirect, pytest.param |
| [ch05](chapters/ch05-markers.md) | Markers | skip, skipif, xfail, selection, registration |
| [ch06](chapters/ch06-usage.md) | Usage & CLI | -k, -m, -x, --lf, --tb, --pdb, --co |
| [ch07](chapters/ch07-builtin-fixtures.md) | Builtin Fixtures | tmp_path, monkeypatch, capsys, caplog, recwarn |
| [ch08](chapters/ch08-logging-capture.md) | Logging & Capture | caplog, capsys, --capture, --log-level |
| [ch09](chapters/ch09-skipping.md) | Skipping & Xfail | skip, skipif, xfail, strict, --runxfail |
| [ch10](chapters/ch10-plugins.md) | Plugins | hooks, conftest.py, -p, entry points |
| [ch11](chapters/ch11-configuration.md) | Configuration | pyproject.toml, addopts, markers, testpaths |
| [ch12](chapters/ch12-test-output.md) | Test Output | --tb, exit codes, --junitxml, report-log |
| [ch13](chapters/ch13-good-practices.md) | Good Practices | src layout, naming, independence, conftest |
| [ch14](chapters/ch14-unittest-compat.md) | Unittest Compat | TestCase, setup_method, xunit-style |
| [ch15](chapters/ch15-examples.md) | Working Examples | markers, parametrize, collection hooks |

## Topic Index

- **fixtures** → ch03[, ch07, ch13]
- **parametrize** → ch04[, ch03, ch15]
- **markers** → ch05[, ch04, ch15]
- **assertions** → ch02[, ch01]
- **monkeypatch** → ch07
- **tmp_path** → ch07
- **capsys** → ch07, ch08
- **caplog** → ch08
- **recwarn** → ch07
- **skip / skipif** → ch09
- **xfail** → ch09
- **plugins** → ch10
- **conftest.py** → ch10, ch11, ch13
- **configuration** → ch11
- **exit codes** → ch12
- **test discovery** → ch01
- **src layout** → ch13
- **unittest** → ch14

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the pytest documentation content only. For hands-on implementation
in your codebase, combine with project-specific tools. For topics beyond this documentation
(plugins like pytest-xdist, pytest-cov, pytest-django), check related skills or ask
the agent directly.
