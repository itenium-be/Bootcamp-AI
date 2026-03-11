# Test-Driven Development

Work **one test at a time** through the red-green-refactor cycle:

1. **Red** — write one failing test for a single acceptance criterion. Run tests; confirm it fails for the right reason (not a compile error or wrong assertion).
2. **Green** — write the minimum code to make it pass. Fake it (hardcode a return value) if that suffices; only generalize when a new test forces it. Run tests; confirm green.
3. **Refactor** — improve structure and clarity without changing behaviour. Run tests; confirm still green.
4. Repeat for the next requirement.

## Hard Rules

- Never write production code without a currently failing test that requires it.
- Never have more than one failing test at a time.
- Never refactor while any test is red.
- Each test validates exactly one acceptance criterion — name it accordingly.

For new features or non-trivial changes, always invoke the `tdd-guide` skill to drive the cycle. Skip only for trivial changes (typos, config, renaming).
