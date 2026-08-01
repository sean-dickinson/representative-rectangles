# Task Breakdown

Working plan for building out the app described in [CLAUDE.md](CLAUDE.md).
Each task is meant to be a reviewable chunk that leaves the app in a working
state. This list is expected to change as we learn things — update it rather
than treating it as fixed.

Status legend: `[ ]` not started, `[~]` in progress, `[x]` done.

## Foundation

- [ ] **1. Auth + app shell**
  Run `bin/rails generate authentication`, seed a teacher user, require login
  app-wide, add a real Bulma layout with nav and flash rendering. Root route
  points at a placeholder teacher dashboard.
  *Done when: log in, log out, and reach a protected empty dashboard.*

- [ ] **2. Core data model — Assignment, Student, Submission**
  Migrations, models, validations, associations. Install Active Storage and
  attach files to Submission. Model-level tests. This is where the roster
  importer boundary gets designed into the schema: students belong to an
  assignment and carry a source / external-id column so a future Canvas sync
  can upsert against them. Each student carries their own unique submission
  token (one per student, not one per assignment) — this keeps the "who's
  missing" view exact and makes re-submission a clean per-record replace.
  *No UI beyond what the tests need.*

- [ ] **3. Assignment CRUD**
  Teacher can create, list, edit, and archive assignments. Include (or at
  least don't design against) "duplicate last year's assignment" — fast
  repeat creation is an explicit requirement.

## Roster

- [ ] **4. Roster importer interface + CSV implementation**
  A `RosterImport` service taking a normalized list of student structs and
  upserting them onto an assignment, plus a CSV source that parses an
  uploaded file into that shape. Upload → preview → confirm flow, with
  per-row error reporting. The interface is the part worth scrutinizing in
  review; the CSV parsing is mechanical. Also add a "send/resend submission
  link" action (single student and bulk) using ActionMailer configured
  against the teacher's own SMTP credentials (e.g. a Gmail/Workspace app
  password) — no paid transactional email service, no self-hosted MTA.

## Student submission

- [ ] **5. Token-based submission entry point**
  Public unauthenticated route resolving a token to a student + assignment.
  Shows the student their name and the assignment, plus an upload form.
  Handles invalid, expired, and already-submitted states. Naive "upload and
  you're done" — no quality validation or preview yet.

- [ ] **6. Image quality validation**
  An analyzer inspecting dimensions/DPI against what's needed to print at
  Letter, returning pass / warn / fail with a reason. Runs on attach, result
  persisted on the submission, surfaced inline to the student. Handles PDFs
  too (page size, raster vs. vector). Its own task because the thresholds
  need tuning against real sample files.

- [ ] **7. Preview, rotate, finalize**
  Student sees a rendered preview, rotates in 90° increments, then confirms.
  Rotation is persisted as a degrees value on the submission record, not
  baked into the stored file — keeps the override in task 9 cheap and lets
  the quality analyzer (task 6) validate the true original file.
  Re-submission replaces the prior file.

## Teacher review

- [ ] **8. Dashboard — who's missing**
  Per-assignment roster view: submitted / not submitted / flagged, with
  counts, sortable and fast to scan. Highest-value screen in the app; keep it
  separate from assignment show rather than tacking it on.

- [ ] **9. Submission review**
  Individual submission view with full-size preview, a teacher-side rotate
  override (updates the same degrees value from task 7, no file
  reprocessing), and a "send back / needs redo" action. Plus a batch
  contact-sheet view of every submission for an assignment.

## Output

- [ ] **10. Spike — PDF generation approach**
  Timeboxed. Build both a Prawn path and an HTML-to-PDF path against 5–6 real
  sample submissions (portrait photo, landscape photo, portrait PDF,
  landscape PDF, odd aspect ratio) and compare output fidelity and per-page
  orientation handling. Deliverable is a decision plus throwaway code.

- [ ] **11. PDF packet generation**
  Implement the chosen approach: one page per submission, scaled to fill
  Letter without cropping, per-page orientation taken from the source and
  the stored rotation degrees applied at render time. Runs as a Solid Queue
  background job, result attached to the assignment, with a download link
  on the dashboard.

## Deployment

- [ ] **12. Production deploy config**
  Kamal to a VPS; SOLID_QUEUE_IN_PUMA and the `storage` volume mount
  (SQLite dbs + Active Storage files) are already scaffolded by the Rails 8
  default `config/deploy.yml`/`config/puma.rb` — no separate job server
  needed at this scale. What's left: off-server backups (e.g. Litestream
  replicating the SQLite files to object storage, since there's no managed-
  DB backup to rely on), and filling in real secrets (`.kamal/secrets`),
  domain, and SSL (the commented `proxy:` block) for the actual VPS.

## Sequencing

Tasks 1–3 are strictly sequential. After that, task 4 and tasks 5–7 can run in
parallel, and task 10 can start any time there are sample files to test
against.
