---
description: "Review: Evaluate test quality using Dave Farley's 8 Properties of Good Tests"
---

## ENTRY POINT: Determine User Intent

**CRITICAL: BEFORE doing anything else, determine how the user invoked this command.**

## AGENT COMPATIBILITY

Use these mappings to keep this command single-source across agents:

- **Invocation syntax**
  - Claude Code: `/msec:farley-score [target]`
  - Codex: plain-language request to run the `farley-score` workflow on `[target]`
- **Interactive menu tool**
  - Claude Code: `AskUserQuestion`
  - Codex: `request_user_input`
- **Command root**
  - Set `COMMAND_ROOT` to the root directory that contains `commands/`, `knowledge/`, `examples/`, and `lib/`
  - In this repository, `COMMAND_ROOT` is the repository root

### If invoked WITH a target path

If the user's message includes a file path, directory path, or specifies what to analyse (e.g., `/msec:farley-score tests/`, `/msec:farley-score src/test/java/`, `/msec:farley-score evaluate test quality for the whole project`), skip the welcome menu and go directly to the **4-Phase Workflow** section below.

### If invoked WITHOUT a target path

If the user simply typed `/msec:farley-score` with no additional arguments or context, present the welcome menu.

**Use the agent's interactive question tool (`AskUserQuestion` in Claude Code, `request_user_input` in Codex) to present the menu. Do NOT output the options as plain text.** This ensures the user gets a navigable interactive menu (arrow keys + Enter) rather than having to type a number.

Question: **"Welcome to Farley Score! What would you like to do?"**
Header: **"Farley Score"**

Options (use these exact labels and descriptions):

1. Label: **"What is this?"** -- Description: "Learn what the Farley Score tool does and how it works"
2. Label: **"Show me a demo"** -- Description: "See a complete sample report instantly from bundled example tests"
3. Label: **"Analyse my tests"** -- Description: "Run a full test quality analysis on your project"
4. Label: **"Coach me"** -- Description: "Switch to interactive coaching mode with Socratic questioning"

---

### Option 1: What is this?

Present this overview to the user:

> **Farley Score** measures test *quality*, not just test *coverage*.
>
> It evaluates your tests against Dave Farley's **8 Properties of Good Tests** -- Understandable, Maintainable, Repeatable, Atomic, Necessary, Granular, Fast, and First (TDD) -- and produces a single number: the **Farley Index** (0-10).
>
> The tool reads your test code (it never modifies anything), detects quality signals and anti-patterns, scores each property on a 0-10 scale, and produces a detailed report with evidence, worst offenders, and prioritised recommendations.
>
> Not all properties are equal. **Understandable** and **Maintainable** have the highest weight (1.5x) because tests that read like specifications and survive refactoring deliver the most long-term value. **Fast** has the lowest weight (0.75x) because a slow but well-designed test beats a fast but poorly designed one.
>
> A typical analysis takes 3-5 minutes depending on the size of your test suite.
>
> **Rating scale:** Exemplary (9-10), Excellent (7.5-8.9), Good (6-7.4), Fair (4.5-5.9), Poor (3-4.4), Critical (0-2.9)

After presenting the overview, ask the user what they'd like to do next (show the menu options again).

---

### Option 2: Show me a demo

Display the pre-packaged Farley Score report from the bundled sample project.

**To locate the report:**
1. Resolve command root: `COMMAND_ROOT=$(pwd)` (or set it explicitly to the directory containing this command package).
2. Read `$COMMAND_ROOT/examples/sample-project/farley-score-report.md`

Present the full report to the user, then walk them through the key sections:

1. **Farley Index**: "This sample suite scored 5.7 out of 10, rated 'Fair'. The sample project contains 21 test methods -- 7 well-designed and 14 deliberately flawed -- to demonstrate the kinds of issues the tool detects."

2. **Property Breakdown**: "The two highest-weighted properties -- Understandable (1.5x) and Maintainable (1.5x) -- scored 4.9 and 4.5 respectively. These are the biggest opportunities for improvement. Meanwhile, Fast scored 8.3 because almost all tests are pure computation."

3. **Tautology Theatre**: "The tool detected 6 tests that are 'tautology theatre' -- tests whose outcome is predetermined regardless of what the production code does. For example, `assertTrue(True)` will always pass even if you delete all your code. These tests provide zero value while inflating coverage numbers."

4. **Worst Offenders**: "The 5 lowest-scoring tests are listed with specific reasons. The worst is a mock-only test where every object is a MagicMock and no real class is ever instantiated."

5. **Recommendations**: "The tool prioritises recommendations by impact -- fixing the highest-weighted, lowest-scoring properties first."

After the walkthrough, ask: **"This report was generated from bundled sample tests. Want me to analyse YOUR tests now? Just tell me where they are (e.g., `tests/` or `src/test/`)."**

---

### Option 3: Analyse my tests

Ask the user: **"Where are your test files? For example: `tests/`, `src/test/java/`, or just say 'the whole project' and I'll find them."**

Wait for their response, then proceed to the **4-Phase Workflow** section with their specified target.

---

### Option 4: Coach me

Say: **"Switching to coaching mode! The Farley Score Coach teaches test quality through guided practice and Socratic questioning."**

Then switch to the Farley Score Coach workflow (`/msec:farley-score-coach` in Claude Code, or run `commands/farley-score-coach.md` in Codex).

---

### Option 5: Something else

If the user types something that doesn't match the options above, respond helpfully based on what they asked. If they want analysis, proceed to the **4-Phase Workflow**. If they want to learn, redirect to coaching mode.

---

# Farley Score -- Test Design Review

**Autonomous analysis: produces a Farley Index (0-10) with per-property breakdown, signal evidence, worst offenders, and prioritized recommendations.**

---

## CRITICAL: Read-Only Analysis

**This command ANALYZES test code. It NEVER modifies code.**

- Do NOT use any file-modifying actions or tools
- Do NOT create, delete, or rename files
- Output is returned as structured text in the conversation
- If the user requests code changes, state that recommendations are in the report and suggest using `/msec:tdd` for implementation

---

## What This Does

When this command runs, the agent will:

1. **Discover** test files, detect language and frameworks
2. **Collect** static signals per property per test method
3. **Detect** Tautology Theatre (4 types) and Mock Anti-Patterns (AP1-AP4)
4. **Score** each property via static analysis + LLM semantic assessment
5. **Compute** the Farley Index using the deterministic Python calculator
6. **Report** with property breakdown, evidence, worst offenders, and recommendations

This is **autonomous review mode** - the agent does the analysis and produces the report.

**Want to LEARN test quality instead?** Use the Farley Score Coach workflow (`/msec:farley-score-coach` in Claude Code; `commands/farley-score-coach.md` in Codex) for interactive teaching.

---

## Examples

```
/msec:farley-score tests/
/msec:farley-score src/test/java/
/msec:farley-score evaluate test quality for the whole project
/msec:farley-score review test design in tests/unit/
```

---

## The 8 Properties of Good Tests

| Code | Property | Weight | What It Measures |
|---|---|---|---|
| U | Understandable | 1.50x | Tests read like specifications; behavior-driven naming; clear organization |
| M | Maintainable | 1.50x | Tests verify behavior not implementation; resilient to refactoring |
| R | Repeatable | 1.25x | Deterministic results; no external dependencies (time, filesystem, network) |
| A | Atomic | 1.00x | Isolated; no shared mutable state; parallelizable |
| N | Necessary | 1.00x | Each test adds unique value; no redundancy or trivial assertions |
| G | Granular | 1.00x | Single outcome per test; failures pinpoint exact issues |
| F | Fast | 0.75x | Pure computation; no I/O or delays |
| T | First (TDD) | 1.00x | Evidence of test-first approach; tests drive design |

---

## Attribution

**Methodology**: Andrea LaForgia's [test-design-reviewer](https://github.com/andlaf-ak/claude-code-agents/tree/main/test-design-reviewer) -- scoring system, signal detection patterns, tautology theatre analysis. Used with permission.

**Framework**: Dave Farley's [Properties of Good Tests](https://www.linkedin.com/pulse/tdd-properties-good-tests-dave-farley-iexge/)

---

## Knowledge Base (Reference During Analysis)

**These files live under `COMMAND_ROOT`.** In this repository, `COMMAND_ROOT` is the project root.

1. Resolve command root: `COMMAND_ROOT=$(pwd)` (or set explicitly when needed)
2. Knowledge files are at `$COMMAND_ROOT/knowledge/farley/`.

**Load these documents during the phases indicated:**

- **Phase 2 (Signal Collection):** Read `$COMMAND_ROOT/knowledge/farley/signal-detection-patterns.md` for language-specific detection heuristics
- **Phase 3 (Scoring):** Read `$COMMAND_ROOT/knowledge/farley/farley-properties-and-scoring.md` for rubrics and formula

**Note:** Only read these files when you reach the relevant phase. Don't load upfront.

---

## Core Principles

These 7 principles define the methodology:

1. **Read-only analysis**: Analyze but never modify code. No file-modifying tools/actions. Output is returned as structured text.
2. **Two-phase scoring**: Score each property through static signal detection first (deterministic), then LLM semantic assessment (controlled). Blend at 60/40 static/LLM per property.
3. **Per-test-method granularity**: Collect signals at individual test method level. Aggregate to file level (mean for positives, P90 for negatives). Aggregate to suite level via LOC-weighted mean.
4. **Evidence-anchored scoring**: Every property score cites specific code locations and signal counts. A score without evidence is not a score -- it is a guess.
5. **Conservative base score**: When no signals are detected for a property, default to 5.0 (Fair). No-signal means unknown quality, not good quality.
6. **Proportional effort**: Scale analysis depth to codebase size. Under 50 test files: analyze all. Over 50: SHA-256 deterministic selection (30%) plus all files exceeding 100 test methods.
7. **Language-aware detection**: Identify the test framework from imports and annotations before signal scanning. Use language-specific patterns from the signal-detection-patterns knowledge file. Do not apply Java patterns to Python or vice versa.

---

## 4-Phase Workflow

### Phase 1: Discovery (2-3 turns)

- Identify test file locations using naming conventions and directory patterns (`test/`, `tests/`, `*_test.*`, `*Test.*`, `*.spec.*`, `*.test.*`)
- Detect primary language, test framework, and mocking framework from imports and annotations
- Count total test files, test methods, and LOC
- If over 50 test files, activate deterministic sampling (SHA-256 hash of filename, select 30% plus all files exceeding 100 test methods)
- **Gate**: language identified, test framework identified, mocking framework identified (if present), test file inventory complete

### Phase 2: Signal Collection (6-10 turns)

- **Read** `$COMMAND_ROOT/knowledge/farley/signal-detection-patterns.md` for language-specific patterns (resolve COMMAND_ROOT as described in Knowledge Base section)
- For each test file (or sampled subset):
  1. Identify test method boundaries (framework-specific markers)
  2. Scan for negative signals per property: sleep, reflection, shared state, ordering, I/O, magic numbers, cryptic names, trivial assertions, mega-tests
  3. Scan for **Tautology Theatre** -- tests whose outcome is predetermined, independent of production code:
     - **Mock tautology**: mock return value configured then asserted on same mock with no production code in between (affects N, M)
     - **Mock-only test**: all objects are mocks, no real class instantiated (affects N, M, T)
     - **Trivial tautology**: `assertTrue(true)`, `assertEquals(1, 1)`, `assertNotNull(new Object())` (affects N)
     - **Framework test**: verifies language/framework behavior, not application code (affects N)
  4. Scan for Mock Anti-Patterns (if mocking framework detected):
     - **Over-specified interactions (AP3)**: verify with exact counts, call ordering, verifyNoMoreInteractions (affects M)
     - **Testing internal details (AP4)**: ArgumentCaptor deep inspection, verify(never()) mirroring branches (affects M)
  5. Scan for positive signals per property: behavior names, nested organization, parameterized tests, AAA structure, builders, parallel markers
  6. Count assertions per test method
  7. Record signal locations (`file:line`) for evidence
- Attribute multi-property signals to all relevant properties (e.g., Thread.sleep affects both R and F)
- **Gate**: signal inventory complete for all 8 properties; each signal has a file:line reference

### Phase 3: Scoring (3-5 turns)

- **Read** `$COMMAND_ROOT/knowledge/farley/farley-properties-and-scoring.md` for rubrics and formula

#### Static Scoring

For each property, compute a 0-10 score based on signal densities using the rubric from the knowledge file.

#### LLM Scoring

For each property, assess the test code holistically against the rubric, providing a 0-10 score with brief justification. Focus on semantic aspects that static analysis misses: naming quality, assertion appropriateness, design influence, tautology theatre.

#### CLI Calculator Integration

**CRITICAL: Use the Python CLI calculator for ALL math. Do NOT compute scores manually.**

**Locating the calculator:** The CLI calculator lives under `COMMAND_ROOT/lib/`.

```bash
# Resolve the command root (if not already set)
COMMAND_ROOT=$(pwd)
CLI="$COMMAND_ROOT/lib/cli_calculator.py"

# Normalize a single property
python3 "$CLI" normalize-property '{"prop":"U","neg_count":2,"pos_count":8,"total_methods":20}'

# Blend static and LLM scores
python3 "$CLI" blend-scores '{"static_score":7.5,"llm_score":8.0}'

# Full end-to-end pipeline (recommended)
python3 "$CLI" full-pipeline '{"properties":{"U":{"neg_count":2,"pos_count":15,"total_methods":20},"M":{"neg_count":3,"pos_count":10,"total_methods":20},"R":{"neg_count":1,"pos_count":18,"total_methods":20},"A":{"neg_count":0,"pos_count":12,"total_methods":20},"N":{"neg_count":4,"pos_count":8,"total_methods":20},"G":{"neg_count":1,"pos_count":16,"total_methods":20},"F":{"neg_count":0,"pos_count":18,"total_methods":20},"T":{"neg_count":2,"pos_count":10,"total_methods":20}},"llm_scores":{"U":8.0,"M":7.5,"R":9.0,"A":8.5,"N":6.5,"G":8.0,"F":9.0,"T":7.0}}'

# Get rating from index
python3 "$CLI" get-rating '{"farley_index":7.8}'
```

**Blend formula**: `final_property_score = 0.60 * static_score + 0.40 * llm_score`

**Farley Index formula**: `(U*1.5 + M*1.5 + R*1.25 + A*1.0 + N*1.0 + G*1.0 + F*0.75 + T*1.0) / 9.0`

- **Gate**: all 8 properties scored with both static and LLM components; Farley Index computed; rating assigned

### Phase 4: Reporting (2-3 turns)

- Identify top 5 worst-offending test methods (lowest per-method scores)
- Compile the Tautology Theatre Analysis section with all 4 subsections (always present -- use "None detected." when empty)
- Generate 3-5 prioritized recommendations targeting highest-weighted properties with lowest scores
- Produce the structured report (see Report Format below)
- Include methodology notes: files analyzed, sampling status, model identifier
- **Gate**: report contains all required sections including Tautology Theatre Analysis with all four subsections

---

## Report Format

```
# Test Design Review

## Farley Index: X.X / 10.0 (Rating)

### Property Breakdown

| Property | Static | LLM | Blended | Weight | Weighted | Key Evidence |
|---|---|---|---|---|---|---|
| Understandable | X.X | X.X | X.X | 1.50x | X.XX | ... |
| Maintainable | X.X | X.X | X.X | 1.50x | X.XX | ... |
| Repeatable | X.X | X.X | X.X | 1.25x | X.XX | ... |
| Atomic | X.X | X.X | X.X | 1.00x | X.XX | ... |
| Necessary | X.X | X.X | X.X | 1.00x | X.XX | ... |
| Granular | X.X | X.X | X.X | 1.00x | X.XX | ... |
| Fast | X.X | X.X | X.X | 0.75x | X.XX | ... |
| First (TDD) | X.X | X.X | X.X | 1.00x | X.XX | ... |

### Signal Summary

| Signal | Count | Affects | Severity |
|---|---|---|---|
| {signal_name} | {count} | {properties} | {High/Medium/Low} |

### Tautology Theatre Analysis

Tests whose outcome is predetermined by their own setup, independent of production code. The defining test: "Would this test still pass if all production code were deleted?" If yes, it is tautology theatre.

#### Mock Tautologies

Configures a mock return value, then asserts that the mock returns it, with no production code in between.

| # | Test Method | Line | Mock Setup | Assertion |
|---|---|---|---|---|
| 1 | {method_name} | {line} | {mock setup} | {assertion} |

> If none detected: "None detected."

#### Mock-Only Tests

Every object in the test is a mock; no real class is instantiated or invoked.

| # | Test Method | Line | Evidence |
|---|---|---|---|
| 1 | {method_name} | {line} | {evidence} |

> If none detected: "None detected."

#### Trivial Tautologies

Assertions that are always true regardless of any code: `assertTrue(true)`, `assertEquals(1, 1)`.

| # | Test Method | Line | Assertion |
|---|---|---|---|
| 1 | {method_name} | {line} | {assertion} |

> If none detected: "None detected."

#### Framework Tests

Tests that verify language or framework behavior, not application code.

| # | Test Method | Line | Assertion | What It Actually Tests |
|---|---|---|---|---|
| 1 | {method_name} | {line} | {assertion} | {explanation} |

> If none detected: "None detected."

#### Tautology Theatre Summary

**{total}** tautology theatre instances across **{affected_methods}** of **{total_test_methods}** test methods: {count} mock tautologies, {count} mock-only tests, {count} trivial tautologies, {count} framework tests.

### Top 5 Worst Offenders

1. {file}:{method} -- Farley {score}/10 -- {key issues}
2. ...

### Recommendations

1. {highest-impact improvement targeting weakest high-weight property}
2. ...
3. ...

### Methodology Notes

- Static/LLM blend: 60/40
- LLM model: {model_id}
- Files analyzed: {count} ({sampling_note})
- Test methods analyzed: {count}
- Language: {language}
- Framework: {framework}

### Dimensions Not Measured

Predictive, Inspiring, Composable, Writable (from Beck's Test Desiderata -- require runtime or team context)

### Reference

Based on Dave Farley's Properties of Good Tests:
https://www.linkedin.com/pulse/tdd-properties-good-tests-dave-farley-iexge/

Scoring methodology by Andrea LaForgia:
https://github.com/andlaf-ak/claude-code-agents/tree/main/test-design-reviewer
```

---

## Rating Scale

| Farley Index | Rating | Interpretation |
|---|---|---|
| 9.0 - 10.0 | Exemplary | Model for the industry; tests serve as living documentation |
| 7.5 - 8.9 | Excellent | High quality with minor improvement opportunities |
| 6.0 - 7.4 | Good | Solid foundation with clear areas for improvement |
| 4.5 - 5.9 | Fair | Functional but needs significant attention to test design |
| 3.0 - 4.4 | Poor | Tests provide limited value; major refactoring needed |
| 0.0 - 2.9 | Critical | Tests may be harmful; consider rewriting from scratch |

---

## Supported Languages and Frameworks

| Language | Test Framework | Mocking Framework |
|---|---|---|
| Java | JUnit 5, JUnit 4 | Mockito |
| Python | pytest, unittest | unittest.mock, pytest-mock |
| JavaScript/TypeScript | Jest, Vitest | Jest mocks, Sinon |
| Go | testing | testify/mock, gomock |
| C# | NUnit, xUnit | Moq, NSubstitute |

---

## Critical Rules

1. **Never modify code.** This command analyzes and reports only. No file-modifying tools/actions.
2. **Record `file:line` references** for every signal detected. Evidence without location is unverifiable.
3. **Score every property on both static and LLM dimensions** before blending. Skipping a dimension produces an unbalanced score.
4. **Use the CLI calculator for ALL math.** Do not compute Farley Index, sigmoid normalization, or blending manually. Run `python3 $COMMAND_ROOT/lib/cli_calculator.py`. See "CLI Calculator Integration" in Phase 3 for path resolution steps.
5. **Apply the Farley Index formula exactly**: `(U*1.5 + M*1.5 + R*1.25 + A*1.0 + N*1.0 + G*1.0 + F*0.75 + T*1.0) / 9.0`. The divisor is 9.0 (sum of weights), not 8 (number of properties).
6. **When scoring T (First/TDD)**, acknowledge that static evidence is indirect. Weight LLM judgment more heavily for this property. Note this in the methodology section.
7. **Conservative base: 5.0** when no signals are detected for a property. No-signal means unknown quality, not good quality.

---

## Configuration

### report_output (default: "inline")

- `"inline"` -- Report displayed in conversation
- `"file"` -- Report written to `farley-score-report.md` in the project root

### sampling_threshold (default: 50)

Number of test files before activating SHA-256 deterministic sampling (30% selection).

---

## When to Use Each MSEC Command

| Situation | Command | Why |
|---|---|---|
| Automated quality review | `/msec:farley-score` | Full autonomous analysis with Farley Index |
| Want to learn test quality | `/msec:farley-score-coach` | Interactive Socratic coaching |
| Writing new code with TDD | `/msec:tdd` | TDD productivity mode |
| Learning TDD methodology | `/msec:tdd-coach` | TDD coaching mode |
