---
name: engineering-system
description: >-
  Unified engineering-quality and debugging discipline (forge + debug matched
  pair in one file). Use it proactively for essentially any non-trivial coding
  work: writing, refactoring, reviewing, or designing code and systems —
  backend, frontend, security, architecture, performance, testing, git,
  databases, APIs, CI/CD — even when the user never asks for
  "production-grade." Also use it whenever the user reports a bug, crash,
  error, flaky test, wrong output, or regression, says "something's broken" or
  "why is X happening", or pastes a stack trace: Part II enforces a reproduce →
  hypothesize → probe → verify → fix loop with bisection and a circuit
  breaker. It scales ceremony to task size (Tier 0–2 / D0–D2), so it stays
  active on small tasks without over-building them. NOT A FIT FOR: purely
  conceptual explanations that produce no code, or trivial one-line edits like
  a typo or rename.
---

# Engineering System

A single discipline with two matched halves:

- **Part I — Forge**: how to *build* — writing, refactoring, reviewing, and designing code.
- **Part II — Debug**: how to *fix* — a strict evidence-driven loop for anything broken.

Both halves scale ceremony to the size of the task. Classify first, then apply
only the rules of that tier. Never apply Tier 2 ceremony to a Tier 0 task.

---

## Part I — Forge

### Step 1: Classify the tier

| Tier | Scope | Examples |
|------|-------|----------|
| **0** | Small, contained, low blast radius | helper function, config tweak, small refactor, test addition |
| **1** | Standard feature work | new endpoint, schema change, module refactor, integration |
| **2** | High stakes or high complexity | auth/security/payments, data migrations, concurrency, public API contracts, cross-service changes |

If unsure between two tiers, take the higher one.

### Tier 0 — always on

These rules apply to *every* change, no matter how small:

1. **Read before you write.** Look at the surrounding code, its idioms, naming,
   and error-handling style. New code must read like it was written by the
   same author as the file it lives in.
2. **Handle the failure path.** Every I/O call, parse, cast, or external call
   either handles its failure or deliberately lets it propagate — never
   silently swallows it.
3. **No dead weight.** No commented-out code, no unused imports, no
   speculative parameters "for later."
4. **Verify, don't assume.** Run the code path you touched — a test, a REPL
   call, the actual app. "It compiles" is not verification.
5. **Name things for the reader.** A name should make the *next* reader's job
   easier, not record your thought process.

### Tier 1 — standard feature work

Everything in Tier 0, plus:

6. **State the contract first.** Before writing the body, be explicit (in code
   or one sentence) about inputs, outputs, error behavior, and edge cases —
   empty, null, zero, duplicate, too large, concurrent.
7. **Design the seam.** New code should attach at an existing boundary
   (interface, module, layer). If you must create a new boundary, that's a
   design decision — say so and justify it.
8. **Tests prove the contract.** Cover the happy path, at least one failure
   path, and the edge case most likely to be hit in production. Test behavior,
   not implementation.
9. **Migrations and data changes are reversible** or have an explicit,
   written-down rollback story.
10. **Commit in reviewable units.** Each commit is one logical change with a
    message that explains *why*, not just *what*.

### Tier 2 — high stakes

Everything in Tiers 0–1, plus:

11. **Write the design before the code.** A short design note: the problem,
    the chosen approach, at least one rejected alternative and why, and the
    failure modes considered.
12. **Enumerate the threat/failure model.** For security-sensitive code:
    who can call this, with what data, and what happens if they lie. For
    infrastructure: what happens on partial failure, retry, replay, and
    concurrent execution.
13. **Idempotency and invariants are explicit.** State the invariants the
    system must preserve and show where each one is enforced.
14. **Plan the rollout.** Feature flag, staged deploy, monitoring signal, and
    the trigger condition for rolling back.
15. **Get a second pass.** Re-review your own diff top-to-bottom as a hostile
    reviewer before declaring it done.

---

## Part II — Debug

Entered whenever something is broken: a bug report, crash, error, flaky test,
wrong output, regression, or a pasted stack trace.

### Step 1: Classify the depth

| Depth | Situation | Ceremony |
|-------|-----------|----------|
| **D0** | Cause is obvious from the error and one look at the code | Confirm with one probe, fix, verify |
| **D1** | Cause is unclear; system is deterministic | Full loop, single-threaded investigation |
| **D2** | Flaky, intermittent, concurrent, environment-dependent, or "impossible" | Full loop + bisection + logging instrumentation |

### The loop (mandatory at D1/D2)

Repeat until fixed:

1. **Reproduce.** Get a minimal, repeatable trigger for the failure. If you
   cannot reproduce it, your only job is to add the instrumentation that will
   catch it next time — do not "fix" what you cannot see.
2. **Hypothesize.** State *one* specific, falsifiable cause. Write it down.
   "Something is wrong with the cache" is not a hypothesis; "the cache returns
   stale entries because invalidation runs before the write commits" is.
3. **Probe.** Design the cheapest observation that would *disprove* the
   hypothesis — a log line, a debugger breakpoint, an isolated unit call, a
   `git bisect` step. Run it.
4. **Verify.** Compare the observation to the prediction. If disproven,
   discard the hypothesis completely and go to 2 — do not patch a dead
   hypothesis to keep it alive.
5. **Fix.** Only once a hypothesis survives a probe designed to kill it:
   apply the smallest fix that addresses the *cause* (not the symptom),
   re-run the reproduction from step 1, and confirm the failure is gone
   *and* nothing adjacent broke.

### Bisection

When the search space is large — "it worked last week," a 500-line function, a
20-step pipeline — bisect instead of reading:

- **History:** `git bisect` between the last known-good and first known-bad commit.
- **Data:** halve the input until the smallest failing case remains.
- **Code path:** cut the pipeline in half with one probe; recurse into the failing half.

Each bisection step must cut the remaining space roughly in half. If a step
can't, it's not a bisection step — pick a better probe.

### Circuit breaker

After **3 consecutive disproven hypotheses**, stop. Do not fire a fourth guess.
Instead:

1. Write down everything *known true* from probes so far (not beliefs — observations).
2. Explicitly list the assumptions you have not yet tested — the bug usually
   lives in the one labeled "surely that can't be it."
3. Re-verify the reproduction still fails the same way (the ground may have shifted).
4. Widen the frame one level: wrong file? wrong service? wrong environment?
   stale build? Test the *frame* before generating new hypotheses inside it.

### Exit criteria

A bug is fixed only when all three hold:

- The original reproduction now passes.
- You can state the root cause in one sentence.
- A test or check exists that would have caught it.

---

## Anti-patterns (both parts)

- **Shotgun surgery**: changing several things at once and re-running — you
  learn nothing from either success or failure.
- **Symptom patching**: adding a null check where the null should never have
  been possible.
- **Ceremony inflation**: writing a design doc for a rename, or demanding a
  reproduction script for an obvious typo. Match the tier.
- **Verification theater**: claiming "done" on the strength of a compile,
  a lint pass, or an untouched test suite.
