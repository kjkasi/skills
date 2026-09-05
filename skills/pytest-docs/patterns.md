# Patterns & Techniques

## Test Naming Convention
**When to use**: Always — establishes clear expectations for test purpose
**How**: Name files `test_<module>.py`, functions `test_<what>_<scenario>_<expected>`, classes `Test<Feature>`
**Trade-offs**: More verbose than generic names, but self-documenting and searchable

## Parametrize Over Loops
**When to use**: Testing same logic with multiple input sets
**How**: Replace `for` loops in tests with `@pytest.mark.parametrize("arg", [values])`
**Trade-offs**: Each case gets individual ID/reporting vs. single test with loop

## Factory as Fixture
**When to use**: Need multiple instances of same type in one test
**How**: Fixture returns a callable: `yield lambda name: create_thing(name)`
**Trade-offs**: More setup code, but reusable and cleanup-aware

## Yield Fixture Teardown
**When to use**: Any fixture that creates resources needing cleanup
**How**: Use `yield` instead of `return`; teardown code after yield runs in reverse order
**Trade-offs**: Clean separation of setup/teardown vs. slightly more complex than return

## Safe Teardown Structure
**When to use**: Complex multi-resource setup that might partially fail
**How**: Split into single-purpose fixtures, each with its own yield teardown
**Trade-offs**: More fixtures, but each failure leaves minimal residue

## Marker-Based Test Selection
**When to use**: Running subsets of tests (slow, network, environment-specific)
**How**: Register markers in config, apply `@pytest.mark.name`, select with `-m "name"`
**Trade-offs**: Requires upfront marker design, but enables powerful selection expressions

## conftest.py as Fixture Hub
**When to use**: Shared fixtures across multiple test files
**How**: Place shared fixtures in conftest.py at appropriate directory level
**Trade-offs**: Auto-discovered but can be surprising; keep hierarchy shallow

## Indirect Parametrize
**When to use**: Parametrizing fixture behavior, not test arguments
**How**: `@pytest.mark.parametrize("fixture_name", [values], indirect=True)`
**Trade-offs**: Cleaner than fixture params when same fixture needs different configs

## pytest.param with Marks
**When to use**: Skip/xfail individual parametrize values
**How**: `pytest.param(value, marks=pytest.mark.skip(reason="..."))`
**Trade-offs**: More explicit than fixture-level conditionals

## Custom Assertion Explanations
**When to use**: Complex types where default comparison output is unclear
**How**: Implement `pytest_assertrepr_compare` hook in conftest.py
**Trade-offs**: One-time setup cost, but dramatically better failure reports

## Monkeypatch Pattern
**When to use**: Testing code that reads env vars, global state, or external dependencies
**How**: `monkeypatch.setattr(target, "attr", mock_value)` — auto-undone after test
**Trade-offs**: Safer than manual mock/restore, but limited to attribute/envvar patching

## Exception Group Testing
**When to use**: Code using `ExceptionGroup` / `except*` syntax
**How**: `pytest.RaisesGroup` with `match`, `check`, `flatten_subgroups` parameters
**Trade-offs**: More precise than `group_contains()`, prevents missing unexpected exceptions
