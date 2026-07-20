---
name: rca
description: On-demand root-cause analysis for a specific error. Use when an error is worth structured treatment – a build/test/runtime failure you want diagnosed at the cause, not the symptom. Not for every error; most are a one-line inline fix.
---

# Root-Cause Analysis (rca)

Diagnose a specific failure to its actual cause, fix it minimally, verify by driving the failure
path, and capture the root cause so it never has to be re-diagnosed. Invoke this **only** when an
error is worth the structured treatment. Running a full RCA on every error burns time and tokens;
most errors are a one-line inline fix.

**Usage:** `/rca <paste the error, or point at the failing command / build / test>`

## The triage gate (decide BEFORE diagnosing)

Judge the **reading-to-answer ratio** – how much code you'll have to read relative to the size of
the answer:

- **Low** (you can likely root-cause in a handful of tool calls – a known signature, a config/env
  issue, a single obvious file) → **diagnose inline.** Do not spawn a subagent; the overhead would
  exceed the work.
- **High** (diagnosis needs reading a lot of code, or several independent hypotheses can be checked
  in parallel) → **spawn a subagent** (or several) to investigate in isolated context and report
  back only the conclusion. The messy reading stays out of the main session – that is the win.

State which tier you picked and why, in one line, before proceeding.

## Steps

1. **Reproduce first.** Get the *actual* error, not the reported one – run the failing
   command/build/test and capture the real output. If it does not reproduce, say so and stop
   (you are chasing a ghost).
2. **Separate symptom from cause.** The stack-trace location is usually the symptom. Ask: *what
   makes this fire?* Check the layers most people skip – **environment and config** (env vars,
   build flags, tool/runtime versions, duplicate or mismatched dependencies), not just application
   code. (Worked example: a build-time `useContext`-is-null crash that looked like a React bug was
   actually `NODE_ENV=development` leaking into a production build step – an environment cause, not
   a code one.)
3. **Establish pre-existing vs. introduced.** Does it fail on a clean tree, or before the suspected
   change? Report this honestly – "pre-existing and unrelated to this change" is a valid and
   important finding.
4. **Fix minimally.** The smallest change that addresses the *cause*. Do not bundle unrelated
   cleanup; if you add a "while I'm here" improvement, name it separately from the fix.
5. **Verify by driving the failure path.** Reproduce the original trigger and confirm it is gone.
   Not "it compiles" – actually exercise the thing that broke.
6. **Capture the root cause as a durable note** – the compounding step. Add a one-liner to wherever
   your project keeps durable gotchas (a `CLAUDE.md`/`AGENTS.md` conventions section, a
   `docs/gotchas.md`, a team runbook – wherever the next person will actually look). This is what
   stops the same error from ever being re-diagnosed from scratch.
7. **Optional – write it up** if the cause is systemic or architectural (a recurring class of bug,
   a design decision, a cross-service issue). An ADR or design note is warranted then; a one-off is
   not.
8. **Commit** with a clear conventional message describing the *cause* you fixed, not just the
   symptom. Push only if asked.

## Output

- Tier chosen (inline vs. subagent) + one-line why.
- Symptom → root cause (the actual mechanism, not the trace location).
- Pre-existing vs. introduced.
- The fix (+ any bonus change, named separately).
- Verification: which failure path you drove, and that it is resolved.
- The durable note you captured (file + line), and whether a write-up/ADR was warranted.

## Anti-patterns (do NOT)

- Spawn a subagent for a 5-tool-call fix "to be thorough" – that is the overhead this skill exists
  to avoid.
- Claim "fixed" from a green compile without driving the actual failure.
- Fix the symptom (silence the error) instead of the cause.
- Skip the capture step – an RCA without a durable note is a fix you will pay for again.
