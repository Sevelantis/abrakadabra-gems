---
name: conftest-py
description: pytest and conftest.py knowledge base for tester-analyzer. Contains fixture patterns, hook reference, and the TEST_REPORT.md output format.
category: testing
tags: [pytest, conftest, fixtures, python]
---

## Role
Reference for pytest/conftest.py analysis and TEST_REPORT.md generation.

## Core Knowledge Areas
- conftest.py: shared fixture definitions, hook implementations, plugin registration
- Fixture scopes: `function` (default) | `class` | `module` | `session`
- Parametrize: `@pytest.mark.parametrize` for data-driven tests
- Hooks: `pytest_configure`, `pytest_runtest_setup`, `pytest_sessionfinish`

## conftest.py Patterns

### Fixture Definition
```python
@pytest.fixture(scope="module")
def test_example_client():
    client = Client()
    yield client
    client.close()
```

### Parametrize
```python
@pytest.mark.parametrize("input,expected", [
    ("test_example_a", True),
    ("test_example_b", False),
])
def test_example_behavior(input, expected):
    assert process(input) == expected
```

### Conftest Scope Hierarchy
- `tests/conftest.py` — session-wide fixtures
- `tests/<module>/conftest.py` — module-scoped overrides

## Output Format — TEST_REPORT.md

Compact, scannable. Adapt sections to what's actually found — skip empty sections.

```markdown
# TEST_REPORT — <scope>

## Coverage Gaps
| Area | Missing Test | Severity |
|---|---|---|
| `test_example_module.py` | edge case: empty input | HIGH |

## Logic Errors
- `test_example_login:42` — asserts `True` but function returns `None` on failure

## Quality Issues
- `test_example_flow` — tests 4 behaviours in one test; split by responsibility
- Missing teardown in `test_example_client` fixture (scope: session)

## Fixtures Inventory
| Name | Scope | File |
|---|---|---|
| `test_example_client` | module | `conftest.py:12` |

## Recommendations
1. <highest impact fix>
2. <second fix>
```

## Red Flags

- **God fixture**: one fixture doing setup + teardown + assertions — split it
- **No scope on shared fixtures**: defaults to `function`, recreated every test — likely unintended
- **Asserting implementation details**: tests break on refactor without behavior change
- **Missing `yield` in fixtures that open resources**: resource leak
