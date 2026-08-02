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
maintainer approves it.** The issue's acceptance criteria say what must be
true; the approved plan commits you to how you'll get there. The reviewer
holds you to both.

Before planning, read the issue, CLAUDE.md, CONVENTIONS.md, and the code as
it actually exists today.

A plan conveys **the general shape of the work and the decisions behind it**.
It is not a specification of the change. Do not list the files you'll touch,
do not sketch code, and do not enumerate the tests you'll write —
CONVENTIONS.md already says what each test layer is for and requires the
failure and edge paths be covered, so restating that in a plan is noise.
Aim for something a person can read in a minute.

Use these sections:

```
## Approach
## Design decisions
## Open questions
```

- **Approach** — the shape of the change in a short paragraph or a few
  bullets. Which parts of the app grow, and how they fit together. Enough
  that the maintainer can picture it and object; no more.
- **Design decisions** — every choice the issue and the conventions did not
  already make for you. For each: what you chose, what you considered
  instead, and why. This is the section the maintainer is actually reading.
  A plan with an empty Design decisions section on a non-trivial task means
  you haven't looked hard enough.
- **Open questions** — anything you'd otherwise have to guess at. If a
  design decision could reasonably go either way and you have no strong
  preference, it belongs here rather than above.

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

1. **Code against the issue.** Is every acceptance criterion actually
   satisfied, failure and edge paths included? The issue is the contract for
   *what* — a PR that follows its plan faithfully but misses an AC fails
   here.
2. **Code against the plan.** Was the approved approach the one built, and
   were the approved design decisions the ones made? Silent divergence is a
   finding even when the result works — the maintainer approved a shape, and
   changing it mid-build is an escalation, not a judgment call.

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
