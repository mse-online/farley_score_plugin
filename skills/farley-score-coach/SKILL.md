---
name: farley-score-coach
description: Run interactive Farley Score coaching from this repository's command spec. Use when asked to teach test quality, explain Dave Farley's properties, practice identifying anti-patterns, or guide users through improving tests with Socratic coaching.
---

# Farley Score Coach

1. Set `COMMAND_ROOT` to the repository root that contains `commands/`, `knowledge/`, `examples/`, and `lib/`.
2. Read and execute `commands/farley-score-coach.md` as the primary workflow.
3. Use `request_user_input` for interactive welcome menus and mode selection.
4. Keep coaching interactive and evidence-based; reference examples from `$COMMAND_ROOT/examples/sample-project/` when useful.
5. When the user requests autonomous scoring/reporting, hand off to `commands/farley-score.md`.
