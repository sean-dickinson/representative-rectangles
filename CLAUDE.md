# Representative Rectangles

A Rails app that replaces a painful manual workflow for a specific school
assignment: students submit a photo or PDF for the "representative
rectangles" assignment, and the teacher currently has to download each
submission from Canvas, paste it into a Google Doc/Slides deck, resize it by
hand, and export a combined PDF to send to the printer. This app exists to
remove that manual assembly step and make it easy for the teacher to track
completion and quality of submissions.

## Who uses this

- **Teacher**: one primary user.
  Logs in with simple built-in auth (Rails' built-in authentication
  generator — email/password, no SSO).
- **Students**: no accounts. Each assignment generates a unique
  link/token that a student uses to submit their file. No login, no
  password reset flows, no student-facing auth system.

## Core workflow the app must support

1. **Teacher creates an assignment** for the year/class — this should be a
   fast, repeatable action since the same assignment gets recreated every
   year with a new roster.
2. **Teacher imports a roster** via CSV (name + email, or similar). Design
   the import as a distinct, swappable step (e.g. a roster importer
   concept/interface that takes a normalized list of students) — a future
   Canvas API-based roster sync is a likely follow-up, and it should be able
   to plug in as a second implementation without reworking the assignment/
   student data model. Don't build the Canvas integration itself yet — no
   OAuth, no token storage, no Canvas client — just don't design the roster
   import in a way that makes it painful to add later.
3. **Student submits** a photo or PDF via their unique link:
   - Validate image quality/resolution at submission time (reject or warn on
     images too low-res to print well at letter size).
   - Let the student see a preview of their submission and rotate it if the
     orientation is wrong, before finalizing.
   - Support re-submission (replacing a prior file) since the teacher needs
     to be able to send students back to fix bad submissions.
4. **Teacher dashboard** shows, per assignment: who has submitted, who
   hasn't, and flags any submissions that failed/warned on quality
   validation. This "who's missing" view is a primary reason this app
   exists — prioritize making it fast to scan.
5. **Teacher previews submissions** — individually and as a batch — before
   generating the final packet.
6. **Generate a print-ready PDF**: one page per submission, each image/PDF
   page scaled to fill a standard US Letter page (8.5x11in) *without*
   cropping, respecting the original orientation (portrait vs. landscape)
   rather than forcing a fixed page orientation for every submission.

## Explicit non-goals

- No Canvas API integration yet (submission pull or OAuth) — CSV roster
  import only. Keep the roster-import boundary clean so this can be added
  later without a rework.
- No student accounts/login.
- No multi-tenant/school-admin layer unless asked for — build for a single
  teacher's assignments first.

## Open technical decisions

- **PDF generation approach** (Prawn vs. HTML-to-PDF via Grover/Puppeteer)
  is intentionally undecided — revisit this once we're implementing the PDF
  generation feature and have concrete scaling/orientation requirements to
  test against, rather than picking a library up front.

## Conventions

- This is a new Rails app — follow standard Rails conventions (RESTful
  resources, skinny controllers, ActiveRecord validations) unless a
  decision above says otherwise.
- Prefer Rails built-ins (Active Storage for file uploads, built-in
  authentication generator) over adding new gems/services unless there's a
  clear gap.
