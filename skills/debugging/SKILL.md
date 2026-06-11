---
name: debugging
description: Systematic root-cause debugging - reproduce, isolate, fix, verify. Use when investigating a bug, failing test, crash, error message, or unexpected behavior, or when the user says "this is broken", "why does this fail", or pastes a stack trace.
---

# Debugging

Find the actual root cause before changing any code. A fix without a confirmed cause is a guess.

## When to use

- A bug report, failing test, crash, error message, or "it worked yesterday" situation.
- Not for adding new behavior — only for making existing behavior match its intent.

## Process

1. Reproduce first. Run the failing scenario and capture the exact error output. If you cannot reproduce it, gather more information (exact steps, environment, versions) before touching code. Never fix what you cannot observe.
2. Read the error message and full stack trace carefully — the answer is often literally in it. Note the exact line where reality diverged from expectation.
3. Form a hypothesis about the cause, then look for evidence that would DISPROVE it, not just confirm it.
4. Isolate by bisection:
   - In code: cut the failing path in half with logging or assertions; determine which half contains the divergence; repeat.
   - In history: if it recently worked, walk the version history to find the breaking change.
   - In data: minimize the failing input to the smallest case that still fails.
5. Identify the root cause and state it in one sentence: "X fails because Y." If you cannot fill in that sentence with evidence, keep digging — do not start patching symptoms.
6. Fix the cause, not the symptom. If the bug was possible because of a missing guard or unclear contract, consider whether the fix belongs at the source rather than the crash site.
7. Verify: run the original reproduction and confirm it passes. Run the surrounding test suite to confirm nothing else broke.
8. Add a regression test that fails without the fix and passes with it.

## Output format

When reporting back: the root cause in one sentence, the evidence that confirms it, the fix applied, and the verification performed (with actual output, not assertions of success).

## Anti-patterns to avoid

- Changing code based on a hunch before reproducing the failure.
- Stacking multiple speculative changes at once — you will not know which one mattered.
- Declaring victory because the error message changed or disappeared once.
- Suppressing the symptom (catch-and-ignore, sleep, retry) when the cause is identifiable.
- Stopping at "it works now" without understanding why it failed before.
