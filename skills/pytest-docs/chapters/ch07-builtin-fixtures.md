# Chapter 7: Builtin Fixtures

## Core Idea
pytest ships with built-in fixtures for common test needs: temporary paths (`tmp_path`), environment patching (`monkeypatch`), output capture (`capsys`, `caplog`), and warning control (`recwarn`).

## Frameworks Introduced
- **tmp_path / tmp_path_factory**: Per-test temporary directories — `tmp_path` is function-scoped, `tmp_path_factory` for broader scopes
- **monkeypatch**: Safe attribute/item/envvar patching with automatic undo — `monkeypatch.setattr`, `monkeypatch.setenv`, `monkeypatch.delattr`
- **capsys / capsys_binary**: Capture stdout/stderr — `capsys.readouterr()` returns namedtuple
- **capfd / capfd_binary**: Lower-level fd capture (catches C-level output)
- **caplog**: Capture logging output — `caplog.text`, `caplog.records`, `caplog.handler`
- **recwarn**: Capture warnings — iterate `recwarn` or use `pytest.warns`

## Key Concepts
- **tmp_path**: `pathlib.Path` pointing to unique temp dir per test
- **tmp_path_factory**: Session-scoped; use `tmp_path_factory.mktemp("name")` for temp dirs
- **monkeypatch.setattr(target, name, value)**: Patch object attributes; auto-restores after test
- **monkeypatch.setenv(key, value)**: Set environment variables; auto-removed after test
- **monkeypatch.delenv(key)**: Remove env var; auto-restored after test
- **capsys.readouterr()**: Returns `RecordCapture(out="", err="", outbytes=b"", errbytes=b"")`
- **caplog.text**: All captured log output as string
- **caplog.set_level(logging.DEBUG)**: Set minimum log level for capture

## Mental Models
- `tmp_path` = pytest's answer to "I need files for this test" — no manual cleanup needed
- `monkeypatch` = safe version of `unittest.mock.patch` — always undoes changes
- `capsys` captures print() output; `caplog` captures logging module output
- Use `tmp_path_factory` when you need temp dirs in session/module-scoped fixtures

## Anti-patterns
- **Manual temp dir cleanup**: Always use `tmp_path` — it handles cleanup automatically
- **Using os.environ directly**: Use `monkeypatch.setenv` — it restores after test
- **Checking capsys in wrong scope**: `capsys` is function-scoped; use `capsys.readouterr()` in the test

## Code Examples
```python
# Temporary directory
def test_write_file(tmp_path):
    file = tmp_path / "data.txt"
    file.write_text("hello")
    assert file.read_text() == "hello"

# Monkeypatch environment variable
def test_env_var(monkeypatch):
    monkeypatch.setenv("API_KEY", "test123")
    assert os.environ["API_KEY"] == "test123"
    # Auto-removed after test

# Monkeypatch object attribute
def test_patch_function(monkeypatch):
    monkeypatch.setattr(module, "get_data", lambda: "mocked")
    assert module.get_data() == "mocked"

# Capture stdout
def test_output(capsys):
    print("hello")
    captured = capsys.readouterr()
    assert captured.out == "hello\n"

# Capture logs
def test_logging(caplog):
    with caplog.at_level(logging.INFO):
        logger.info("test message")
    assert "test message" in caplog.text

# Capture warnings
def test_warning(recwarn):
    warnings.warn("deprecated", DeprecationWarning)
    assert len(recwarn) == 1
    assert "deprecated" in str(recwarn[0].message)
```

## Key Takeaways
1. `tmp_path` provides per-test temp directories — auto-cleaned
2. `monkeypatch.setattr/delattr/setenv/delenv` — auto-undone after test
3. `capsys.readouterr()` captures stdout/stderr — use `-s` to disable
4. `caplog` captures logging output — use `caplog.set_level()` to control
5. `recwarn` captures warnings; `pytest.warns` is context-manager alternative
6. Use `tmp_path_factory` for session/module-scoped temp directories

## Connects To
- **Ch 3**: Fixtures — all built-ins are fixtures with specific scopes
- **Ch 8**: Logging/Capture — deeper dive into capsys and caplog
- **Ch 1**: Getting Started — `tmp_path` introduced in first examples
