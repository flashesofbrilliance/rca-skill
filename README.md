# rca

A [Claude Code](https://claude.com/claude-code) skill that turns "something broke" into a
disciplined root-cause analysis: triage the effort, reproduce the *real* error, separate symptom
from cause, fix minimally, verify by driving the failure path, and capture the root cause so nobody
re-diagnoses it next month.

It encodes one opinion most debugging workflows skip: **decide how much to spend before you start.**
Not every error deserves a full investigation. The skill's first move is a triage gate that routes a
five-minute fix inline and reserves subagent fan-out for the cases that actually need it.

---

## Why it exists

Two failure modes show up again and again when an agent (or a person) debugs under pressure:

1. **The fix lands on the symptom, not the cause.** The stack trace gets silenced, the build goes
   green, and the same bug fires again next week from a slightly different angle.
2. **The fix leaves no trace.** Whatever was learned about *why* it broke evaporates, so the next
   person re-derives it from scratch.

`rca` makes the cause-not-symptom discipline and the capture-the-root-cause step non-optional, and
adds a triage gate up front so the process itself does not become the waste.

---

## Install

The repository root *is* the skill, so you can clone it straight into a skills directory.

**User scope (available in every project):**

```bash
git clone https://github.com/flashesofbrilliance/rca-skill.git ~/.claude/skills/rca
```

**Project scope (this repo only):**

```bash
git clone https://github.com/flashesofbrilliance/rca-skill.git .claude/skills/rca
```

Prefer not to clone? Copy `SKILL.md` into `~/.claude/skills/rca/SKILL.md` (or the project-scoped
`.claude/skills/rca/SKILL.md`) by hand. That single file is the skill.

Restart or reload Claude Code so it picks up the new skill.

---

## How to use

Invoke it in a Claude Code session and hand it the failure:

```
/rca <paste the error, or point at the failing command / build / test>
```

Examples of what to pass:

```
/rca npm run build is failing on main with "Cannot read properties of null (reading 'useContext')"
/rca the auth test in tests/login.spec.ts started flaking after the last deploy
/rca <paste a full stack trace here>
```

**Reach for it when** an error is worth structured treatment: a build, test, or runtime failure you
want diagnosed at the cause, especially one that has bitten you before or that you do not fully
understand yet.

**Do not reach for it when** the fix is obvious and one line. Running a full RCA on every trivial
error burns time and tokens. The skill says this to itself, and its own triage gate will route a
small fix inline rather than over-process it.

---

## Use cases

- **The flaky test.** A test passes locally and fails in CI. `rca` forces a real reproduction first
  (against the environment where it actually fails), separates the flake's trigger from its
  stack-trace location, and captures the finding so the flake is not re-investigated on its next
  appearance.
- **The "works on my machine" build break.** A build crashes only in one environment. The skill
  checks the layers most debugging skips first: env vars, build flags, tool and runtime versions,
  duplicate or mismatched dependencies, before touching application code.
- **The regression after a deploy.** Something broke and it is unclear whether the last change did
  it. `rca` explicitly establishes pre-existing vs. introduced, and treats "pre-existing and
  unrelated to this change" as a valid, reportable finding rather than something to paper over.
- **The recurring error you keep re-diagnosing.** Every RCA ends by writing a one-line durable note
  wherever your project keeps its gotchas (a `CLAUDE.md` / `AGENTS.md` conventions section, a
  `docs/gotchas.md`, a team runbook). The point is to pay the diagnosis cost once.
- **The large, ambiguous failure.** When several independent hypotheses need checking or the
  diagnosis requires reading a lot of code, the triage gate routes the investigation to subagents so
  the messy reading stays out of your main session and only the conclusion comes back.

---

## What a run looks like

`rca` reports in a fixed shape so the output is scannable and comparable across runs:

- **Tier chosen** (inline vs. subagent) and a one-line why.
- **Symptom → root cause** the actual mechanism, not the trace location.
- **Pre-existing vs. introduced.**
- **The fix** (plus any bonus change, named separately).
- **Verification** which failure path was driven, and that it is resolved.
- **The durable note** captured (file + line), and whether a write-up or ADR was warranted.

---

## The method at a glance

1. **Triage gate** decide inline fix vs. subagent investigation by the reading-to-answer ratio,
   and state the choice before diagnosing.
2. **Reproduce first** get the actual error, not the reported one.
3. **Separate symptom from cause** check environment and config, not just application code.
4. **Pre-existing vs. introduced** reported honestly.
5. **Fix minimally** smallest change that addresses the cause; bonus changes named separately.
6. **Verify by driving the failure path** not "it compiles."
7. **Capture the root cause** as a durable note so it is never re-diagnosed.
8. **Commit** with a message describing the cause you fixed, not just the symptom.

**Anti-patterns it guards against:** spawning a subagent for a five-tool-call fix; claiming "fixed"
from a green compile without exercising the failure; fixing the symptom instead of the cause; and
skipping the capture step.

Full instructions live in [`SKILL.md`](./SKILL.md).

---

## License

MIT. See [LICENSE](./LICENSE).
---

## Part of the ARCS family

An open, MIT-licensed tool in the [flashesofbrilliance](https://github.com/flashesofbrilliance) / ARCS family — small, composable, provenance-carrying. The tools are open; the ARCS intelligence that orchestrates them is private.


---

## Part of the ARCS ecosystem

A standalone, independently-adoptable open tool — small, composable, provenance-carrying. The [`arcs`](https://github.com/flashesofbrilliance) umbrella is a thin aggregator that links to tools like this one; it never vendors or owns them. The tools are open; the tailored ARCS that orchestrates them is private.
