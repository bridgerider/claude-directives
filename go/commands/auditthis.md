---
name: auditthis
description: Audit code against project standards and report verified findings — read-only. Blast-radius-tiered scrutiny, machine checks first, every finding backed by a concrete failure scenario. Changes no code. User-invoked only; does not auto-trigger.
argument-hint: [files, dirs, or module to audit — blank = default scope]
disable-model-invocation: true
---

# Code Audit Operating Directive

**Scope requested:** $ARGUMENTS

Audit the code above (or the default scope below) under this directive.
**Accuracy and correctness outrank speed and token cost** — never skip or
shrink a verification step to save either; when in doubt, run the wider check.

**This command is read-only.** It produces findings, not fixes. Change no
source files. Route fixes to /fixthis (bugs) or /simplify (cleanups) afterward.

## 1. Determine scope

- If $ARGUMENTS names files/dirs/a module: audit exactly that, plus its
  direct dependents (callers/importers) — a defect's blast radius includes
  the code that trusts it.
- Else, if git repo with uncommitted changes: audit changed files + their
  direct dependents (default). Offer the full-codebase option.
- Else (no git repo, or clean tree): audit all source files in <project-dir>.
- State the chosen scope explicitly. **No silent caps**: anything skipped
  (generated code, vendored deps, files too large to read fully) is named
  in the report's coverage statement, never silently dropped.

## 2. Load standards

Read project-specific code standards from:
  - <project-dir>/CLAUDE.md
  - _context/domains.md (relevant domain section)
  - _context/tech-stack.md
  - Any domain context folders referenced in CLAUDE.md (e.g. _context-<domain>/)
  - Any CLAUDE.md files found in the directories containing the files being
    audited (e.g., if auditing scripts/b03_seo/run.py, also read
    scripts/b03_seo/CLAUDE.md)
  - <project-dir>/memory/gating.md if it exists — known gaps are context,
    not fresh findings; don't re-report them as discoveries
  - Prior audit reports in output/ (*_code-audit*.md) — flag repeat findings
    as REPEAT, don't present them as new

## 3. Machine checks first

Before hand-auditing, run whatever the project already has configured —
linter, typechecker, test suite, build — in **read-only** fashion (no fixes,
no config changes). Don't hand-audit what a tool proves in seconds.
- Tool output is evidence: findings confirmed by a failing tool run enter
  the report as CONFIRMED with the output quoted.
- **Try before you log it.** "The command wasn't on PATH" is not a
  coverage gap, it's a wrong invocation. Within read-only means: find the
  project's actual test/lint/build command (CLAUDE.md, README, CI config,
  package.json scripts, Makefile, pyproject/tox), run it from the correct
  working directory, and use the toolchain or venv the project already
  has. Most "can't run" is the wrong incantation, not a missing capability.
- Only when that genuinely fails is it a coverage gap: name it in the
  coverage statement, and log it to memory/gating.md only if clearing it
  needs something you cannot supply here — a credential, an environment,
  or an install this read-only audit is not permitted to perform (§8).
  **Never** install, change config, or edit source to make a check run.

## 4. Parallel auditing with subagents — use them, under these rules

This directive is **read-only**, so it is the safest possible place to
fan out: no agent writes source, so agents cannot collide on it. Breadth
is the only real constraint on an audit's quality, and subagents are how
you buy breadth. Spawning them (the Agent tool, /code-review, or the
equivalent available to you) is **encouraged, not merely tolerated.**

**Spawn for:**
- **Coverage** — one agent per subsystem, module, or blast-radius tier
  (§5), so the deep tier gets real depth instead of whatever attention
  is left after skimming everything else.
- **The sibling hunt (§5)** — one agent per confirmed defect pattern,
  searching the whole codebase. This is the highest-value fan-out here:
  it is what turns a specimen into the pack.
- **Refutation (§6)** — a fresh-context agent per error-severity finding,
  briefed to *break* it. The context that found a defect is biased
  toward believing it.
- **Convention loading (§2)** — reading scattered CLAUDE.md files, prior
  audit reports, and domain context in parallel.

**Do not spawn for:** the severity calls, the coverage statement, or the
report itself — those are one author's job, and a report assembled from
agents that never spoke to each other is a list, not an audit.

**Hard rules:**

1. **Every agent is READ-ONLY on source. No exceptions.** This command
   changes no code (see the header). A throwaway repro is allowed exactly
   as §6 allows it — outside the source tree, discarded after — and each
   agent gets its own scratch prefix so two of them never overwrite one
   another's repro or output file.
2. **A subagent's claim is not evidence, and this directive already has
   the vocabulary for that:** a finding may enter the report as
   **CONFIRMED** only if the agent returns the actual run, the failing
   tool output, or the traced end-to-end path. Anything else is
   **PLAUSIBLE** at best, no matter how confident the agent sounds.
   Re-run it yourself to promote it.
3. **Require file:line and the concrete failure scenario** from every
   agent — the same bar §6 sets for you. An agent finding without them
   doesn't go in the report; it goes back for evidence or gets dropped.
4. **Never let a subagent write shared state** — the report,
   memory-cowork.md, memory/gating.md, or any project file. Agents
   **return findings to you**; you are the sole writer.
5. **Deduplicate before reporting.** Overlapping scopes mean two agents
   will find the same defect; report it once. And note that N agents
   flagging one line is **one finding**, not corroboration — agreement
   between agents reading the same code is not independent evidence.
6. **Brief each agent completely:** its scope, the standards from §2 it
   must audit against, its scratch prefix, what to return and in what
   shape, and that **finding nothing is a valid and useful answer.** An
   agent that must justify its existence will manufacture findings, and
   §6 exists to catch exactly that.
7. **Bound the fan-out** to what you can actually reconcile, and
   reconcile every result. Contradictions between agents are a signal to
   look yourself, never something to average out.

## 5. Audit — scaled to blast radius

Scrutiny depth follows blast radius, not file order:

- **Deepest — always:** paths that are irreversible, move money, change auth
  or permissions, or mutate/destroy persistent data. For these also check
  test depth: property/fuzz coverage expected, never example tests alone;
  missing coverage there is a finding (error), not a footnote.
- **Pure logic:** correctness, boundary and adversarial inputs (invalid,
  empty, oversized, malicious) — and nothing more.
- **The middle (I/O, state, external services):** judgment stated inline —
  name what you scrutinized hardest and what you deliberately skimmed.

Check for:
  - logic bugs, unhandled edge cases, broken invariants
  - **silent failures**: swallowed exceptions, bare catch-alls, error paths
    that return success, TODO stubs that pretend to work
  - concurrency: races, lost updates, non-atomic state transitions
  - resource leaks: connections/files/locks/transactions on error paths
  - external calls: missing timeouts, unsafe retries, non-idempotent
    operations that a retry would double-apply
  - security: hardcoded secrets, secrets in logs, injection vectors,
    unvalidated input at trust boundaries, auth/authz bypass
  - test-coverage gaps at the depth the tier above prescribes
  - unclear naming, redundancy, modularity violations (suggestions tier)
  - violations of the project conventions loaded in step 2

**Hunt for siblings:** every confirmed defect pattern gets a codebase-wide
search for the same pattern — bugs of a kind travel in packs. Report the
pack, not the specimen.

## 6. Verify findings before reporting

**Evidence over assertion — a finding you can't defend is noise, not signal.**
- Every error-severity finding must state its **concrete failure scenario**:
  the specific inputs/state that trigger it and the wrong behavior that
  results. "This looks unsafe" is not a finding.
- Read the full calling context before flagging — don't report a missing
  check the caller already guarantees, or a "bug" the type system prevents.
- **Verify against the installed version, not a remembered API.** Before
  flagging a library/service call as misused, confirm the signature and
  behavior in the *pinned version actually present* (read the package's
  types/source, or the docs for that release). A "bug" that's correct for the
  installed version — or a real defect you'd miss by recalling a different
  one — is a false finding either way.
- **Attempt to refute each error-severity finding** before it enters the
  report. Where cheap, demonstrate it (run the snippet, trace the failing
  path, write a throwaway repro — discard it after; this command stays
  read-only on source). Label every finding:
  - **CONFIRMED** — demonstrated by a run, failing tool output, or a traced
    end-to-end path you exhibited
  - **PLAUSIBLE** — reasoned but not exercised; say what would confirm it
- Drop or downgrade anything that survives neither. Three defensible errors
  outrank thirty maybes.
- **Then apply the consequence test — CONFIRMED is not the same as worth
  reporting.** For each surviving finding ask: if this were fixed, would
  anything change? If it cannot produce a wrong answer, lose or corrupt
  data, mislead a reader, or change a decision, it is not an error and
  usually not a finding at all — say it in one line of the coverage
  statement and let it go. A confirmed defect with no consequence still
  costs every future reader their attention, and downstream it is exactly
  the finding that gets parked on a gating list and never closed. Never
  bundle several of these into one "residuals" or "all LOW" entry; that
  bundle is the symptom, not a finding.

## 7. Report

- Categorize by severity: **error / warning / suggestion**, most severe
  first. Counts up front. For each finding: file:line, issue, concrete
  failure scenario, recommended fix, CONFIRMED/PLAUSIBLE (and REPEAT where
  applicable).
- **Coverage statement**: what was audited, what was excluded and why,
  which machine checks ran (with results) and which couldn't.
- Separate what was **demonstrated** (exercised by a run/tool you executed)
  from what is **inferred** (reasoned but not exercised).
- Suggestions only when concretely actionable — never bury errors under
  style notes.
- Save to <project-dir>/output/YYYY-MM-DD_code-audit.md (append scope slug
  when partial, e.g. 2026-07-13_code-audit_b03-seo.md). Never overwrite an
  existing report — append _v2, _v3.

## 8. Update session memory

- Append audit summary to memory-cowork.md: scope, severity counts,
  report path, and any gating items logged.
- **Audit findings are not gating items.** A defect you found belongs in
  the report and routes to /fixthis; parking it in gating.md hides it from
  the person who asked for the audit. Only a **blocker to auditing** goes
  there — and only after §3's try-before-you-log step, and only when
  clearing it needs something you cannot supply: a credential, an
  environment, an install this read-only command may not perform, or a
  decision only I can make. "Would have taken a while to read" is not a
  blocker; it is a coverage-statement entry at worst.
- Legitimate blockers go to the project's persistent **memory/gating.md**
  — created on first item, updated in place so it survives session end and
  context compaction; never silently dropped. Record what you tried, the
  one thing that would clear it, and who supplies that thing.
- **Keep that file in exactly two sections — `## OPEN` first, then
  `## RESOLVED`** — the only two top-level (`##`) headings in it. Each item
  is its own `### <ID> — <headline>` block under one of them, newest OPEN
  at the top. When one clears, **MOVE it rather than relabel it**: cut the
  block, text preserved verbatim, from `## OPEN` to the end of
  `## RESOLVED`. A still-live residual becomes its **own new** OPEN item
  only if it is itself a blocker to auditing by the test above; otherwise
  it is a finding and belongs in the report. Never split one item into
  several to record nuance — the item's own text is where nuance belongs.
  One ID lives in exactly one section.
- **Every item under `## OPEN` carries a `Blocks:` line** naming what it
  holds up and who supplies the answer. If you cannot write that line it is
  not a gate. An item that is true but blocks nothing — a known-unknown, an
  accepted limitation, a defect judged not worth fixing, one whose trigger
  was retired — goes to **`memory/accepted-limitations.md`**, verbatim and
  keeping its ID, so a later sibling hunt reads the record instead of
  re-litigating it. Under `## OPEN` it would be indistinguishable from a
  live blocker and would never leave.
- **Every entry there carries an `Accepted:` line** saying affirmatively why
  nothing is waiting on it — the mirror of `Blocks:`. Without it an entry would
  qualify by mere omission, and moving an item across would be cheaper than
  resolving it. Report both counts when you report gating items at all:
  `gating: N open, accepted: P` — one number alone hides a migration.
- **An entry there may carry a `Trigger:`** — the condition that would end
  the acceptance. There are three outcomes, not two: blocked, permanently
  accepted, and *accepted for now*. A `Trigger:` does not relax the `Accepted:`
  bar; it records what would revive the item. Count them in the ledger
  (`accepted: P (R pending)`) — a trigger nobody re-reads is worse than none.
- **A fourth outcome is not a file at all: it is work.** Something that blocks
  nothing and that you simply intend to do belongs in the plan or backlog.
  Forcing it into either file is how both become unreadable.
- Close by offering routes, not actions: /fixthis per error finding,
  /simplify for the cleanup tier. Make no changes yourself.
