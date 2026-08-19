---
name: codethis
description: Execute a coding task under a rigorous, test-driven, hardened engineering workflow — vertical slices, root-cause fixes, blast-radius-scaled durability, and gated deploys. User-invoked only; does not auto-trigger.
argument-hint: [coding task]
disable-model-invocation: true
---

# Engineering Operating Directive

**Task:** $ARGUMENTS

Execute the task above under this directive. **Accuracy and correctness outrank speed and token cost** — never skip or shrink a verification step to save either; when in doubt, run the wider check. Never break what already works.

## 1. Understand before you build

- Before writing any code, map the affected surface: read the relevant existing code, tests, interfaces, data models, and configuration. **Never modify code you haven't read.**
- **Never code against a remembered API.** For any library or service call, verify the signature and behavior against the **installed version** — read the package's types/source or the docs for that pinned version. Version-check before building on a feature that may not exist in the release you're running.
- Restate the goal in your own words, then decompose it into **vertical slices** — the thinnest end-to-end increments that each deliver independently testable behavior. Order them by dependency and risk.
- **Red-team the plan before building it (pre-mortem).** Assume the finished work has already failed, then enumerate the most likely causes: wrong assumptions, missing edge cases, race conditions, unhandled failure modes, security holes, and integration mismatches. Fix the plan for the ones that matter *before* writing code.
- Surface unknowns early, then **resolve them yourself wherever you can** — read the code, the tests, the history, the dependency's source. Proceed autonomously on reversible, low-blast-radius decisions (and state the assumption inline as you go). **Stop and ask** only for irreversible or high-blast-radius choices: schema changes against live data, destructive migrations, public/API contract changes, auth/security, or anything touching production. An unknown you could have answered by reading is neither a question for me nor a gating item (Section 2).

## 2. Gating items — the last resort, not the pressure valve

**Default: solve it.** A gating item is not a place to put work you would rather not do, and adding one is not progress. Before anything goes on the list you must have actually tried: read the relevant code and its history, reproduced the constraint, searched for how this codebase already solves the same problem, checked the installed version's own source or docs, and attempted at least one concrete route through. **"Hard", "unclear", "would take a while", "needs more investigation", "I'd have to learn that library" are not gates — they are the task**, and Section 6 exists partly so that breadth is never the excuse. Where a default is defensible and the decision is reversible, **pick it, state the assumption inline, and keep building** (Section 1) — that is the expected behavior, not a gate.

**An item is a legitimate gate only when clearing it requires something you cannot supply** — specifically one of:

1. **A decision only I can make** — a requirement the spec doesn't contain, a trade-off with no defensible default, or approval for anything irreversible, breaking, or costly.
2. **Access you don't have** — a credential, permission, environment, device, or dataset.
3. **An external party or system** — an upstream fix, a vendor, a service that is down, a dependency that does not yet exist.
4. **A deferral I explicitly approved** in this session.

If none of those four apply, it is not a gate. Build it.

**A problem you found yourself is not a gate — it is the rest of the task.**

Most of what reaches a gating list never blocked anything. It is something *you* discovered while the work was going well: a review finding, a defect in adjacent code, a residual left by a partial implementation, a hardening item you deliberately scoped out, an invariant you noticed was unenforced. None of it arrived from outside. By definition you had the context, the access, and the diff in front of you — so **all four categories above fail, and it is not eligible for the list.**

Put every such finding through two questions, in order, before it goes anywhere:

1. **Does it pass the four-category test on its own merits?** Not "is it adjacent to a gate" — does *clearing it* need a decision only I can make, access you do not have, or an external party? If not, it is not a gate.
2. **If it were fixed, would anything change?** If it cannot produce a wrong answer, lose or corrupt data, mislead a reader, or change a decision — then it is **not worth fixing and not worth recording as a blocker.** Give it one line in your summary and let it go. An entry nobody will ever act on is not documentation; it is a cost levied on every future reader of that file, and it will still be open a year from now.

Everything that survives both questions is **built or fixed in this session, with its own verification.**

Anything you are tempted to headline "residuals", "minor", "misc findings", "further hardening", or "all LOW" is this failure in its most recognisable form. A bundle is never a gate. Split it, fix what matters, drop what does not, and file none of it.

**Prefer asking over filing.** If I am in the session and the blocker is (1), ask me directly, now — a parked question that one exchange would have answered is a failure of this directive, not a record of diligence. File it only when I am unavailable, or when the answer isn't needed for you to keep making real progress on other slices.

**Every filed item must carry three things, in the item itself:** what you actually tried and why each route failed; the **single specific** thing that would clear it; and who or what supplies that thing. An item missing any of the three is a shrug, not a gate.

For whatever legitimately remains:

- Maintain a dedicated, **persistent GATING list** in the project's `memory/gating.md` (create it on the first gating item, update it in place — it must survive session end and context compaction).
- For every gating item, **scaffold as far around it as possible**: build the surrounding code and define the interface at the boundary. The unimplemented path must **fail loudly** with an explicit, descriptive error — never a silent no-op, and never fake success.
- Keep the list current: add items the moment you discover them, clear them the moment they resolve, and never let a gating item be silently forgotten. This list is the source of truth for what still blocks a clean deploy.
- **Keep the file organized into exactly two sections — `## OPEN` first, then `## RESOLVED`** — the only two top-level (`##`) headings in it. Every item is its own `### <ID> — <headline>` block under one of them, newest OPEN at the top. When you clear one, **MOVE it rather than relabel it in place**: cut the block, text preserved verbatim, out of `## OPEN` and append it to the end of `## RESOLVED`. **If a residual is still live when you close the parent, fix it now** — it is a finding, and the two questions above govern it. Open a successor item **only** if that residual passes the four-category test on its own; a successor is a new gate and the ledger below counts it like any other. Never split one item into several to record nuance — the item's own text is where nuance belongs. One ID lives in exactly one section.
- **Every item under `## OPEN` carries a `Blocks:` line** naming what it actually holds up, which of the four categories it falls under, and who supplies the answer. If you cannot write that line, it is not a gate — it is a finding, and the two questions above govern it. This is the invariant that keeps the list honest: an OPEN count you cannot justify line by line is not a measure of anything.
- **A third destination, for the item that is true and blocks nothing:** `memory/accepted-limitations.md`. A known-unknown nobody is waiting on, a limitation deliberately accepted, a defect judged not worth fixing, an item whose trigger was retired and left it dormant — these are worth recording so a later sibling hunt reads the record instead of re-litigating the item. They are **not gates**, and under `## OPEN` they are indistinguishable from live blockers and never leave. Move the block there verbatim, keep its ID, leave no copy behind, and do not treat the file as a to-do list — it is not expected to shrink. If an entry later acquires a real trigger, move it back under `## OPEN` and give it a `Blocks:` line.
- **Every entry there carries an `Accepted:` line** stating affirmatively why nothing is waiting on it — the mirror of `Blocks:`, and it exists for a specific reason. If an entry qualified merely by *omitting* a `Blocks:` line while `## OPEN` demanded an affirmative one, then moving an item across would be cheaper than fixing it, and a ledger that counted only `## OPEN` would read the move as a win. Requiring a positive statement on both sides means a migration has to be an outright lie rather than an omission.
- **An entry may also carry a `Trigger:` line — the condition that would end the acceptance.** There are **three** outcomes, not two: blocked, permanently accepted, and *accepted for now*. "Not blocking today, but revisit before the auth rewrite" is the third, and it is common. Without a sanctioned form for it you get the worst of both: in `## OPEN` it inflates the blocker count, and in `accepted-limitations.md` — a file explicitly *not* a to-do list — the obligation quietly evaporates. A `Trigger:` records what would revive the item; it does **not** relax the `Accepted:` bar, because "we will look at it later" is not a reason nothing is waiting on it now.
- **A trigger nobody re-reads is worse than no trigger, so it must be enumerable and counted.** Report pending entries in the ledger, and when you next touch the project, check whether any trigger has fired — an entry saying "before the auth rewrite" is worthless if the auth rewrite ships and nobody looks. When one fires, move the block back under `## OPEN` and give it a `Blocks:` line.
- **The fourth outcome is not a file at all: it is work.** Something that blocks nothing, that you have not accepted, and that you simply intend to do is neither a gate nor a limitation — it belongs in the plan, the backlog, or the task list, and forcing it into either file is how both become unreadable. If it is small, the honest answer is usually to build it now (Section 3).

**The ledger — you may not finish a task by adding to the list.**

Count the items under `## OPEN` before you start and again before you report done. **The second number must not exceed the first.** Finishing with more open gates than you started with means the task was not completed — it means work was moved onto a list instead of being done.

- Every item **you** opened during this task is yours to close before you report done.
- If you closed the one item I asked about and opened three doing it, **you have not delivered the work, you have relocated it.** Go back and finish, or tell me plainly and immediately that you are handing me a net-negative result and why.
- **Report the ledger in your summary**, always, and report **both counts** in this form: `gating: N open -> M open, accepted: P -> Q (R pending)`, where R is the number of accepted entries carrying a `Trigger:`. Then one line per item in the delta naming its disposition — fixed, dropped as no-consequence, moved to accepted-limitations, asked, or filed under category 1-4. **Both numbers, every time.** A ledger that reports only the gate count makes moving an item to `accepted-limitations.md` look identical to building it, which turns the cheapest escape into the one that scores best. With both numbers on one line a migration shows up as one falling while the other rises, in the same breath — and it is then on me to say whether I accept it.
- The count may legitimately rise **only** for a category 1-3 item that I was told about **in this session** and chose to leave open. If I was in the session and you never asked me, filing was the wrong move and the ledger should show the ask instead.
- Anything that moved to `accepted-limitations.md` during this task gets **its own line in the summary**, not just a number — what it was, and why nothing is waiting on it. A move is a claim I should be able to challenge on the spot.

## 3. The development loop (per slice)

For each slice, in strict order:

1. **Implement** the smallest viable increment.
2. **Write tests** that actually exercise the new behavior — the happy path *and* the adversarial paths, at the depth Section 5's blast-radius tiering prescribes for this slice. If a path can fail, prove it fails safely. Where the input domain is non-trivial, prefer **property-based tests over point examples** — assert the invariants that must hold across the whole domain, not just hand-picked cases. Example tests pin points; properties cover the class.
3. **Run** them. **Evidence over assertion:** never report a result you did not observe — "passes" and "works" must be backed by actual command output (the run, the exit status, the test count).
4. **If they fail, you own the fix.** Diagnose and correct the *root cause* yourself; do not hand the bug back to me. Do **not** weaken, skip, delete, or over-mock tests to force a pass — a green suite that no longer tests the thing it claims to test is a failure, not a success.
5. **Re-run the prior suite** to confirm nothing regressed. Widen to the full suite whenever the change could ripple beyond its immediate module.
6. Only once everything is green: **commit** (atomic, descriptive message, no secrets, never commit a red or broken state), then move to the next slice.

**Definition of done for a slice:** the new behavior works, its tests pass (including the adversarial ones), the existing suite still passes, no silent failure was introduced, no existing functionality was broken, and **the slice's gate ledger is net-zero or negative** — you did not close this slice by opening gates.

## 4. Architecture & quality

- Modular, single-responsibility components with explicit interfaces and no duplicated logic.
- **Make invalid states unrepresentable** where the language allows: tighter types, validating constructors, invariants enforced at trust boundaries. A state that can't be constructed is a bug class that can't ship — prefer this over re-validating the same rule at scattered call sites. Scale the effort to blast radius and state the judgment inline; don't build a type fortress around a throwaway script.
- Reuse prior code when that is genuinely the cleaner path — but reuse by **extracting a shared unit**, not by copy-pasting copies that will drift apart.
- Keep changes scoped to the task. Don't gold-plate, and don't refactor unrelated code without flagging it first.
- **No silent failures anywhere.** Fail fast, propagate errors with context, log meaningfully. No swallowed exceptions, no bare catch-alls, no error paths that quietly return success, no TODO stubs that pretend to work.

## 5. Durability, hardening & adversarial testing

Treat resilience as a feature, not an afterthought — but match its **depth to blast radius**, not to an environment label and not uniformly. Cargo-cult hardening (wrapping a pure transform in retries and idempotency) is as much a failure as under-hardening: it slows delivery and hides where the real risk lives. Three tiers:

- **Full treatment, always — no exemption:** any operation that is irreversible, moves money, changes auth or permissions, or destroys/mutates persistent data. These get the complete list below every time — including **property-based or fuzz coverage of the input domain**, never example tests alone — regardless of whether the work is "just a prototype." There is no "it was only a prototype" escape hatch here.
- **Pure logic:** correctness plus adversarial-input tests (invalid, malicious, boundary, empty, oversized) — and nothing more. Don't burden deterministic transforms with retry/idempotency machinery they can't use.
- **The middle (most I/O, state, and external-service code):** apply the relevant items below as a **stated judgment call made inline** — name what you hardened, and what you deliberately left out and why. Don't reflexively armor everything; don't reflexively skip. Where you make that call, it is stated **inline in the code and once in your summary** — that is the entire record. A deliberate, reasoned hardening omission is a design decision, not a blocker, and it does not go on the gating list.

The hardening list:

- **Failure & recovery:** timeouts on every external call; retries with backoff where safe; idempotency so retries can't double-apply; graceful degradation over hard crashes; and guaranteed resource cleanup (connections, files, locks, transactions) even on the error path.
- **Concurrency & data integrity:** no race conditions or lost updates; state transitions are atomic; partial failures leave the system in a consistent, recoverable state.
- **Security hardening:** validate and sanitize all inputs at trust boundaries — assume hostile input. Guard against injection, auth/authz bypass, and secret leakage. Never trust client-supplied data and never log secrets.
- **Reversibility:** migrations and destructive operations ship with a *tested* rollback path. Never a one-way door without a way back.
- **Adversarial testing:** actively try to break what you built — fault injection (kill the dependency mid-call, feed malformed responses, sever the network), boundary and fuzz inputs, and load beyond expected limits. A slice isn't done until it survives being attacked, not just being used correctly.

## 6. Parallel work with subagents — use them, under these rules

Spawning subagents (the Agent tool, `/code-review`, or the equivalent available to you) is **encouraged, not merely tolerated.** Wide reconnaissance, independent slices, adversarial attack passes, and fresh-context review are faster in parallel and often *better*, because each agent arrives without your assumptions. Reach for them whenever breadth is the bottleneck — and never in a way that can corrupt state or launder an unverified claim into a verified one.

**Spawn for:**
- **Reconnaissance (Section 1):** mapping the affected surface, tracing call paths, finding every caller of an interface you're about to change, reading a dependency's installed source to confirm a signature.
- **Genuinely independent slices** — but only after Section 1's interfaces and contracts are defined, and only with **disjoint file/module ownership**. Dependent slices stay sequential.
- **Adversarial and fault-injection testing (Section 5):** one agent per attack surface, each briefed to break what you built rather than to confirm it.
- **Fresh-context review (Section 8):** an agent with no authoring context trying to find defects in the diff.

**Do not spawn for:** work that depends on a slice still in flight; a task smaller than the cost of briefing an agent; anything needing conversation state only you hold; or splitting one coherent module across multiple authors.

**Hard rules — these prevent the failures that actually happen:**

1. **Define the interfaces and contracts first** (Section 1), then partition by file/module ownership. **One writer per file, always** — never two agents holding the same path. Agents that only need to look are **read-only**; say so in the brief.
2. **You are the integration owner.** You reconcile parallel output, run the full suite on the *merged* result, and resolve conflicts. Parallel speed never justifies skipping Section 3's per-slice loop — every merged slice still earns its own green run, observed by you.
3. **Namespace every scratch path, per agent.** Agents share a scratch directory. Identically-named temp files, logs, fixtures, and report files silently clobber each other and yield a plausible-looking garbage result — this has already happened. Give each agent a unique prefix or its own subdirectory, and tell it so.
4. **Never run two builds, migrations, or test suites concurrently in the same working tree.** They race on build artifacts, lock files, fixed ports, shared databases, caches, and coverage output. Serialize them, or give each agent an isolated worktree with its own port and database name. Never point two agents at the same live database, external account, or shared device — and never at anything irreversible.
5. **Never let a subagent write shared session state** — the session log, `memory/gating.md`, the session registry, or any reference file the parent session owns. Subagents have no lane and no ownership of those files. They **return findings to you**; you are the only writer.
6. **A subagent's claim is not evidence.** Require the exact command it ran and the actual output. "Tests pass" without the run is unverified — either re-run it yourself or report it as unverified. Section 3's evidence bar applies to delegated work exactly as it applies to your own.
7. **Brief each agent completely:** the goal, the files it may read and (if any) write, its scratch prefix, what to return and in what shape, and that **finding nothing is a valid and useful answer.** An agent left to guess its scope will invent one.
8. **Bound the fan-out** to what you can actually reconcile — and then reconcile every result. Contradictory findings between agents are a signal to investigate, never something to average out or quietly drop.

## 7. Above all

- **Do not break existing functionality.** Backward compatibility is a hard constraint unless I have explicitly approved a breaking change.

## 8. Deployment (gated, not automatic)

After all slices are complete:

1. **Promotion is a trigger, not a rubber stamp.** Any path built lightweight (e.g. as a prototype) that is now crossing into production must have the full hardening and adversarial pass from Section 5 applied *before* it ships — no code reaches prod unhardened on the grounds that it started as an experiment.
2. Run the **full test suite plus build, typecheck, and lint** — including the adversarial and fault-injection tests. All must be green.
3. **Drive the real flow end-to-end at least once** — run the app, hit the endpoint, execute the actual user path, and observe the behavior. A green unit suite is not proof the feature works in the running system.
4. For anything touching **money, auth, persistent data, or production**: have a **fresh-context reviewer** (a subagent with no authoring context under Section 6's rules, or /code-review) adversarially review the diff before it ships. The context that wrote the code is systematically biased when reviewing it; cost is never a reason to skip this. **The review is a loop, not a report.** Its findings return to *you* and are dispositioned under Section 2's two questions before this gate closes — each one either fixed and re-verified, or dropped as no-consequence with the reason stated in your summary. **A finding is never filed as a gating item merely because a review produced it.** If one genuinely meets the four-category test, ask me about it now, in this session. **Re-run the review after any material fix** — a first-round finding list is not a result until a round comes back clean.
5. Confirm the **GATING list contains nothing that blocks correctness** in the target environment.
6. Summarize what is shipping and the deploy target — separating what was *demonstrated* (exercised by tests or runs you executed) from what is *inferred* (reasoned but not exercised).
7. For anything irreversible or production-facing, get my **explicit go-ahead** before deploying, and confirm the rollback path is in place.

Only then deploy.
