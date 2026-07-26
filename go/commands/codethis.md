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
- Surface unknowns early. Proceed autonomously on reversible, low-blast-radius decisions (and state the assumption inline as you go). **Stop and ask** only for irreversible or high-blast-radius choices: schema changes against live data, destructive migrations, public/API contract changes, auth/security, or anything touching production.

## 2. Gating items

- Maintain a dedicated, **persistent GATING list** in the project's `memory/gating.md` (create it on the first gating item, update it in place — it must survive session end and context compaction): anything blocking full correctness that you cannot resolve right now (missing credentials, external approvals, undecided requirements, unavailable services, upstream dependencies).
- For every gating item, **scaffold as far around it as possible**: build the surrounding code and define the interface at the boundary. The unimplemented path must **fail loudly** with an explicit, descriptive error — never a silent no-op, and never fake success.
- Keep the list current: add items the moment you discover them, mark them resolved when cleared, and never let a gating item be silently forgotten. This list is the source of truth for what still blocks a clean deploy.

## 3. The development loop (per slice)

For each slice, in strict order:

1. **Implement** the smallest viable increment.
2. **Write tests** that actually exercise the new behavior — the happy path *and* the adversarial paths, at the depth Section 5's blast-radius tiering prescribes for this slice. If a path can fail, prove it fails safely. Where the input domain is non-trivial, prefer **property-based tests over point examples** — assert the invariants that must hold across the whole domain, not just hand-picked cases. Example tests pin points; properties cover the class.
3. **Run** them. **Evidence over assertion:** never report a result you did not observe — "passes" and "works" must be backed by actual command output (the run, the exit status, the test count).
4. **If they fail, you own the fix.** Diagnose and correct the *root cause* yourself; do not hand the bug back to me. Do **not** weaken, skip, delete, or over-mock tests to force a pass — a green suite that no longer tests the thing it claims to test is a failure, not a success.
5. **Re-run the prior suite** to confirm nothing regressed. Widen to the full suite whenever the change could ripple beyond its immediate module.
6. Only once everything is green: **commit** (atomic, descriptive message, no secrets, never commit a red or broken state), then move to the next slice.

**Definition of done for a slice:** the new behavior works, its tests pass (including the adversarial ones), the existing suite still passes, no silent failure was introduced, and no existing functionality was broken.

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
- **The middle (most I/O, state, and external-service code):** apply the relevant items below as a **stated judgment call made inline** — name what you hardened, and what you deliberately left out and why. Don't reflexively armor everything; don't reflexively skip.

The hardening list:

- **Failure & recovery:** timeouts on every external call; retries with backoff where safe; idempotency so retries can't double-apply; graceful degradation over hard crashes; and guaranteed resource cleanup (connections, files, locks, transactions) even on the error path.
- **Concurrency & data integrity:** no race conditions or lost updates; state transitions are atomic; partial failures leave the system in a consistent, recoverable state.
- **Security hardening:** validate and sanitize all inputs at trust boundaries — assume hostile input. Guard against injection, auth/authz bypass, and secret leakage. Never trust client-supplied data and never log secrets.
- **Reversibility:** migrations and destructive operations ship with a *tested* rollback path. Never a one-way door without a way back.
- **Adversarial testing:** actively try to break what you built — fault injection (kill the dependency mid-call, feed malformed responses, sever the network), boundary and fuzz inputs, and load beyond expected limits. A slice isn't done until it survives being attacked, not just being used correctly.

## 6. Parallelization with subagents

- Spin up dev subagents to parallelize **only where the work is genuinely independent**, and never at the cost of quality or coherence.
- Before parallelizing, **define the shared interfaces and contracts** so parallel work integrates cleanly. Partition by file/module ownership to avoid collisions. Keep dependent work sequential.
- Designate **one integration owner** to reconcile parallel outputs, run the full suite on the merged result, and resolve conflicts. Parallel speed never justifies skipping the per-slice loop in Section 3.

## 7. Above all

- **Do not break existing functionality.** Backward compatibility is a hard constraint unless I have explicitly approved a breaking change.

## 8. Deployment (gated, not automatic)

After all slices are complete:

1. **Promotion is a trigger, not a rubber stamp.** Any path built lightweight (e.g. as a prototype) that is now crossing into production must have the full hardening and adversarial pass from Section 5 applied *before* it ships — no code reaches prod unhardened on the grounds that it started as an experiment.
2. Run the **full test suite plus build, typecheck, and lint** — including the adversarial and fault-injection tests. All must be green.
3. **Drive the real flow end-to-end at least once** — run the app, hit the endpoint, execute the actual user path, and observe the behavior. A green unit suite is not proof the feature works in the running system.
4. For anything touching **money, auth, persistent data, or production**: have a **fresh-context reviewer** (a subagent with no authoring context, or /code-review) adversarially review the diff before it ships. The context that wrote the code is systematically biased when reviewing it; cost is never a reason to skip this.
5. Confirm the **GATING list contains nothing that blocks correctness** in the target environment.
6. Summarize what is shipping and the deploy target — separating what was *demonstrated* (exercised by tests or runs you executed) from what is *inferred* (reasoned but not exercised).
7. For anything irreversible or production-facing, get my **explicit go-ahead** before deploying, and confirm the rollback path is in place.

Only then deploy.
