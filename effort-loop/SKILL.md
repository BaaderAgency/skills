---
name: effort-loop
description: >-
  Build on low, verify on high. A Claude Code build loop: interview (medium
  effort) -> builder subagent (low effort) -> you review and iterate on low ->
  a fresh verifier subagent (high effort) runs held-out test cases and edge
  cases against the real output -> ship only on PASS. TRIGGER on "/effort-loop",
  "effort loop", "build on low verify on high", "low builder high verifier",
  "build it cheap then verify hard", or when you want a build run through a
  cheap-build / expensive-verify split with held-out tests. Testing method
  adapted from Anthropic's build-eval + hillclimb post (held-out test set, one
  change per round, revert on overfit).
---

# Effort Loop: build on low, verify on high

```
main session (medium)   interviews you, writes Done = X + test cases
        |
        v
builder   (low)         writes the code, sees TRAIN cases only
        |
        v
you review; iterate on low, one change per round
        |
        v
verifier  (high)        fresh context, runs HELD-OUT cases + edge cases
        |
        v
ship only on verifier PASS
```

Why: writing code is cheap work, finding what is wrong with it is where the
thinking pays. Low effort builds fast and is cheap to iterate; high effort is
spent on a context that has never seen the builder's reasoning.

## HARD GATES

This loop never replaces your project's own review, deploy or approval steps.
It runs inside them.

**1. The verifier is never the builder.**
- NO: verifying in the main session that wrote or reviewed the code.
- NO: reusing the builder subagent (SendMessage) as the verifier.
- NO: giving the verifier the builder's reasoning, only its output + the spec.
- YES: a new `effort-verifier` subagent, fresh each round, given: the "Done = X"
  spec, the path to the work, and the TEST cases.

**2. The builder never sees the TEST cases.**
- NO: pasting test cases, failing transcripts, or grader answers into a builder brief.
- NO: putting a verifier's example wordings, inputs, or test text into a fix brief,
  even "just as an example". Fresh verifiers invent new cases, but leaking
  examples still teaches the builder the exam.
- YES: the builder gets Done = X + TRAIN cases. A failure is passed back as ONE
  root-cause line ("empty input crashes the parser"), not the raw test.

**3. "Done" needs the verifier's evidence.**
- NO: saying "it works" on a build that has no verifier PASS.
- YES: report language without PASS: "built on low, awaiting high verification".
- YES: the verifier report states the effort level it ran at. If it cannot
  confirm it ran at high, say so and treat the PASS as unconfirmed.

**4. One change per round, revert on overfit.**
- Train up AND test up -> keep. Train up, test flat -> revert (overfitting).
  Any regression -> revert.
- Never make a change too small to show above the noise of the cases.

**5. Read every failing test name.**
- NO: accepting "only the known failure" from a test run without reading the names.
- YES: list every failing name and match each to a known cause. An unmatched name
  is a regression until proven otherwise.

## Step 0: print the check

```
EFFORT-LOOP CHECK: Done = X written? Y/N · cases split train/test? Y/N · verifier agent fresh? Y/N · verifier PASS? Y/N
```

## Step 1: Interview (main session, medium)

Ask what "done" means, then write into the project's `memory/task_plan.md`
(create it if needed):

1. **Done = X**: one or two sentences, observable, in the user's own terms.
2. **Cases (5-10 to start)**, sourced in this order: real production inputs,
   bug reports or tickets, then hand-written cases, then synthesized ones.
   - Pick cases a human judged hard or real. Never pick a case only because a
     model failed it (that samples one model's failure fingerprint).
   - Include edge cases: empty, huge, malformed, hostile, off-spec.
3. **Grader per case, cheapest that fits**: code check (exact match, schema,
   tests pass, HTTP status, file exists) first. A model judge only for
   open-ended output, with a checkable-claims rubric, never a 1-10 scale, and
   never the model being tested.
4. **Split**: randomly assign roughly 60% TRAIN, 40% TEST. Write TEST cases to
   `memory/holdout.md` and tell the user the builder will not see that file.
   Tiny builds (under ~5 cases) may skip the split, but never skip the verifier.

## Step 2: Build on low (builder subagent)

Spawn the `effort-builder` agent (`agents/effort-builder.md`, `effort: low`)
with a self-contained brief: Done = X, TRAIN cases, constraints, where to write.
Use `model: "opus"` on the Agent call. One builder per round; the brief must
stand alone (the builder cannot see this conversation).

## Step 3: Review, iterate on low

Show the result. The user's notes go back to the builder as the next round,
still on low. One change per round. Do not call the work finished here.

## Step 4: Verify on high (fresh verifier)

When the user says it looks right, spawn a NEW `effort-verifier`
(`agents/effort-verifier.md`, `effort: high`, `model: "opus"`). Brief it with
Done = X, the work path, and the TEST cases from `holdout.md`. It must run
every TEST case, try its own edge cases, exercise the real thing (run the code,
hit the route, open the output), and return PASS / FAIL with evidence and the
effort level it believes it ran at.

- **PASS** -> Step 5.
- **FAIL** -> translate each failure into a root-cause line (no raw test text),
  send to a new builder round at low, then a new verifier. Apply gate 4.
- **Stall**: after 2-3 flat rounds, sort the remaining failures by cause before
  changing anything: a broken or ambiguous case, a broken grader, a flaky
  environment (run-to-run variance, leftover state), or a real bug. Fix a bad
  case or grader first; a bad case is not a code bug.

## Step 5: Ship

Log the round history to `memory/progress.md` (round, change, train, test,
verdict). Then run your project's normal finish steps (build, deploy checks).

## Setup (once per project)

Copy the two agent files into the project so Claude Code loads them:

```
mkdir -p .claude/agents && cp ~/.claude/skills/effort-loop/agents/*.md .claude/agents/
```

**If the `effort-verifier` / `effort-builder` agent type is not available in the
session**: spawn a general Opus agent (`model: "opus"`) and paste the agent
file's rules into its brief. The effort is then self-reported: record it as
"effort self-reported, not confirmed" in `memory/progress.md`.

Effort per subagent is set by the `effort:` line in each agent's frontmatter
(Claude Code docs: https://code.claude.com/docs/en/sub-agents). It overrides the
session's effort while that subagent runs. The fallback is the `/effort low` or
`/effort high` command in the session for that step.

## Borrowed from Anthropic's eval-design post

Four traits of a good eval set: mirrors real use; scores rise with a stronger
model or more thinking; the best model at the highest effort stays well under
100% (headroom); low run-to-run variance. Check grader consistency (grade the
same output twice) and plumbing noise (timeouts, truncation) before trusting a
score. Source: "Automating eval design and hillclimbing with Claude",
https://claude.dev/blog/automating-eval-design-and-hillclimbing/
