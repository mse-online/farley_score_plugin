---
name: farley-score
description: Run autonomous Farley Score test quality analysis from this repository's command spec. Use when asked to evaluate test quality, calculate a Farley Index, review tests against Dave Farley's 8 properties, or produce a structured test design report with evidence and prioritized recommendations.
---

# Farley Score

1. Set `COMMAND_ROOT` to the repository root that contains `commands/`, `knowledge/`, `examples/`, and `lib/`.
2. Read and execute `commands/farley-score.md` as the primary workflow.
3. Use `request_user_input` for interactive menus instead of plain-text numbered prompts.
4. Keep analysis read-only: do not modify source files while scoring/reporting.
5. Use `python3 "$COMMAND_ROOT/lib/cli_calculator.py"` for all scoring math.
6. Switch to coaching by following `commands/farley-score-coach.md` when the user asks to learn instead of review.
