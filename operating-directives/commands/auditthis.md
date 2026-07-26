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
- If a standard check can't run (missing deps, no test runner, credentials),
  say so in the coverage statement and log it to memory/gating.md.

## 4. Audit — scaled to blast radius

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

## 5. Verify findings before reporting

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

## 6. Report

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

## 7. Update session memory

- Append audit summary to memory-cowork.md: scope, severity counts,
  report path, and any gating items logged.
- New blockers discovered (unrunnable tests, missing deps/credentials) go
  to the project's persistent **memory/gating.md** — created on first item,
  updated in place so it survives session end and context compaction; never
  silently dropped.
- Close by offering routes, not actions: /fixthis per error finding,
  /simplify for the cleanup tier. Make no changes yourself.
