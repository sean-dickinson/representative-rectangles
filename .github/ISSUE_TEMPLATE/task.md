---
name: Task
about: A unit of work with acceptance criteria, implemented by an agent
title: "Task NN — "
labels: task
---

<!--
HOW TO WRITE THIS ISSUE

This issue body IS the spec. An agent reads it cold, with no memory of any
conversation, and implements from it. docs/AGENT-WORKFLOW.md describes the
process it follows. Issue #5 is a worked example.

The single rule that governs the whole thing:

  Say WHAT must be true and WHY. Do not say HOW to build it.

The agent's first step is to post a plan for approval, and that plan is where
class names, routes, method signatures, and the test list get decided — with
the real codebase in front of it. Anything you specify here that belongs in
the plan will be stale by the time the task is picked up.

Write nothing that is already in CLAUDE.md or docs/CONVENTIONS.md. The agent
reads both. Restating them here means two copies to keep in sync.

Write nothing point-in-time. No "we decided to switch from X", no "as
discussed", no reference to how things used to be. A fresh agent has no idea
what changed and the note will only confuse it.

Delete any section below that genuinely doesn't apply. Delete these comments
as you go, or leave them — they don't render.
-->

<!--
HEADER — one line. Branch name uses the TASK number from the title, not the
issue number (they don't match). "Depends on" lists issues that must be
merged first. Leave the "read these" sentence as-is.
-->
Branch `task-NN-short-slug`. Depends on #N. Blocks #N.

Read [CLAUDE.md](../blob/main/CLAUDE.md), [docs/CONVENTIONS.md](../blob/main/docs/CONVENTIONS.md), and [docs/AGENT-WORKFLOW.md](../blob/main/docs/AGENT-WORKFLOW.md) before starting. **Post a plan and wait for approval before writing code** — see the workflow doc.

## Goal

<!--
Two or three sentences. What is true after this task that wasn't before, in
terms the teacher would recognize — not in terms of files or classes.

If the task exists for a non-obvious reason (it settles a decision the rest
of the app inherits, its thresholds need tuning against real data, it's
deliberately separate from a neighbouring task), say so. That reason shapes
how an agent judges its own trade-offs.
-->

## Out of scope

<!--
The most valuable section. Name the things
an agent would plausibly build here, and say which task owns them instead:

  - Real dashboard content: assignment lists, rosters, counts (tasks 3 and 8)

"Don't stub it either" is worth saying for anything a helpful agent might
scaffold ahead of time.

Also list capability the app is never getting, if it's near this task —
student accounts, roles, multi-tenancy. CONVENTIONS.md covers the standing
ones; only repeat those close enough to this task to be tempting.
-->

## Constraints

<!--
Non-negotiable INPUTS to the plan, as distinct from acceptance criteria,
which are verifiable OUTCOMES. The test: could a reviewer check this by
looking at the running app? If yes it's an AC. If it only shapes how the
work is done, it's a constraint.

  Constraint: "Authentication is Rails' built-in generator, not a gem."
  AC:         "Nothing is reachable while signed out."

Constraints should be things that genuinely won't go stale — usually a house
rule from CLAUDE.md or CONVENTIONS.md applied to this task, or a design
decision the rest of the app depends on. If you're writing a constraint
because you have a hunch about implementation, don't; let the plan propose
it and push back then.

Delete this section if the task has none. Many won't.
-->

## Acceptance criteria

<!--
Numbered AC-1, AC-2, … The agent refines each into something concretely
checkable in its plan and cites these numbers; the reviewer verifies both
levels. Stable numbering matters — don't renumber after work starts.

Each one states an observable outcome. Bold lead-in, then a sentence or two.

  Good: "**AC-3 — Failed sign-in reveals nothing.** Wrong password and
         unknown email produce the same visible outcome: an error, no
         session, and no way to tell which happened."

  Bad:  "**AC-3** — SessionsController#create renders :new with
         status: :unprocessable_entity and a generic flash."

The bad one names the mechanism, so it goes stale and it forecloses a better
approach. The good one survives any implementation.

Cover the failure and edge paths, not just the happy path — an AC that only
describes success gets you a PR that only tests success.

Don't list the tests to write. CONVENTIONS.md already requires coverage of
every stated AC; the plan decides which layer each belongs at.
-->

**AC-1 — <short outcome>.** <What must be observably true.>

**AC-2 — <short outcome>.** <What must be observably true.>

## Deferred

<!--
Things this task knowingly leaves incomplete, and who picks them up. Use it
when a task would otherwise look half-finished to a reviewer, or when an
agent would "helpfully" finish something you don't want finished yet.

Say what to leave alone and why:

  **Password reset delivery.** SMTP isn't configured and the credentials are
  an open decision. Keep the generated code, leave it unconfigured, don't
  test it — task 4 wires it up.

Delete if nothing applies.
-->

## Not findings

<!--
Pre-empt the review round you can already predict. Anything an agent would
reasonably flag that is actually intentional here.

Be precise about the boundary — the useful version says what IS still a
finding:

  The empty dashboard is a placeholder by design — but a *missing* empty
  state is a finding (AC-8).

CONVENTIONS.md has the standing list. This section is only for what's
specific to this task. Delete if there's nothing.
-->
