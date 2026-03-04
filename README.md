# Farley Score Plugin for Claude Code

Test quality assessment using Dave Farley's 8 Properties of Good Tests.

Analyses your test suite and produces a **Farley Index** (0-10) with per-property breakdown, signal evidence, worst offenders, and prioritised recommendations.

## Installation

```
/plugin marketplace add mse-online/farley_score_plugin
/plugin install msec@farley-score-plugin
```

Or from the shell:

```bash
claude plugin marketplace add mse-online/farley_score_plugin
claude plugin install msec@farley-score-plugin
```

## Commands

### `/msec:farley-score [target]`

**Autonomous test quality review.** Analyses your tests across 8 weighted properties and produces a detailed report with a Farley Index score.

```
/msec:farley-score              # Welcome menu (first-time users)
/msec:farley-score tests/       # Direct analysis (experienced users)
```

### `/msec:farley-score-coach [topic]`

**Interactive test quality coaching.** Teaches you to identify and fix test quality issues through Socratic questioning, guided practice, and score predictions.

```
/msec:farley-score-coach                    # Welcome menu
/msec:farley-score-coach practice           # Practice with built-in examples
/msec:farley-score-coach mode:intermediate  # Set difficulty level
```

## The 8 Properties of Good Tests

| Code | Property | Weight | What It Measures |
|---|---|---|---|
| U | Understandable | 1.50x | Tests read like specifications |
| M | Maintainable | 1.50x | Tests verify behavior, not implementation |
| R | Repeatable | 1.25x | Same result every time, anywhere |
| A | Atomic | 1.00x | Isolated, no shared state |
| N | Necessary | 1.00x | Every test adds unique value |
| G | Granular | 1.00x | Single outcome per test |
| F | Fast | 0.75x | Pure computation, no I/O |
| T | First (TDD) | 1.00x | Written before implementation |

## Rating Scale

| Farley Index | Rating |
|---|---|
| 9.0 - 10.0 | Exemplary |
| 7.5 - 8.9 | Excellent |
| 6.0 - 7.4 | Good |
| 4.5 - 5.9 | Fair |
| 3.0 - 4.4 | Poor |
| 0.0 - 2.9 | Critical |

## Supported Languages

Java (JUnit), Python (pytest, unittest), JavaScript/TypeScript (Jest, Vitest), Go (testing), C# (NUnit, xUnit).

## Attribution

- **Dave Farley** -- [8 Properties of Good Tests](https://www.linkedin.com/pulse/tdd-properties-good-tests-dave-farley-iexge/)
- **Andrea Laforgia** -- [Farley Score methodology](https://github.com/andlaf-ak/claude-code-agents/tree/main/test-design-reviewer), scoring system, and signal detection patterns. Used with permission.
- **Bernard McCarty** -- Plugin implementation

## License

MIT
