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
- If you can't reproduce it, do not guess-fix — and do not park it either. **"Not reproducible" is where the work starts, not a verdict.** Escalate deliberately: re-read the report for the exact version, build, OS, and environment; pull logs, stack traces, and the actual failing input; reconstruct the caller's *state* rather than the caller's summary; replay against the reported revision, not just the current one; instrument the suspect path; then push the harsh conditions a happy path hides — concurrency and ordering, cold cache, empty/oversized/malformed input, clock, timezone and locale, resource exhaustion, permission and network failure. Most irreproducible bugs are an unmatched precondition, not a mystery. This ladder parallelizes well (Section 8). Before you start it, set yourself a concrete budget — a number of rungs or a wall-clock limit — and say what it is. **When the budget runs out, the result is a question to me, not an entry on a list:** bring the specific thing you need, everything you ruled out and how, and the one observation that would break the tie. Escalate to a **gating item (Section 9) only after you have genuinely exhausted the ladder**, the missing piece is something only I or an external system can supply, and I am genuinely unavailable to answer it.

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
- **The ledger is part of verification.** Before you report the fix verified, run Section 9's count. A fix that is green on tests and net-positive on open gates is not finished.

## 7. Close the gap

- Ask **why it wasn't caught**, and fix that too: the missing test, the weak validation, the swallowed error, the absent guardrail. The regression test from Section 3 is the floor; add the guardrail that would have caught the whole class.
- **Prevent the class, not just the instance.** The regression test pins one point in the input space; climb as high on this ladder as the code allows, and state which rung you stopped at and why:
  1. **Make the bug unrepresentable** — a tighter type, a validating constructor, an invariant enforced at the boundary, so the bad state can't be constructed at all.
  2. **Property-based or fuzz test** over the bug's input domain, so the whole class is exercised continuously — not just the one reported case.
  3. **Example-based regression test only** (the Section 3 floor) — acceptable alone only when the input domain is trivial or the cost of the higher rungs clearly exceeds the blast radius.
  For anything touching money, auth, or persistent-data mutation, rung 3 alone is never sufficient — reach rung 1 or 2.
  If rung 1 or 2 requires structural change beyond the minimal fix, land the minimal fix + regression test as its own verified commit *first*, then the class-prevention work as a separate commit. Never mix them in one diff.
- **Hunt for siblings — then fix them.** Search the codebase for the same bug pattern elsewhere; bugs of a kind travel in packs, and fixing one while ignoring its twins is half a job. You already hold the diagnosis and the test pattern, so the marginal cost per sibling is small: **repair each one, with its own regression coverage**, as commits separate from the primary fix. The search itself parallelizes well — see Section 8. **Scope, not difficulty, is the only reason to stop.** A sibling in the same subsystem is repaired in this session, however many turn up — volume is precisely what Section 8's parallelism is for. A sibling that is genuinely a *different deliverable* (another subsystem, a structural refactor, or a change I would have to approve) is **proposed to me as its own named task in your summary, now** — a piece of work with a scope and an estimate, not a line on the gating list. "There were more of them than expected" is not a deferral reason.

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

**A problem you found yourself is not a gate — it is the rest of the task.**

Most of what reaches a gating list never blocked anything. It is something *you* discovered while the work was going well: a review finding, a sibling defect, a residual left by a partial fix, a hardening item you chose not to build, a rung of the class-prevention ladder you did not climb. None of it arrived from outside. By definition you had the context, the access, and the diff in front of you — so **all four categories above fail, and it is not eligible for the list.**

Put every such finding through two questions, in order, before it goes anywhere:

1. **Does it pass the four-category test on its own merits?** Not "is it adjacent to a gate" — does *clearing it* need a decision only I can make, access you do not have, or an external party? If not, it is not a gate.
2. **If it were fixed, would anything change?** If it cannot produce a wrong answer, lose or corrupt data, mislead a reader, or change a decision — then it is **not worth fixing and not worth recording as a blocker.** Give it one line in your summary and let it go. An entry nobody will ever act on is not documentation; it is a cost levied on every future reader of that file, and it will still be open a year from now.

Everything that survives both questions is **fixed in this session, with its own verification.**

Anything you are tempted to headline "residuals", "minor", "misc findings", "further hardening", or "all LOW" is this failure in its most recognisable form. A bundle is never a gate. Split it, fix what matters, drop what does not, and file none of it.

**Prefer asking over filing.** If I am in the session and the blocker is (1), ask me the question directly, now — a parked item that one exchange would have answered is a failure of this directive, not a record of diligence. File it only when I am unavailable, or when the answer isn't needed for you to keep making real progress elsewhere.

**Every filed item must carry three things, in the item itself:** what you actually tried and why each route failed; the **single specific** thing that would clear it; and who or what supplies that thing. An item missing any of the three is a shrug, not a gate.

For whatever legitimately remains:

- Maintain a dedicated, **persistent GATING list** in the project's `memory/gating.md` (create it on the first gating item, update it in place — it must survive session end and context compaction).
- Scaffold around each where possible, make the unresolved path **fail loudly** (never a silent no-op or fake success), and keep the list current — added on discovery, cleared the moment it resolves, never silently forgotten. This is the source of truth for what still blocks a clean fix.
- **Keep the file organized into exactly two sections — `## OPEN` first, then `## RESOLVED`** — so a reader sees live blockers up top and closed ones never clutter them. Every item is its own `### <ID> — <headline>` block filed under one of those two headings; those are the only two top-level (`##`) sections in the file.
  - **New item** → add it under `## OPEN`, newest at the top of the section.
  - **When you clear one, MOVE it, don't relabel it in place:** cut the whole block — text preserved verbatim — out of `## OPEN` and append it to the end of `## RESOLVED`. A gate counts as closed (fixed / done / won't-fix / superseded) even if it carries documented residuals. **If a residual is still live when you close the parent, fix it now** — it is a finding, and the two questions above govern it. Open a successor item **only** if that residual passes the four-category test on its own; a successor is a new gate and the ledger below counts it like any other. Never split one item into several to record nuance — the item's own text is where nuance belongs.
  - **Still actionable** (needs a decision, a build, an operator go/no-go, or is parked/deferred) → it stays under `## OPEN`.
  - One ID lives in exactly one section — never duplicated across both, never left at `##`/top level outside the two sections.
- **Every item under `## OPEN` carries a `Blocks:` line** naming what it actually holds up, which of the four categories it falls under, and who supplies the answer. If you cannot write that line, it is not a gate — it is a finding, and the two questions above govern it. This is the invariant that keeps the list honest: an OPEN count you cannot justify line by line is not a measure of anything.
- **A third destination, for the item that is true and blocks nothing:** `memory/accepted-limitations.md`. A known-unknown nobody is waiting on, a limitation deliberately accepted, a defect judged not worth fixing, an item whose trigger was retired and left it dormant — these are worth recording so a later sibling hunt reads the record instead of re-litigating the item. They are **not gates**, and under `## OPEN` they are indistinguishable from live blockers and never leave. Move the block there verbatim, keep its ID, leave no copy behind, and do not treat the file as a to-do list — it is not expected to shrink. If an entry later acquires a real trigger, move it back under `## OPEN` and give it a `Blocks:` line.
- **Every entry there carries an `Accepted:` line** stating affirmatively why nothing is waiting on it — the mirror of `Blocks:`, and it exists for a specific reason. If an entry qualified merely by *omitting* a `Blocks:` line while `## OPEN` demanded an affirmative one, then moving an item across would be cheaper than fixing it, and a ledger that counted only `## OPEN` would read the move as a win. Requiring a positive statement on both sides means a migration has to be an outright lie rather than an omission.
- **An entry may also carry a `Trigger:` line — the condition that would end the acceptance.** There are **three** outcomes, not two: blocked, permanently accepted, and *accepted for now*. "Not blocking today, but revisit before the auth rewrite" is the third, and it is common. Without a sanctioned form for it you get the worst of both: in `## OPEN` it inflates the blocker count, and in `accepted-limitations.md` — a file explicitly *not* a to-do list — the obligation quietly evaporates. A `Trigger:` records what would revive the item; it does **not** relax the `Accepted:` bar, because "we will look at it later" is not a reason nothing is waiting on it now.
- **A trigger nobody re-reads is worse than no trigger, so it must be enumerable and counted.** Report pending entries in the ledger, and when you next touch the project, check whether any trigger has fired — an entry saying "before the auth rewrite" is worthless if the auth rewrite ships and nobody looks. When one fires, move the block back under `## OPEN` and give it a `Blocks:` line.
- **There is no destination for "work I intend to do", and that is deliberate.** A finding that blocks nothing and that you have not accepted does not get parked, noted, or carried — it is resolved before you report done, in one of the dispositions already named: done now, asked about, filed as a category 1-4 gate, moved to accepted-limitations with an `Accepted:` line, or dropped as no-consequence. **There is no sixth, and a suggestion voiced in prose is not a disposition.** "We should probably also…", "a follow-up would be…", "worth revisiting later" — that sentence, with nothing behind it, is the failure this rule exists to stop: it reads as diligence, it costs you nothing, and it records nothing, so the task gets reported as complete with its real remainder sitting in prose I skim once. The moment you catch yourself writing one, stop and resolve it.
- **Why no list exists to put it on.** Every register of not-quite-work grows without bound and is read by nobody, and a ledger that prices the recorded outcomes while leaving one outcome free makes the free one the cheapest escape — which is precisely how the count stays flat while the work evaporates. So: if it is small, do it now (Section 5). If doing it now would derail this task, it is big enough to ask me about, and my answer is the record. If it is neither of those, it did not survive the "would anything change" question and it is a drop.
- **Every finding gets one line in the summary — including the ones that never touch either file.** The ledger's per-item lines cover the delta in `gating.md` and `accepted-limitations.md`; a finding you resolved by doing it, by dropping it, or by asking never enters that delta and is otherwise invisible. Give it its own line anyway. That line is what lets me say "no, do that one" while the work is still in front of us, and it is the only thing that makes a drop safe.

**The ledger — you may not finish a task by adding to the list.**

Count the items under `## OPEN` before you start and again before you report done. **The second number must not exceed the first.** Finishing with more open gates than you started with means the task was not completed — it means work was moved onto a list instead of being done.

- Every item **you** opened during this task is yours to close before you report done.
- If you closed the one item I asked about and opened three doing it, **you have not delivered the fix, you have relocated it.** Go back and finish, or tell me plainly and immediately that you are handing me a net-negative result and why.
- **Report the ledger in your summary**, always, and report **both counts** in this form: `gating: N open -> M open, accepted: P -> Q (R pending)`, where R is the number of accepted entries carrying a `Trigger:`. Then one line per item in the delta naming its disposition — fixed, dropped as no-consequence, moved to accepted-limitations, asked, or filed under category 1-4. **Both numbers, every time.** A ledger that reports only the gate count makes moving an item to `accepted-limitations.md` look identical to fixing it, which turns the cheapest escape into the one that scores best. With both numbers on one line a migration shows up as one falling while the other rises, in the same breath — and it is then on me to say whether I accept it.
- The count may legitimately rise **only** for a category 1-3 item that I was told about **in this session** and chose to leave open. If I was in the session and you never asked me, filing was the wrong move and the ledger should show the ask instead.
- Anything that moved to `accepted-limitations.md` during this task gets **its own line in the summary**, not just a number — what it was, and why nothing is waiting on it. A move is a claim I should be able to challenge on the spot.

## 10. Above all

- **Do not break existing functionality.** Backward compatibility is a hard constraint unless I've explicitly approved otherwise.
- Don't let the fix quietly change unrelated behavior. The diff should do exactly one thing: kill this bug.

## 11. Commit & deploy (gated, not automatic)

1. For any fix touching **money, auth, persistent data, or production**: before commit, have a **fresh-context reviewer** (a subagent with no authoring context under Section 8's rules, or /code-review) adversarially attempt to refute the root-cause claim and find defects in the diff. The context that wrote a fix is systematically biased when reviewing it; cost is never a reason to skip this. **The review is a loop, not a report.** Its findings return to *you* and are dispositioned under Section 9's two questions before this gate closes — each one either fixed and re-verified, or dropped as no-consequence with the reason stated in your summary. **A finding is never filed as a gating item merely because a review produced it.** If one genuinely meets the four-category test, ask me about it now, in this session. **Re-run the review after any material fix** — a first-round finding list is not a result until a round comes back clean.
2. **Commit atomically:** the fix and its regression test together, with a message that states the **root cause** — not "fix bug." Never commit a red state or secrets.
3. Before deploy: **full suite + build + typecheck + lint green**, including the new regression test.
4. Confirm the **GATING list contains nothing that blocks correctness** in the target environment.
5. Summarize the **root cause, the fix, and its blast radius** — separating what was *demonstrated* (exercised by a test or run you executed) from what is *inferred* (reasoned but not exercised).
6. For anything irreversible or production-facing, get my **explicit go-ahead** and confirm the rollback path. For an incident, keep the mitigation in place until the fix is verified live, then remove it deliberately.

Only then deploy.
