---
description: Run a persistent test-fix loop until all tests pass. Use when the user asks to keep iterating until tests are green, fix failing tests, or grind until tests pass.
---

# Test-Fix Loop

Run an autonomous loop that executes the test suite, analyzes failures, applies fixes,
and repeats until all tests pass or the maximum iteration limit is reached.

## Behaviour

The agent runs this loop manually, iteration by iteration, until the exit condition is met.
The loop terminates when:
- All tests pass AND `.cursor/scratchpad.md` contains `DONE`, OR
- The iteration count reaches `MAX_ITERATIONS` (default: 5)

> **Advanced extension**: For a fully autonomous loop that continues between agent turns
> without user intervention, wire a `stop` hook in `.cursor/hooks.json` pointing to a
> script that reads `scratchpad.md` and injects a follow-up message. This requires
> Cursor's **Nightly** release channel. Add it only when the team needs it.

## Steps per Iteration

1. Run the full test suite:
   ```bash
   <test-command>
   ```
2. If all tests pass:
   - Write `DONE: all tests passing as of iteration N` to `.cursor/scratchpad.md`
   - Stop the loop
3. If tests fail:
   - Identify the failing test(s) and read the error output carefully
   - Locate the relevant source file(s) using grep or semantic search
   - Apply the minimal fix required to make the failing test(s) pass
   - **Do not** modify test files to make them pass — fix the implementation
   - Re-run only the affected test file for speed: `<test-command> <file>`
   - Log progress to `.cursor/scratchpad.md`:
     ```
     [Iteration N] Fixed: <brief description of change>
     Remaining failures: <count>
     ```
4. Continue to the next iteration

## Notes

- This skill is intentionally narrow — it only fixes failing tests, not style or type errors.
- If a test is fundamentally broken (wrong assertion, wrong mock), stop and ask the user.
- Do not attempt more than 3 changes to the same file in one loop — escalate to the user.
