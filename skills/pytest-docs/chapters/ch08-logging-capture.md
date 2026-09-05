# Chapter 8: Logging and Capture

## Core Idea
pytest captures stdout/stderr by default (visible on failure), supports the Python logging module via `caplog`, and provides fine-grained control over what output is captured and displayed.

## Frameworks Introduced
- **Output Capture System**: pytest captures stdout/stderr during tests; shows on failure, hides on pass (configurable)
- **caplog Fixture**: Captures logging module output with level filtering and record inspection
- **--capture=MODE**: `fd` (default), `sys`, `no` — controls capture mechanism
- **logging.ini / pyproject.toml logging config**: Configure logging levels and handlers for test runs

## Key Concepts
- **--capture=fd**: Captures at file descriptor level (default) — catches C-level output
- **--capture=sys**: Captures at sys.stdout/stderr level
- **--capture=no / -s**: No capture — output visible immediately
- **caplog.text**: All captured log output as string
- **caplog.records**: List of `LogRecord` objects for inspection
- **caplog.set_level(level)**: Set minimum capture level
- **caplog.at_level(level)**: Context manager for temporary level change
- **--log-format / --log-date-format**: Customize log output format
- **--log-level**: Set logging level for test run

## Mental Models
- Capture is "opt-out" — pytest hides output on pass, shows on failure
- `caplog` is for the logging module; `capsys` is for print/stdout
- Use `caplog.set_level(logging.DEBUG)` to see debug-level logs in tests
- `--capture=no` is the same as `-s` — both disable capture entirely

## Anti-patterns
- **Using print for debugging**: Use logging module + caplog for structured output
- **Forgetting to check capsys/caplog**: Output is only visible on failure unless you assert
- **Over-capturing**: Too many logs slow tests; use `caplog.set_level()` to filter

## Code Examples
```python
# Capture and inspect logs
def test_logging(caplog):
    caplog.set_level(logging.INFO)
    logger.info("Starting process")
    logger.warning("Low memory")
    assert "Starting process" in caplog.text
    assert len(caplog.records) == 2

# Capture specific logger
def test_specific_logger(caplog):
    with caplog.at_level(logging.DEBUG, logger="myapp.db"):
        myapp.db.connect()
    assert "Connected" in caplog.text

# Check log records
def test_log_records(caplog):
    logger.error("Something failed")
    assert caplog.records[0].levelname == "ERROR"
    assert caplog.records[0].message == "Something failed"
```

```bash
# CLI options
pytest -s                           # No capture (same as --capture=no)
pytest --capture=sys                # Capture at sys level
pytest --log-level=DEBUG            # Show all log levels
pytest --log-format="%(asctime)s %(levelname)s %(message)s"
pytest --logging-level=INFO         # Set logging level for root logger
```

## Key Takeaways
1. Output capture is on by default — visible on failure, hidden on pass
2. `-s` / `--capture=no` disables capture — essential for debugging
3. `caplog` captures logging module output — use `caplog.set_level()` for filtering
4. `caplog.records` gives access to individual LogRecord objects
5. `--log-level` and `--log-format` control logging display in CLI

## Connects To
- **Ch 6**: Usage — `-s` flag for debugging
- **Ch 7**: Builtin Fixtures — capsys, caplog, recwarn are built-in fixtures
- **Ch 12**: Test Output — failure reporting and output formatting
