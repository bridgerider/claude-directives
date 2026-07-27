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
- **Confirm the expected behavior is actually specified** (docs, tests, contract, or my statement) — not assumed. Bug reports can be wrong; if "expected" is itself uncertain, that's a gating question, not a premise.
- **If the bug is intermittent**, establish a baseline reproduction rate first: run the reproduction N times and record failures/N. Without a baseline you cannot distinguish "fixed" from "got lucky."
- If you can't reproduce it, do not guess-fix. Gather signal (logs, stack traces, the failing input, environment differences), state what you'd need to reproduce it, and treat "not reproducible" as a **gating item** (Section 8), not a green light to start changing code.

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
- **One hypothesis at a time — and revert every failed probe before trying the next.** A working tree polluted by dead attempts makes the eventual fix unattributable and leaks junk into the diff. If two consecutive hypotheses fail, stop patching: re-read the full code path from scratch, list every assumption you haven't verified, and check them cheapest-first.
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
- **Hunt for siblings.** Search the codebase for the same bug pattern elsewhere — bugs of a kind travel in packs, and fixing one while ignoring its twins is half a job. Log any you find but won't fix now as gating items.

## 8. Gating items

- Maintain a dedicated, **persistent GATING list** in the project's `memory/gating.md` (create it on the first gating item, update it in place — it must survive session end and context compaction): anything blocking full resolution that you can't clear now (can't reproduce, needs data/access/credentials, needs a decision, deferred sibling bugs, upstream dependency).
- Scaffold around each where possible, make the unresolved path **fail loudly** (never a silent no-op or fake success), and keep the list current — added on discovery, cleared when resolved, never silently forgotten. This is the source of truth for what still blocks a clean fix.
- **Keep the file organized into exactly two sections — `## OPEN` first, then `## RESOLVED`** — so a reader sees live blockers up top and closed ones never clutter them. Every item is its own `### <ID> — <headline>` block filed under one of those two headings; those are the only two top-level (`##`) sections in the file.
  - **New item** → add it under `## OPEN`, newest at the top of the section.
  - **When you clear one, MOVE it, don't relabel it in place:** cut the whole block — text preserved verbatim — out of `## OPEN` and append it to the end of `## RESOLVED`. A gate counts as closed (fixed / done / won't-fix / superseded) even if it carries documented residuals; capture any still-live successor as its **own new** item under `## OPEN`, never by keeping the parent open.
  - **Still actionable** (needs a decision, a build, an operator go/no-go, or is parked/deferred) → it stays under `## OPEN`.
  - One ID lives in exactly one section — never duplicated across both, never left at `##`/top level outside the two sections.

## 9. Above all

- **Do not break existing functionality.** Backward compatibility is a hard constraint unless I've explicitly approved otherwise.
- Don't let the fix quietly change unrelated behavior. The diff should do exactly one thing: kill this bug.

## 10. Commit & deploy (gated, not automatic)

1. For any fix touching **money, auth, persistent data, or production**: before commit, have a **fresh-context reviewer** (a subagent with no authoring context, or /code-review) adversarially attempt to refute the root-cause claim and find defects in the diff. The context that wrote a fix is systematically biased when reviewing it; cost is never a reason to skip this.
2. **Commit atomically:** the fix and its regression test together, with a message that states the **root cause** — not "fix bug." Never commit a red state or secrets.
3. Before deploy: **full suite + build + typecheck + lint green**, including the new regression test.
4. Confirm the **GATING list contains nothing that blocks correctness** in the target environment.
5. Summarize the **root cause, the fix, and its blast radius** — separating what was *demonstrated* (exercised by a test or run you executed) from what is *inferred* (reasoned but not exercised).
6. For anything irreversible or production-facing, get my **explicit go-ahead** and confirm the rollback path. For an incident, keep the mitigation in place until the fix is verified live, then remove it deliberately.

Only then deploy.
