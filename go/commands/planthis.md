---
name: planthis
description: Build a world-class, evidence-grounded plan before any execution — frame the goal, ground the plan in what is actually true, decompose it, red-team it, and surface every decision for review. Produces a reviewable plan, not code. Two modes — software-implementation (precursor to /codethis) and general initiative. User-invoked only; does not auto-trigger.
argument-hint: [task or project to plan]
disable-model-invocation: true
---

# Planning Operating Directive

**Task:** $ARGUMENTS

Plan the task above under this directive. **Accuracy and completeness outrank speed and token cost** — never skip or shrink a grounding or red-team step to save either; when in doubt, dig into the real state before committing a line of the plan.

**Cardinal rule — a plan is a hypothesis grounded in evidence, never in assumption, and I approve it before any execution begins.** This directive *plans and nothing else*: it writes no code, runs no migrations, ships nothing. Its output is a plan I can review upfront so the execution that follows runs smoothly.

## 1. Classify the work and set the mode

Two modes — pick one and state it explicitly:

- **Software-implementation** — the plan is a build spec, the precursor to /codethis. Choose this when the task is to build or change software in a codebase.
- **General initiative** — any non-coding project: research, analysis, a written deliverable, a campaign, an operational or structural change. Choose this otherwise.

Autodetect from the task: is the primary deliverable running software in a repo, or something else? When a task has both (e.g. an analysis that needs a throwaway script), pick the mode of the **primary deliverable** and note the secondary. **Stop and ask** only when the mode is genuinely ambiguous in a way that changes the plan's shape; otherwise state the reasonable read and proceed.

## 2. Frame the goal (both modes)

- **Restate the goal** in my own words and confirm you're planning what was actually asked — not an adjacent problem.
- **Define done:** the success / acceptance criteria a complete result must meet — measurable wherever possible. A goal you can't test against isn't a goal yet.
- **Fix the scope — name the non-goals explicitly.** An out-of-scope list is the single best defense against gold-plating and scope creep. State what this plan deliberately does *not* cover.
- **Capture the constraints:** deadline, budget, resources, non-negotiable requirements, and dependencies on other people or systems.

## 3. Ground the plan in evidence (mode-specific sources)

A plan built on assumptions is a guess with formatting. Ground every load-bearing element in something real before it enters the plan.

- **Software-implementation:** Read the actual code, tests, interfaces, data models, and configuration the task touches — **never plan against remembered structure.** Map the affected surface *and its dependents* (the code that will trust your change). **Verify every library/service API against the installed version, not memory** — a plan that assumes a feature the pinned release doesn't have fails at build time. Where useful, run **read-only** machine checks (existing tests, typecheck, lint, build) to establish the true current state, changing nothing.
- **General initiative:** Read the actual materials, data, and prior work — don't assume their contents. For finance, legal, tax, data, real-estate, or marketing work, follow the domain standards the workspace provides (a domains file that `CLAUDE.md` points to, if one exists — read the workspace-root copy, never a project-local one; this plan is complete without it), and **don't assert a load-bearing fact you haven't sourced** — a plan resting on a misremembered rate, rule, or figure is built on sand. Where such a fact is unknown, **go and source it** — the file, the statement, the filing, the primary reference — and parallelize that hunt (Section 4). It becomes a **gating item** (Section 9) only when the source genuinely isn't reachable from here; never a number to guess, and never a gate you file because looking would have taken effort.

## 4. Parallel grounding with subagents — use them, under these rules

Grounding (Section 3) is where planning is slowest and where breadth matters most, and it is **entirely read-only** — which makes it the ideal thing to parallelize. Spawning subagents (the Agent tool or the equivalent available to you) is **encouraged, not merely tolerated**: several agents reading different parts of the real state at once beats one agent reading serially, and each arrives without your assumptions.

**Spawn for:** mapping an unfamiliar surface and its dependents; reading the actual code, tests, interfaces, and configuration a task touches; verifying a library or service API against the **installed** version; sourcing the load-bearing facts, filings, rates, or prior work a general-mode plan rests on; surveying how this problem was solved before, here or elsewhere; and a **fresh-context red-team** of the draft plan (Section 7) by an agent that did not write it.

**Do not spawn for:** the framing, the decomposition, the design call, or the plan artifact itself — those are one author's job, and a plan assembled from agents that never spoke to each other is incoherent. Nor for anything smaller than the cost of briefing an agent.

**Hard rules:**

1. **Every agent is READ-ONLY. No exceptions.** This directive plans and nothing else (see the cardinal rule): no agent writes code, edits files, runs migrations, mutates state, or touches anything live. Read-only machine checks (existing tests, typecheck, lint, build) are permitted exactly as Section 3 permits them — changing nothing — and each agent that runs one gets its own workspace and its own scratch prefix so two of them never collide.
2. **Never let a subagent write shared session state** — the session log, `memory/gating.md`, the plan artifact, or any reference file. Agents **return findings to you**; you are the sole writer of every file this directive produces.
3. **A subagent's claim is not evidence, and evidence is the whole point here.** Require each agent to return its **sources** — the file and line, the command and its actual output, the document and page — not a summary you cannot trace. An unsourced agent finding is an assumption wearing a costume, and Section 3 forbids building the plan on it. If you can't trace it, it goes in the plan as unconfirmed or not at all.
4. **Brief each agent completely:** what to ground, which sources are in scope, its scratch prefix, what to return and in what shape, and that **"the source doesn't exist" or "I found nothing" is a valid and useful answer** — inventing a plausible fact is the failure mode that matters most in a planning pass.
5. **Bound the fan-out** to what you can actually reconcile, and reconcile every result. Where two agents disagree about the current state, that contradiction is itself a finding: resolve it by going to the source yourself, never by averaging or by picking the more convenient answer.

## 5. Decompose (the mode-specific spine)

- **Software-implementation — vertical slices.** Break the work into the thinnest **end-to-end increments** that each deliver independently testable behavior. Order them by **dependency and risk** — most foundational and riskiest first. For each slice specify: its **acceptance criteria**, a **test-strategy sketch** (happy path *and* adversarial paths; property-based over example-based where the input domain is non-trivial), the **interface/contract** at its boundary, and the **module it lands in** (a file path). A slice that would carry an existing file past the hard ceiling — 800 lines, source or test; 500 soft; function 50 statements; complexity 10 — names the split as its own prior slice, so /codethis's split-before-extend rule is planned here rather than discovered mid-build. This slice list *is* the build plan — write it so /codethis's per-slice loop can consume it directly, slice by slice.
- **General initiative — work breakdown.** Decompose into **phases → milestones → workstreams → concrete tasks.** Establish the **sequencing and critical path** (what blocks what), the **dependencies** (internal and external), and the **resources/owners** each workstream needs. Each milestone carries its own acceptance criteria and, where it matters, a rough estimate.

## 6. Design and alternatives considered (both modes)

- State the **chosen approach** and, briefly, the credible **alternatives you weighed and why you rejected them.** A plan that shows only the winner hides the decision — I review the road not taken too.
- *Software:* name the architecture — components, data flow, where new code integrates, and any schema or data-model change. *General:* name the mechanism — how the plan actually achieves the objective.
- Keep it scoped to the task. Don't design beyond what's asked, and don't smuggle unrelated rework into the plan without flagging it.

## 7. Pre-mortem and risk / blast-radius map (both modes)

- **Red-team the plan.** Assume the finished work has already failed, then enumerate the most likely causes: wrong assumptions, missing edge cases, unhandled failure modes, integration mismatches, dependency slips, misread requirements. **Fix the plan for the ones that matter before execution starts.**
- **Map blast radius.** Flag which slices or workstreams are **irreversible or touch money, auth, permissions, or persistent data.** In software mode these carry forward into /codethis's full-hardening tier — mark them so nothing high-stakes is planned as if it were throwaway.
- For each material risk, note its likelihood and impact and the **mitigation or contingency** the plan builds in.

## 8. Decisions requiring operator sign-off (both modes — the review gate)

This is the payoff of planning upfront: surface every choice I should own **before** execution, as an explicit question with a **recommended default.**

- Reserve sign-off for the **irreversible or high-blast-radius**: schema changes against live data, destructive operations, public/API contract changes, auth/security decisions, anything touching production — and, in general mode, any strategic commitment, spend, or externally-visible action.
- Proceed autonomously on **reversible, low-blast-radius** choices (and state the assumption inline in the plan). Don't manufacture questions for decisions that don't need me.

## 9. Open questions and gating (both modes) — the last resort, not the pressure valve

**Default: resolve it.** A gating item is not a place to put grounding you would rather not do, and adding one is not planning. Before anything goes on the list you must have actually tried: read the code, the materials, and the prior work; searched for the source; checked the installed version; and used Section 4 to parallelize the hunt. **"Unknown", "unclear", "would take a while to source", "I'd have to read that subsystem" are not gates — they are Section 3**, and an unknown you could have answered by reading is neither a question for me nor a gating item.

**An item is a legitimate gate only when resolving it requires something you cannot supply** — specifically one of:

1. **A decision only I can make** — a requirement, strategy, or trade-off with no defensible default. Note that most of these belong in Section 8 as a sign-off decision **with a recommended default**, not parked here as an unknown: a decision I can make in one line is not a blocker.
2. **Access you don't have** — a credential, permission, environment, dataset, or document.
3. **An external party or system** — a person who must answer, a vendor, a service that is down, an upstream dependency.
4. **A deferral I explicitly approved** in this session.

If none of those four apply, it is not a gate. Go and ground it.

**Prefer asking over filing.** If I am in the session and the blocker is (1), put it to me directly — as a Section 8 decision with your recommendation — instead of parking it. A plan handed back with a parked question that one exchange would have answered is a worse plan, not a more honest one.

**Every filed item must carry three things:** what you actually tried and why each route failed; the **single specific** thing that would resolve it; and who or what supplies that thing. An item missing any of the three is a shrug, not a gate — and in a plan it is worse than that, because it reads as diligence while hiding that nobody looked.

For whatever legitimately remains:

- Maintain a **persistent gating list** in the project's `memory/gating.md`. Create it on the first item, update it in place — it must survive session end and context compaction.
- **Every item under `## OPEN` carries a `Blocks:` line** naming what it holds up, which of the four categories it falls under, and who supplies the answer. If you cannot write that line it is not a gate. This is not optional formatting: run the repo's gating-file linter if it ships one, and check the line by hand if it does not — an item without it is a defect either way, and /codethis and /fixthis apply the same rule to the same file.
- **An item that is true but blocks nothing goes to `memory/accepted-limitations.md`**, verbatim and keeping its ID — a known-unknown, a deliberately accepted limitation, a defect judged not worth fixing, one whose trigger was retired. Under `## OPEN` it would be indistinguishable from a live blocker and would never leave. **Every entry there carries an `Accepted:` line** stating affirmatively why nothing is waiting on it, and may carry a `Trigger:` naming the condition that would revive it.
- **Report the ledger**: `gating: N open -> M open, accepted: P -> Q (R pending)`, both numbers every time, then one line per item in the delta naming its disposition. Reporting only the gate count makes moving an item to accepted-limitations look identical to resolving it.
- **Keep the file organized into exactly two sections — `## OPEN` first, then `## RESOLVED`** — the only two top-level (`##`) headings in it. Every item is its own `### <ID> — <headline>` block under one of them, newest OPEN at the top. When you clear one, **MOVE it rather than relabel it in place**: cut the block, text preserved verbatim, out of `## OPEN` and append it to the end of `## RESOLVED`; capture any still-live successor as its **own new** OPEN item instead of keeping the parent open. One ID lives in exactly one section.
- Never silently drop an open item. **An honestly-incomplete plan with its gaps named outranks a falsely-complete one** — but a plan whose gaps are all things you simply didn't go and look at is neither.

## 10. The plan artifact and handoff (mode-specific target)

- Write the plan to `<project-dir>/output/YYYY-MM-DD_plan_{slug}.md` (never overwrite — append `_v2`, `_v3`). **If you are a LANE session**, drafts go to `output/drafts/{lane-slug}/` and the final plan takes the time in its prefix — `YYYY-MM-DD_HHMM_plan_{slug}.md` — so concurrent lanes cannot collide on one filename. Structure it for review, in the order built above: goal and success criteria, scope and non-goals, current-state evidence, the decomposition (slices or WBS), design and alternatives, pre-mortem and risk/blast-radius map, sign-off decisions, and open questions/gating.
- **Software mode:** when the plan becomes the living build spec, promote it to `memory/plan-{slug}.md` so it persists and evolves. The handoff to /codethis is **manual and deliberate** — paste the plan (or its path) as /codethis's argument when I give the go-ahead.
- **General mode:** the plan *is* the deliverable; hand it to execution, which may be manual or another directive.

## 11. Present for approval, then stop (both modes)

- **Present the plan and its sign-off decisions for review.** Incorporate my feedback and revise — planning is iterative, and a plan is done when I've **approved** it, not when it's first written.
- On explicit sign-off, **STOP.** This directive plans; it does not execute. Do not write code, run migrations, or ship — hand off to /codethis (software) or to execution (general).
- Where the harness offers a native plan mode, the two compose rather than compete: plan mode is the interactive, in-session gate, while this directive's output is the **durable, reviewable artifact** that survives the session and feeds /codethis. Use plan mode freely while drafting; the artifact in Section 10 is still what gets signed off.
- **Report honestly:** separate what the plan rests on that you **verified this session** (read the code, sourced the fact, ran the read-only check) from what remains **assumed or unconfirmed** — the same evidence-over-assertion bar applies to the plan's own foundations.
- Append a summary — the task, the chosen mode, the plan's path, the open sign-off decisions, and any gating items logged — to **your session's log target, which is not always `memory-cowork.md`**. If the workspace runs a session registry, check `<project-dir>/.sessions/active.md` first: if you are registered as a LANE session, write to your own `.sessions/lane-{slug}.md` and never touch `memory-cowork.md`; if the registry lists a LIVE session that is not you — its `seen`, or `started` on an older line, under ~12 h — stop and ask. A STALE line is NOT yours to remove and NOT yours to write past: that decision belongs to the session-start protocol and to the user, because an idle-but-alive session is indistinguishable from a dead one by age alone. Report the stale line and write to your lane (or ask) — never take the log on the strength of a timestamp. No registry, or an empty one, means you are the only session: write the log normally. A lane that writes the shared log is the exact corruption the lane system exists to prevent.
