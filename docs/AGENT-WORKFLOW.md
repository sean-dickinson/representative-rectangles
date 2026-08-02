# Agent workflow

How work moves from an issue to a merged PR. [CLAUDE.md](../CLAUDE.md) covers
*what* we're building, [docs/CONVENTIONS.md](CONVENTIONS.md) covers *how code
should look*, and each task issue covers *what that task requires*. This file
covers the process connecting them — it is the contract between the
implementer and the reviewer, and both read it.

Nothing here restates the conventions. The gate, branch naming, commit
format, and what's not worth flagging live in CONVENTIONS.md.

## The cycle

| # | State | Label | Who acts |
| --- | --- | --- | --- |
| 1 | Ready to start | `task` | implementer picks it up |
| 2 | Plan posted, awaiting approval | `needs-approval` | maintainer approves or amends |
| 3 | Building | `in-progress` | implementer |
| 4 | PR open, under review | `needs-review` | reviewer |
| 5 | Findings to address | `in-progress` | implementer |
| 6 | Approved | `needs-review` removed | **maintainer merges** |

`blocked` sits outside this sequence: it means something went wrong and the
work has stopped. `needs-approval` is the expected pause every task takes at
step 2 — don't use `blocked` for it, or a genuine problem won't stand out in
the issue list.

Agents never merge and never push to `main`. Merging is the maintainer's,
always.

## Finding your work

Each task is a GitHub issue labelled `task`, and its body is the spec.
`gh issue list --label task` is the index. An issue's "Depends on" line names
the issues that must be merged first — don't start a task whose dependencies
are open.

An issue's number is not its task number. The task number is in the issue
title, and branches use the **task** number: `task-01-auth-and-app-shell`.

## Implementer: plan first

**Post a plan as an issue comment and stop. Do not write code until the
maintainer approves it.** The approved plan is what the reviewer judges the
PR against, so it is a contract, not a sketch.

Before planning, read the issue, CLAUDE.md, CONVENTIONS.md, and the code as
it actually exists today. The issue's acceptance criteria are deliberately
outcome-level; turning them into implementation is your job, and it's the
part the maintainer is approving.

The plan comment states:

- **Approach** — the shape of the change: models, controllers, routes,
  views, jobs. Name things.
- **Acceptance criteria, refined** — each issue AC restated as something
  concretely checkable, labelled with the issue's AC number. Every issue AC
  gets covered; if one can't be, say so and why rather than dropping it.
- **Tests** — what you'll write at each layer, including the failure and
  edge paths. CONVENTIONS.md makes happy-path-only incomplete.
- **Anything the issue leaves open** — every question you'd otherwise guess
  at. This is the section the maintainer is really reading.

Apply `needs-approval`, tag `@sean-dickinson`, and stop. On approval, swap
`needs-approval` for `in-progress` and build. If the maintainer amends the
plan, the amended version is the contract.

If the plan turns out to be wrong once you're building — not merely
inconvenient, but wrong — stop and escalate rather than quietly diverging.

## Escalation

Escalating is a normal outcome, not a failure. It is always cheaper than
guessing.

**Escalate when:**

- The issue or the approved plan doesn't settle something you need decided,
  and the alternatives lead to materially different work
- An acceptance criterion can't be met as written
- The issue conflicts with CONVENTIONS.md or CLAUDE.md
- A decision CONVENTIONS.md marks as deliberately open would have to be
  invented to proceed
- You'd have to add a gem, change the frontend stack, or touch something the
  issue put out of scope

**Decide it yourself when:** it's a routine judgment call a careful colleague
would make without asking, and being wrong is cheap to reverse.

**How:** comment on the issue, tag `@sean-dickinson`, cite the AC number,
state the options and which you'd pick and why, apply `blocked`, and stop.
One comment per blocker, not a running log. Do everything the blocker
doesn't touch before you stop.

## Review

Review fires when a PR is opened, independently — the implementer doesn't
grade its own work. The reviewer reads the PR diff, the linked issue, and the
approved plan comment, and works from the diff rather than from the
implementer's account of it.

Two levels, both required:

1. **Code against the plan.** Was the approved approach built, with the
   tests it promised?
2. **Plan against the issue.** Do the delivered ACs actually satisfy the
   issue's outcome-level ACs? A plan that met a lowered bar still fails
   here.

Every finding cites an issue AC, a plan item, or a specific rule in
CONVENTIONS.md. Anything that bothers you and isn't covered by one of those
is a **question**, not a finding — CONVENTIONS.md's "Not worth flagging"
list is binding, and so is each issue's own "Not findings" section.

The reviewer has read-only tools and does not fix what it finds.

### Ending the loop

- Reviewer posts findings → implementer fixes and re-requests review. Repeat.
- The implementer may **push back once** on a finding, with reasoning, and
  the reviewer may accept or hold.
- If the reviewer holds and the implementer still disagrees, that's a
  blocker: apply `blocked`, tag `@sean-dickinson`, state both positions,
  stop. Do not loop a third time on the same finding.
- When the reviewer has no findings it says so and removes `needs-review`.
  The maintainer merges.

## Pull requests

Per CONVENTIONS.md: a branch and a PR per task, conventional commit
subjects. The PR description additionally must:

- Say `Closes #N` so merging closes the issue
- Link the approved plan comment
- State which issue ACs are met, and name any that aren't and why
- State what was deliberately left out, and what you're least sure about

`bin/ci` must pass before requesting review, plus `bin/rails test:system`
if the task touched a view or a Stimulus controller.
