# Test Design Review

## Farley Index: 5.7 / 10.0 (Fair)

### Property Breakdown

| Property | Static | LLM | Blended | Weight | Weighted | Key Evidence |
|---|---|---|---|---|---|---|
| Understandable | 4.6 | 5.5 | 4.9 | 1.50x | 7.42 | 7/21 methods use behavior-driven names; 4 cryptic or multi-behavior names |
| Maintainable | 4.5 | 4.5 | 4.5 | 1.50x | 6.72 | 5 mock anti-patterns (AP1-AP4) and private member access; 7 behavior-focused |
| Repeatable | 7.6 | 7.5 | 7.5 | 1.25x | 9.43 | 1 time.sleep; 2 shared mutable state dependencies; 18/21 pure |
| Atomic | 7.9 | 7.0 | 7.6 | 1.00x | 7.56 | shared_calc class-level variable creates inter-test dependency; 18/21 isolated |
| Necessary | 1.8 | 3.5 | 2.5 | 1.00x | 2.46 | 6 tautology theatre instances (29% of all tests add zero value) |
| Granular | 7.2 | 7.0 | 7.1 | 1.00x | 7.15 | 15/21 single-assertion; test_all_operations has 6 assertions |
| Fast | 8.1 | 8.5 | 8.3 | 0.75x | 6.21 | 1 time.sleep(0.5); 20/21 pure computation |
| First (TDD) | 3.6 | 5.0 | 4.2 | 1.00x | 4.16 | 7 behavior-driven "should" names; 7 tests show anti-TDD signals |

### Signal Summary

| Signal | Count | Affects | Severity |
|---|---|---|---|
| Behavior-driven names (should/when/returns) | 7 | U+, T+ | Positive |
| Behavior-focused assertions | 7 | M+ | Positive |
| Fresh instance per test | 18 | A+ | Positive |
| Single logical assertion | 15 | G+ | Positive |
| Pure computation (no I/O) | 20 | R+, F+ | Positive |
| Tautology theatre (all types) | 6 | N-, M- | Critical |
| Mock anti-patterns (AP1, AP2, AP3, AP4) | 4 | M-, N-, T- | High |
| Shared mutable state (class-level Calculator) | 2 | A-, R- | High |
| time.sleep(0.5) | 1 | R-, F- | High |
| Private member access (._history) | 1 | M- | Medium |
| Cryptic test name ("test_it_works") | 1 | U- | Medium |
| Multi-behavior name ("test_all_operations") | 1 | U-, G- | Medium |
| Mega-test (6 assertions) | 1 | G- | Medium |

### Tautology Theatre Analysis

Tests whose outcome is predetermined by their own setup, independent of production code. The defining test: "Would this test still pass if all production code were deleted?" If yes, it is tautology theatre.

#### Mock Tautologies

Configures a mock return value, then asserts that the mock returns it, with no production code in between.

| # | Test Method | Line | Mock Setup | Assertion |
|---|---|---|---|---|
| 1 | test_mock_returns_configured_value | test_calculator.py:93 | `mock_calc.add.return_value = 42` | `assertEqual(result, 42)` |

#### Mock-Only Tests

Every object in the test is a mock; no real class is instantiated or invoked.

| # | Test Method | Line | Evidence |
|---|---|---|---|
| 1 | test_mock_only_no_real_code | test_user_service.py:80 | `mock_service = MagicMock()` -- no UserService instantiated; asserts mock return value and mock call count |

#### Trivial Tautologies

Assertions that are always true regardless of any code: `assertTrue(true)`, `assertEqual(1, 1)`.

| # | Test Method | Line | Assertion |
|---|---|---|---|
| 1 | test_true_is_true | test_calculator.py:61 | `self.assertTrue(True)` |
| 2 | test_one_equals_one | test_calculator.py:64 | `self.assertEqual(1, 1)` |

#### Framework Tests

Tests that verify language or framework behavior, not application code.

| # | Test Method | Line | Assertion | What It Actually Tests |
|---|---|---|---|---|
| 1 | test_python_addition | test_calculator.py:101 | `assertEqual(2 + 2, 4)` | Python's `+` operator on integers |
| 2 | test_python_string_methods | test_calculator.py:104 | `assertEqual("hello".upper(), "HELLO")` | Python's `str.upper()` method |

#### Tautology Theatre Summary

**6** tautology theatre instances across **6** of **21** test methods: 1 mock tautology, 1 mock-only test, 2 trivial tautologies, 2 framework tests. These 6 tests (29% of the suite) provide zero verification of production behaviour and create false confidence in test coverage.

### Top 5 Worst Offenders

1. **test_user_service.py:80** `test_mock_only_no_real_code` -- ~2.5/10 -- Mock-only AP2; no production code exercised; every object is a MagicMock; would still pass if UserService were deleted
2. **test_calculator.py:93** `test_mock_returns_configured_value` -- ~3.0/10 -- Mock tautology AP1; configures `mock_calc.add.return_value = 42` then asserts `result == 42`; logically equivalent to `x = 42; assert x == 42`
3. **test_calculator.py:61** `test_true_is_true` -- ~3.0/10 -- Trivial tautology; `assertTrue(True)` can never fail; zero value; affects Necessary and First
4. **test_calculator.py:49** `test_all_operations` -- ~3.5/10 -- Mega-test with 6 assertions across add, subtract, multiply, divide, history count, and clear; not Granular; failure doesn't pinpoint the issue
5. **test_user_service.py:66** `test_register_calls_everything_in_order` -- ~4.0/10 -- Over-specified AP3; verifies exact call counts and ordering (`assert_called_before`); any behaviour-preserving refactoring breaks this test

### Recommendations

1. **Remove tautology theatre (N=2.5, weight 1.0x)**: Delete or replace the 6 tests that add zero value. The 2 trivial tautologies (`assertTrue(True)`, `assertEqual(1, 1)`), 2 framework tests (`2 + 2`, `"hello".upper()`), mock tautology, and mock-only test provide no production code verification. Replace with behaviour-focused tests that exercise real Calculator and UserService instances.

2. **Fix mock anti-patterns to improve Maintainable (M=4.5, weight 1.5x)**: Replace `test_register_calls_everything_in_order` with a test that verifies the *outcome* (user saved, email sent) rather than the *call sequence*. Replace `test_deactivate_sets_active_false` with a test that verifies deactivation through the public API rather than inspecting internal `call_args` state.

3. **Improve test naming for Understandable (U=4.9, weight 1.5x)**: Rename cryptic tests using the `test_should_[action]_[condition]` pattern. For example, rename `test_it_works` to `test_should_return_only_active_users`. This also improves First/TDD (T) since behaviour-driven names are a strong TDD signal.

4. **Eliminate shared mutable state (A=7.6, R=7.5)**: Replace the class-level `shared_calc = Calculator()` with per-test instances. `test_shared_state_history` currently depends on `test_shared_state_add` running first -- this violates both Atomic and Repeatable.

5. **Split the mega-test (G=7.1)**: Break `test_all_operations` into individual tests: `test_should_add`, `test_should_subtract`, `test_should_multiply`, `test_should_divide`, `test_should_track_history`, `test_should_clear_history`. Each tests one behaviour with one assertion.

### Methodology Notes

- Static/LLM blend: 60/40
- LLM model: claude-opus-4-6 (illustrative sample run; actual model depends on agent/runtime)
- Files analyzed: 2 (no sampling -- under 50-file threshold)
- Test methods analyzed: 21
- Language: Python 3
- Framework: unittest, unittest.mock (MagicMock)
- T (First/TDD) scoring note: Static evidence for TDD is indirect; LLM judgment weighted more heavily for this property based on naming patterns, AAA structure, and API-focused test design
- Note: This sample project contains deliberately mixed-quality tests (7 good, 14 bad) designed to demonstrate the Farley Score methodology

### Dimensions Not Measured

Predictive, Inspiring, Composable, Writable (from Beck's Test Desiderata -- require runtime or team context)

### Reference

Based on Dave Farley's Properties of Good Tests:
https://www.linkedin.com/pulse/tdd-properties-good-tests-dave-farley-iexge/

Scoring methodology by Andrea LaForgia:
https://github.com/andlaf-ak/claude-code-agents/tree/main/test-design-reviewer
