---
name: fixthis
description: Diagnose and fix a bug the right way — reproduce first, capture it in a failing test, prove the root cause before changing code, fix minimally, verify no regressions, and close the gap that let it through. User-invoked only; does not auto-trigger.
argument-hint: [bug description or issue]
disable-model-invocation: true
---

# Debugging Operating Directive

**Bug:** $ARGUMENTS

Fix the bug above under this directive. A fix that hides the symptom, that you can't explain, or that breaks something else is not a fix.

**Accuracy and correctness outrank speed and token cost.** Never skip or shrink a verification step to save time or tokens; when in doubt, run the wider check.

## 1. Reproduce before anything else

- **Reproduce it reliably first.** A bug you can't reproduce is a bug you can't confirm fixed. Establish the exact inputs, state, environment, and steps that trigger it.
- Pin down **expected vs. actual** behavior precisely — that gap is your definition of done.
- **Confirm the expected behavior is actually specified** (docs, tests, contract, commit/PR history, or my statement) — not assumed. Bug reports can be wrong. If "expected" is itself uncertain, **settle it from those sources before you touch code** — that is research you can do, not a blocker. Only when the sources genuinely don't contain the answer *and* it is a product call rather than a fact is it mine to make: ask me directly, then (Section 9).
- **If the bug is intermittent**, establish a baseline reproduction rate first: run the reproduction N times and record failures/N. Without a baseline you cannot distinguish "fixed" from "got lucky."
- If you can't reproduce it, do not guess-fix — and do not park it either. **"Not reproducible" is where the work starts, not a verdict.** Escalate deliberately: re-read the report for the exact version, build, OS, and environment; pull logs, stack traces, and the actual failing input; reconstruct the caller's *state* rather than the caller's summary; replay against the reported revision, not just the current one; instrument the suspect path; then push the harsh conditions a happy path hides — concurrency and ordering, cold cache, empty/oversized/malformed input, clock, timezone and locale, resource exhaustion, permission and network failure. Most irreproducible bugs are an unmatched precondition, not a mystery. This ladder parallelizes well (Section 8). Escalate to a **gating item (Section 9) only after you have genuinely exhausted it** and the missing piece is something only I or an external system can supply — and then ask for that one thing outright rather than filing it and moving on.

## 2. Triage: incident vs. latent bug

- **Active production incident** (users impacted, data at risk): stop the bleeding first. Apply the fastest *safe* mitigation — rollback, feature flag, disable the path — before root-causing. Mitigation and root cause are different jobs; do the urgent one first, then the correct one, and don't let the mitigation become the permanent "fix."
- **Latent / non-urgent bug:** go straight to disciplined root-cause. No hotfix-and-move-on.
- State which mode you're in and why. **Stop and ask** before any mitigation with real side effects (rolling back live data, flags that change behavior for real users, anything irreversible).

## 3. Capture the bug in a failing test

- Before fixing anything, write a test that reproduces the bug and **fails for the right reason**. This is your reproduction, encoded: it proves the bug exists now and will prove it's gone later.
- If the bug genuinely can't be expressed as an automated test, say so explicitly and define the exact manual reproduction you'll use to verify. Never skip verification because it's inconvenient.

## 4. Diagnose the root cause — don't patch the symptom

- **Localize before you theorize.** Narrow the failure to the smallest region: bisect commits, binary-search the code path, add targeted instrumentation/logging, inspect state at the boundary. Shrink the search space before forming theories.
- Form an **explicit hypothesis** about the mechanism, then **confirm it** — reproduce the *cause*, not just the symptom. Prove *why* the bug happens before you change a line.
- **One hypothesis at a time in the working tree — and revert every failed probe before trying the next.** A tree polluted by dead attempts makes the eventual fix unattributable and leaks junk into the diff. (To test several hypotheses at once, give each one its own isolated copy or worktree — see Section 8. The rule is about the *shared* tree, not about serial thinking.) If two consecutive hypotheses fail, stop patching: re-read the full code path from scratch, list every assumption you haven't verified, and check them cheapest-first.
- **Never "fix" what you can't explain.** If a change makes the symptom disappear but you can't state the mechanism in one sentence, you haven't fixed the bug — you've masked or relocated it. Keep going until you can name the cause plainly.

## 5. Fix minimally and surgically

- Change the **least code** that corrects the root cause. Resist refactoring, cleanup, or fixing adjacent issues while you're in there — that expands blast radius and muddies the diff. Note them separately instead.
- Fix the **cause, not each symptom** it produced. One root cause often has several visible effects; correcting the cause should resolve them together.
- Hold the normal quality bar: single responsibility, explicit interfaces, no duplicated logic, and **no silent failures** — fail loud, with context. Don't introduce a swallowed error while removing one.

## 6. Verify the fix

- **Evidence over assertion.** Never report a result you did not observe in this session. "Passes," "works," and "fixed" must be backed by actual command output — the run, the exit status, the test count.
- The failing test from Section 3 now **passes**.
- The **original reproduction** no longer triggers the bug, under the same conditions it first appeared.
- **For an intermittent bug**, the fix is verified only when the same N-run battery from Section 1 shows zero failures — never claim a nondeterministic bug fixed from a single green pass.
- **Re-run the suite** (widen to the full suite when the change could ripple) to confirm no regression — a fix that breaks something else is not a fix.
- **Adversarially probe** the fix: nearby inputs, boundaries, and the exact conditions that produced the bug, to confirm you fixed the *class* and not just the one case.

## 7. Close the gap

- Ask **why it wasn't caught**, and fix that too: the missing test, the weak validation, the swallowed error, the absent guardrail. The regression test from Section 3 is the floor; add the guardrail that would have caught the whole class.
- **Prevent the class, not just the instance.** The regression test pins one point in the input space; climb as high on this ladder as the code allows, and state which rung you stopped at and why:
  1. **Make the bug unrepresentable** — a tighter type, a validating constructor, an invariant enforced at the boundary, so the bad state can't be constructed at all.
  2. **Property-based or fuzz test** over the bug's input domain, so the whole class is exercised continuously — not just the one reported case.
  3. **Example-based regression test only** (the Section 3 floor) — acceptable alone only when the input domain is trivial or the cost of the higher rungs clearly exceeds the blast radius.
  For anything touching money, auth, or persistent-data mutation, rung 3 alone is never sufficient — reach rung 1 or 2.
  If rung 1 or 2 requires structural change beyond the minimal fix, land the minimal fix + regression test as its own verified commit *first*, then the class-prevention work as a separate commit. Never mix them in one diff.
- **Hunt for siblings — then fix them.** Search the codebase for the same bug pattern elsewhere; bugs of a kind travel in packs, and fixing one while ignoring its twins is half a job. You already hold the diagnosis and the test pattern, so the marginal cost per sibling is small: **repair each one, with its own regression coverage**, as commits separate from the primary fix. The search itself parallelizes well — see Section 8. Defer a sibling only when repairing it is genuinely a different project (another subsystem, a structural refactor, or a change I would have to approve); then say so out loud and get my call rather than quietly filing it (Section 9).

## 8. Parallel work with subagents — use them, under these rules

Spawning subagents (the Agent tool, `/code-review`, or the equivalent available to you) is **encouraged, not merely tolerated.** A wide sibling hunt, a multi-file code-path trace, a matrix of reproduction attempts, or an independent review is faster in parallel and often *better*, because each agent arrives without your assumptions. Reach for them whenever breadth is the bottleneck — and never in a way that can corrupt state or launder an unverified claim into a verified one.

**Spawn for — all read-only or isolated:**
- **Reconnaissance:** trace a call path, find every caller of a changed function, locate the spec or the commit that introduced the behavior, read a dependency's installed source to confirm a signature.
- **The sibling hunt (Section 7):** one agent per pattern, per subsystem, or per naming convention — they cover far more ground than one sequential search.
- **Reproduction sweeps (Section 1):** the same reproduction across different inputs, versions, or environments — provided each run gets its own workspace, port, database, and temp paths.
- **Independent hypothesis exploration**, but only when each agent works in **its own isolated copy or worktree**. Section 4's "one hypothesis at a time" is a rule about the *working tree* and it holds: two agents probing the same checkout produce an unattributable fix and a polluted diff.
- **Fresh-context adversarial review:** an agent with no authoring context trying to refute your root cause or break your fix — required by Section 11 for money, auth, persistent data, or production.

**Do not spawn for:** work with a sequential dependency on something still in flight; the minimal fix itself (one author, one diff); a task smaller than the cost of briefing an agent; or anything that needs conversation state only you hold.

**Hard rules — these prevent the failures that actually happen:**

1. **One writer per file, always.** Subagents default to **read-only**; say so in the brief. If one must write, give it explicit ownership of **disjoint** named paths, and never let two agents hold the same path. You are the integration owner: you merge, you re-run the suite on the merged result, you resolve conflicts.
2. **Namespace every scratch path, per agent.** Agents share a scratch directory. Identically-named temp files, logs, fixtures, and report files silently clobber each other and yield a plausible-looking garbage result — this has already happened. Give each agent a unique prefix or its own subdirectory, and tell it so.
3. **Never run two builds, migrations, or test suites concurrently in the same working tree.** They race on build artifacts, lock files, fixed ports, shared databases, caches, and coverage output. Serialize them, or give each agent an isolated worktree with its own port and database name. Never point two agents at the same live database, external account, or shared device.
4. **Never let a subagent write shared session state** — the session log, `memory/gating.md`, the session registry, or any reference file the parent session owns. Subagents have no lane and no ownership of those files. They **return findings to you**; you are the only writer.
5. **A subagent's claim is not evidence.** Require each agent to return the exact command it ran and the actual output. "Tests pass" without the run is unverified — either re-run it yourself or report it as unverified. Section 6's evidence bar applies to delegated work exactly as it applies to your own.
6. **Brief each agent completely:** the goal, the files it may read and (if any) write, its scratch prefix, what to return and in what shape, and that **finding nothing is a valid and useful answer.** An agent left to guess its scope will invent one.
7. **Bound the fan-out** to what you can actually reconcile — and then reconcile every result. Contradictory findings between agents are a signal to investigate, never something to average out or quietly drop.

## 9. Gating items — the last resort, not the pressure valve

**Default: solve it.** The gating list is not a place to put work you would rather not do, and adding to it is not progress. Before anything goes on it, you must have actually tried: read the relevant code and its history, reproduced the constraint, searched for how this codebase already solves the same problem, checked the installed version's own source or docs, and attempted at least one concrete route to a fix. **"Hard", "unclear", "would take a while", "needs more investigation", "I'd have to read that subsystem" are not gates — they are the task**, and Section 8 exists partly so that breadth is never the excuse.

**An item is a legitimate gate only when clearing it requires something you cannot supply** — specifically one of:

1. **A decision only I can make** — a behavior or product call the spec doesn't contain, a trade-off with no defensible default, or approval for something irreversible, breaking, or expensive.
2. **Access you don't have** — a credential, permission, environment, device, or dataset.
3. **An external party or system** — an upstream fix, a vendor, a service that is down, something outside this codebase's control.
4. **A deferral I explicitly approved** in this session.

If none of those four apply, it is not a gate. Finish it.

**Prefer asking over filing.** If I am in the session and the blocker is (1), ask me the question directly, now — a parked item that one exchange would have answered is a failure of this directive, not a record of diligence. File it only when I am unavailable, or when the answer isn't needed for you to keep making real progress elsewhere.

**Every filed item must carry three things, in the item itself:** what you actually tried and why each route failed; the **single specific** thing that would clear it; and who or what supplies that thing. An item missing any of the three is a shrug, not a gate.

For whatever legitimately remains:

- Maintain a dedicated, **persistent GATING list** in the project's `memory/gating.md` (create it on the first gating item, update it in place — it must survive session end and context compaction).
- Scaffold around each where possible, make the unresolved path **fail loudly** (never a silent no-op or fake success), and keep the list current — added on discovery, cleared the moment it resolves, never silently forgotten. This is the source of truth for what still blocks a clean fix.
- **Keep the file organized into exactly two sections — `## OPEN` first, then `## RESOLVED`** — so a reader sees live blockers up top and closed ones never clutter them. Every item is its own `### <ID> — <headline>` block filed under one of those two headings; those are the only two top-level (`##`) sections in the file.
  - **New item** → add it under `## OPEN`, newest at the top of the section.
  - **When you clear one, MOVE it, don't relabel it in place:** cut the whole block — text preserved verbatim — out of `## OPEN` and append it to the end of `## RESOLVED`. A gate counts as closed (fixed / done / won't-fix / superseded) even if it carries documented residuals; capture any still-live successor as its **own new** item under `## OPEN`, never by keeping the parent open.
  - **Still actionable** (needs a decision, a build, an operator go/no-go, or is parked/deferred) → it stays under `## OPEN`.
  - One ID lives in exactly one section — never duplicated across both, never left at `##`/top level outside the two sections.

## 10. Above all

- **Do not break existing functionality.** Backward compatibility is a hard constraint unless I've explicitly approved otherwise.
- Don't let the fix quietly change unrelated behavior. The diff should do exactly one thing: kill this bug.

## 11. Commit & deploy (gated, not automatic)

1. For any fix touching **money, auth, persistent data, or production**: before commit, have a **fresh-context reviewer** (a subagent with no authoring context under Section 8's rules, or /code-review) adversarially attempt to refute the root-cause claim and find defects in the diff. The context that wrote a fix is systematically biased when reviewing it; cost is never a reason to skip this.
2. **Commit atomically:** the fix and its regression test together, with a message that states the **root cause** — not "fix bug." Never commit a red state or secrets.
3. Before deploy: **full suite + build + typecheck + lint green**, including the new regression test.
4. Confirm the **GATING list contains nothing that blocks correctness** in the target environment.
5. Summarize the **root cause, the fix, and its blast radius** — separating what was *demonstrated* (exercised by a test or run you executed) from what is *inferred* (reasoned but not exercised).
6. For anything irreversible or production-facing, get my **explicit go-ahead** and confirm the rollback path. For an incident, keep the mitigation in place until the fix is verified live, then remove it deliberately.

Only then deploy.
