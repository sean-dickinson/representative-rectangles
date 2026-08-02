# Conventions

How code in this app is expected to look. [CLAUDE.md](../CLAUDE.md) covers
*what* we're building and for whom; this file covers *how*, and it's the
standard a code review judges against.

Two audiences:

- **Implementing** — follow these unless a task spec explicitly overrides
  them. If you think a convention here is wrong for the task at hand, say so
  in your summary rather than quietly departing from it.
- **Reviewing** — flag departures from these rules. Do *not* invent
  additional standards; if something bothers you that isn't covered here,
  raise it as a question, not a finding. See
  [Not worth flagging](#not-worth-flagging).

## The gate

`bin/ci` must pass before any work is handed off for review. It runs setup,
RuboCop, the security scanners, `bin/rails test`, and a seed replant.

One wrinkle: system tests are commented out of `bin/ci` but do run in GitHub
Actions. Run `bin/rails test:system` yourself whenever you touch a view or a
Stimulus controller.

Never commit with failing tests or lint errors and describe it as done.

## Ruby and Rails style

- **Style is RuboCop's job, not yours.** We inherit `rubocop-rails-omakase`
  unmodified. Run `bin/rubocop -a`. Don't argue style in review, and don't
  add cops or `.rubocop.yml` overrides without a reason stated in the PR.
- **Scaffold with generators.** Use `bin/rails generate model/controller/migration
  ...` to create new models, controllers, and migrations rather than
  hand-writing the boilerplate — let Rails produce the migration, test
  skeleton, and file layout, then edit from there.
- **Rails-first.** Prefer a Rails built-in over a new gem, always: Active
  Storage for files, the built-in authentication generator for login, Solid
  Queue for jobs, Solid Cache, Action Mailer, `Rails.application.credentials`
  for secrets. Adding a gem requires naming the built-in you rejected and
  why. The one open exception is the PDF library, which is a deliberate
  undecided (see task 10).
- **No new frontend stack.** Hotwire (Turbo + Stimulus) and Bulma via
  `cssbundling-rails`, importmaps for JS. No React, no bundler swap, no
  Tailwind, no CDN script tags.
- Ruby 4.0 / Rails 8.1 idiom. Use what the version gives you rather than
  older workarounds.
- **The app runs in UTC.** `config.time_zone` is deliberately left at the
  default — per-user timezone is handled at display time, not by shifting the
  app's zone. Store UTC, convert when rendering. Don't set `config.time_zone`
  and don't flag its absence.

## Domain vocabulary

Use these words in class names, columns, routes, and UI copy. Inventing a
synonym for one of them is a review finding.

| Term | Means |
| --- | --- |
| **Assignment** | One year/class instance of the representative rectangles assignment |
| **Student** | A roster entry on an assignment. Not a user — no account, no password |
| **Submission** | A student's uploaded photo or PDF, plus its quality result and rotation |
| **Roster import** | Bringing a list of students onto an assignment from some source |
| **Submission link / token** | The unique unauthenticated URL a student uses |
| **Quality check** | Resolution/printability validation, resulting in pass / warn / fail |
| **Packet** | The generated print-ready combined PDF for an assignment |

Avoid "upload" as a noun for a Submission, "class" as a model name, and
"user" for anything student-facing.

## Models

- Validations, associations, scopes, and derived state live on the model.
  Enums for anything with a fixed set of states (quality result, submission
  status) — use Rails enums backed by a string column, not integers, so the
  database stays readable.
- Database constraints back up important validations: `null: false`,
  unique indexes on tokens and on `(assignment_id, email)`. A uniqueness
  validation without a unique index is a review finding.
- **Extract a class when there's a reason, not on principle.** Multi-step
  operations that touch several records, have their own error reporting, or
  are meant to be swappable (roster import, quality analysis, packet
  generation) get their own class under `app/models/` as a plain object.
  Single-record CRUD does not.
- `.claude/skills/sandi-metz-developer` is the reference for object design
  when you do extract. Small objects, dependencies injected, no
  `NullObject`-style ceremony for problems we don't have yet.
- No `default_scope`. No callbacks that touch other aggregates or enqueue
  side effects the caller can't see — enqueue jobs from the caller.

## Controllers

- RESTful resources. Reach for a new controller instead of a non-CRUD action
  on an existing one: `submissions#update` over `students#resubmit`.
- Skinny: find, delegate, respond. If a controller action is more than about
  ten lines, the work belongs in a model or a service object.
- Always strong params in a private `*_params` method.
- Redirect after a successful mutation, re-render with
  `status: :unprocessable_entity` on validation failure. Flash via
  `notice:`/`alert:`; the layout renders both.
- Teacher controllers inherit authentication from `ApplicationController`.
  There is exactly one teacher — no roles, no policy objects, no
  authorization gem.

## The public submission path

This is the only unauthenticated, internet-facing write path in the app.
Review it harder than anything else.

- Tokens are generated with `has_secure_token` (or `SecureRandom.urlsafe_base64`
  at 32+ bytes) — never derived from an id, name, or email.
- One token per **student**, not per assignment.
- Look students up by token only. A public route that accepts a student id
  or email is a security finding.
- The public pages show the student their own name and the assignment, and
  nothing else. No roster, no counts, no other students, no teacher data.
- Invalid or unusable tokens render a plain "this link isn't valid" page —
  never a 500, never an exception page, and never a message that
  distinguishes "no such token" from "expired" in a way that helps someone
  guess.
- Skip CSRF protection nowhere. The upload form is a normal Rails form.
- Public controllers live in their own namespace with their own base class
  that does not require authentication, so the app-wide default stays
  "login required."

## Files and Active Storage

- One attachment per submission. Re-submission replaces it (`attach` on the
  same `has_one_attached`), and the prior file is purged.
- Validate content type and size on the model. Accept images and PDFs;
  reject everything else with a message a student can act on.
- **Rotation is stored as a degrees integer on the submission**, never baked
  into the stored file. The original upload is immutable — the quality check
  and the packet both read the true original.
- Image inspection and variants go through `image_processing`/libvips.
  Anything slow (analysis, previews, packet generation) happens in a job,
  not in a request.

## Jobs

- Solid Queue, running in-Puma. Jobs must be idempotent — assume they can
  run twice.
- Pass record ids, never AR objects. Handle the record being gone.
- Anything a user waits on gets a visible state on the record (queued /
  processing / done / failed) so the UI can show it. Don't leave the teacher
  staring at a page with no feedback.

## Assets and CSS

Two separate dependency trees, and they don't overlap:

- **App JavaScript** — importmap pins, no bundler. Audited by
  `bin/importmap audit`.
- **CSS** — Bulma's sass source comes from npm and is compiled by
  `cssbundling-rails` into `app/assets/builds/application.css`, which
  Propshaft serves. Audited by `npm audit`.

Rules:

- **npm is the package manager.** `package-lock.json` is committed; run
  `npm ci` in CI and Docker, `npm install` locally. Don't add yarn
  invocations — one package manager, and it's npm.
- `app/assets/builds/` is gitignored. Never commit the compiled CSS.
- You rarely need to build by hand: `bin/dev` watches, and `css:build` is
  wired into both `assets:precompile` and `test:prepare`, so tests and
  deploys build it themselves. `npm run build:css` if you want it once.
- Customize Bulma through `application.bulma.scss` (`@use 'bulma/sass' with
  (...)`) or Bulma 1.x's `--bulma-*` custom properties. Don't hand-edit
  anything in `node_modules`, and don't vendor a copy of Bulma.
- The build must stay warning-free — `--quiet-deps` silences Bulma's own
  internal deprecations, so any warning you see is coming from our stylesheet
  and should be fixed rather than tolerated.
- Node is only ever used to compile CSS. It has no place in application code
  or in a request cycle.

## Views

- ERB and Bulma classes. Partials for anything rendered in more than one
  place; `_form.html.erb` shared between new and edit.
- No inline `style=` attributes and no `<script>` blocks in views. Behavior
  goes in a Stimulus controller.
- Turbo Frames/Streams are welcome where they genuinely help (rotate
  preview, roster row updates). Don't reach for them to avoid a full page
  render that would be fine.
- The dashboard is the highest-value screen in the app and it must load in
  one pass — eager-load associations, count in SQL. An N+1 on the roster or
  dashboard views is a review finding, not a nitpick.
- Every screen needs its empty state written, not just the happy path with
  data.

## Testing

Minitest, the Rails default. `fixtures :all` and parallel workers are
already on.

- **Fixtures, not factories.** `factory_bot` would be a new gem; don't add
  it. Keep fixtures minimal and meaningful.
- Tests run in parallel — no order dependence, no shared mutable global
  state, no reliance on autoincrement ids.
- What each layer is for:
  - **Model tests** — validations, state transitions, and any extracted
    service object's behavior including its failure paths. This is where
    roster import and the quality analyzer get covered properly.
  - **Controller/integration tests** — auth is enforced, params are
    filtered, the right redirect/status comes back, and the public token
    routes reject bad tokens.
  - **System tests** — the few flows where the interaction *is* the
    feature: student upload → preview → rotate → confirm, and the teacher
    scanning the dashboard. Keep these few; they're slow.
- Test behavior at its boundary, not implementation. Don't stub Active
  Record. Don't assert on private methods.
- **Don't test the framework.** A test asserting an association, validation,
  or enum exists — without exercising behavior that would break if it
  didn't — proves nothing. Test the behavior the association/validation
  enables (e.g. "assigning a duplicate email is invalid" not "validates
  uniqueness of email"), not that you wrote the declaration.
- **Every task's tests must cover the error and edge paths, not only the
  happy path.** A PR with only happy-path tests is incomplete. Missing
  coverage of a stated acceptance criterion is a review finding.
- Real sample files (photos and PDFs, portrait and landscape, and at least
  one too-low-res) live in `test/fixtures/files/`. The quality analyzer's
  thresholds are asserted against those real files, not against fabricated
  dimensions.

## Migrations

- One migration per schema change, reversible, `schema.rb` committed with
  it. Never edit an already-committed migration.
- SQLite: adding a column with a `null: false` default is fine; renames and
  type changes are more expensive — prefer additive changes.
- No data backfills mixed into a schema migration. If you need one, make it
  a separate migration or a rake task and say so.

## Comments and documentation

- Comment *why*, not *what*. Don't narrate code that reads clearly.
- Non-obvious decisions (a chosen DPI threshold, a scaling formula, a
  workaround) get a one-line comment naming the reason.
- Delete the Rails-generated placeholder comments from files you meaningfully
  edit. Don't leave commented-out code behind.

## Git

- A branch per task, named `task-NN-short-slug`. Never commit directly to
  `main`.
- Conventional commit subjects (`feat:`, `fix:`, `chore:`, `refactor:`,
  `test:`), imperative mood, present tense.
- A PR per task. The description states what was built, what was
  deliberately left out, which acceptance criteria are met, and what parts
  the author is least sure about.

## Not worth flagging

These are deliberate choices, not oversights. Raising them as findings wastes
a review round.

- No service layer / repository / query objects for plain CRUD.
- No authorization framework — one teacher, one role.
- No student accounts, sessions, or password flows.
- No multi-tenancy, no school or district model.
- No API, no JSON endpoints, no serializers unless a task asks.
- No Canvas integration, OAuth, or token storage. The roster importer is
  designed to accept a second implementation later; the absence of that
  implementation is correct.
- SQLite in production is intentional (Rails 8 + Solid stack, single small
  app). Don't suggest Postgres.
- No i18n extraction — English strings inline in views are fine.
- No RSpec. No factory gem. No linter or formatter beyond omakase RuboCop.
- Bulma's default look is fine. Design polish is out of scope unless a task
  says otherwise.
- Performance work beyond avoiding N+1s and keeping heavy work in jobs is
  premature at one-teacher, ~30-students-per-assignment scale.

## Where conventions are deliberately open

Don't invent an answer and don't flag its absence — the task spec decides,
and if the spec is silent, ask.

- **PDF generation library** — Prawn vs. HTML-to-PDF, decided by the task 10
  spike.
- **Quality thresholds** — the minimum resolution for a warn vs. a fail
  needs tuning against real sample files.
- **Token lifetime** — whether submission links expire, and what archiving an
  assignment does to its live links.
- **Mail delivery** — Action Mailer over the teacher's own SMTP credentials;
  where those credentials come from and how bounces surface isn't settled.
