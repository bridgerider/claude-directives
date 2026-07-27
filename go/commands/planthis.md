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
- **General initiative:** Read the actual materials, data, and prior work — don't assume their contents. For finance, legal, tax, data, real-estate, or marketing work, follow the relevant `_context/domains.md` standards, and **don't assert a load-bearing fact you haven't sourced** — a plan resting on a misremembered rate, rule, or figure is built on sand. Where such a fact is unknown, that is a **gating item** (Section 8), not a number to guess.

## 4. Decompose (the mode-specific spine)

- **Software-implementation — vertical slices.** Break the work into the thinnest **end-to-end increments** that each deliver independently testable behavior. Order them by **dependency and risk** — most foundational and riskiest first. For each slice specify: its **acceptance criteria**, a **test-strategy sketch** (happy path *and* adversarial paths; property-based over example-based where the input domain is non-trivial), and the **interface/contract** at its boundary. This slice list *is* the build plan — write it so /codethis's per-slice loop can consume it directly, slice by slice.
- **General initiative — work breakdown.** Decompose into **phases → milestones → workstreams → concrete tasks.** Establish the **sequencing and critical path** (what blocks what), the **dependencies** (internal and external), and the **resources/owners** each workstream needs. Each milestone carries its own acceptance criteria and, where it matters, a rough estimate.

## 5. Design and alternatives considered (both modes)

- State the **chosen approach** and, briefly, the credible **alternatives you weighed and why you rejected them.** A plan that shows only the winner hides the decision — I review the road not taken too.
- *Software:* name the architecture — components, data flow, where new code integrates, and any schema or data-model change. *General:* name the mechanism — how the plan actually achieves the objective.
- Keep it scoped to the task. Don't design beyond what's asked, and don't smuggle unrelated rework into the plan without flagging it.

## 6. Pre-mortem and risk / blast-radius map (both modes)

- **Red-team the plan.** Assume the finished work has already failed, then enumerate the most likely causes: wrong assumptions, missing edge cases, unhandled failure modes, integration mismatches, dependency slips, misread requirements. **Fix the plan for the ones that matter before execution starts.**
- **Map blast radius.** Flag which slices or workstreams are **irreversible or touch money, auth, permissions, or persistent data.** In software mode these carry forward into /codethis's full-hardening tier — mark them so nothing high-stakes is planned as if it were throwaway.
- For each material risk, note its likelihood and impact and the **mitigation or contingency** the plan builds in.

## 7. Decisions requiring operator sign-off (both modes — the review gate)

This is the payoff of planning upfront: surface every choice I should own **before** execution, as an explicit question with a **recommended default.**

- Reserve sign-off for the **irreversible or high-blast-radius**: schema changes against live data, destructive operations, public/API contract changes, auth/security decisions, anything touching production — and, in general mode, any strategic commitment, spend, or externally-visible action.
- Proceed autonomously on **reversible, low-blast-radius** choices (and state the assumption inline in the plan). Don't manufacture questions for decisions that don't need me.

## 8. Open questions and gating (both modes)

- Maintain a **persistent gating list** in the project's `memory/gating.md` for anything blocking a complete or executable plan that you can't resolve now: unknowns, missing credentials or access, undecided requirements, unavailable services, unsourced load-bearing facts, upstream dependencies. Create it on the first item, update it in place — it must survive session end and context compaction.
- Never silently drop an open item. **An honestly-incomplete plan with its gaps named outranks a falsely-complete one.**

## 9. The plan artifact and handoff (mode-specific target)

- Write the plan to `<project-dir>/output/YYYY-MM-DD_plan_{slug}.md` (never overwrite — append `_v2`, `_v3`). Structure it for review, in the order built above: goal and success criteria, scope and non-goals, current-state evidence, the decomposition (slices or WBS), design and alternatives, pre-mortem and risk/blast-radius map, sign-off decisions, and open questions/gating.
- **Software mode:** when the plan becomes the living build spec, promote it to `memory/plan-{slug}.md` so it persists and evolves. The handoff to /codethis is **manual and deliberate** — paste the plan (or its path) as /codethis's argument when I give the go-ahead.
- **General mode:** the plan *is* the deliverable; hand it to execution, which may be manual or another directive.

## 10. Present for approval, then stop (both modes)

- **Present the plan and its sign-off decisions for review.** Incorporate my feedback and revise — planning is iterative, and a plan is done when I've **approved** it, not when it's first written.
- On explicit sign-off, **STOP.** This directive plans; it does not execute. Do not write code, run migrations, or ship — hand off to /codethis (software) or to execution (general).
- **Report honestly:** separate what the plan rests on that you **verified this session** (read the code, sourced the fact, ran the read-only check) from what remains **assumed or unconfirmed** — the same evidence-over-assertion bar applies to the plan's own foundations.
- Append a summary to `memory-cowork.md`: the task, the chosen mode, the plan's path, the open sign-off decisions, and any gating items logged.
