# Representative Rectangles

A Rails app for collecting and assembling student submissions for the
"representative rectangles" assignment. Students upload a photo or PDF via a
unique link, the teacher tracks who's missing and reviews quality, and the app
generates a print-ready PDF packet — one page per submission, scaled to fill
US Letter without cropping.

Replaces a manual workflow of downloading each Canvas submission, pasting it
into a Google Doc, resizing by hand, and exporting for the printer.

- [CLAUDE.md](CLAUDE.md) — what we're building and for whom
- GitHub issues labeled `task` — ordered plan of work, and what's blocked
- [docs/CONVENTIONS.md](docs/CONVENTIONS.md) — how code here is expected to
  look, and the standard code review judges against

## Requirements

- Ruby 4.0.0 (see `.ruby-version`)
- Node 22.22.3 (see `.node-version`) — used *only* to compile Bulma's sass
  into the CSS bundle, never at runtime
- `libvips` for image processing (`brew install vips` on macOS)

SQLite is the database in every environment, including production. There's no
Postgres, Redis, or separate job server to run — Solid Queue, Solid Cache, and
Solid Cable are all SQLite-backed, and jobs run inside Puma.

## Getting started

```sh
bin/setup
```

Installs gems, installs npm dependencies, prepares the database, and starts
the server. Pass `--skip-server` to stop before booting.

Afterwards, run the app with:

```sh
bin/dev
```

This runs Puma and a sass `--watch` process together via `Procfile.dev`, so
CSS rebuilds as you edit. Visit http://localhost:3000.

## Tests and checks

```sh
bin/ci
```

`bin/ci` runs setup, RuboCop, three security scanners (`bundler-audit`,
`npm audit`, Brakeman), the test suite, and a seed replant. Individual pieces:

```sh
bin/rails test         # unit, model, controller, integration
bin/rails test:system  # system tests (not in bin/ci; runs in GitHub Actions)
bin/rubocop -a         # style, autocorrecting
bin/brakeman           # security static analysis
```

System tests are deliberately excluded from `bin/ci` for speed but do run in
CI, so run them yourself when you touch a view or a Stimulus controller.

## CSS

Bulma comes from npm and is compiled by `cssbundling-rails` into
`app/assets/builds/application.css`, which Propshaft serves. That directory is
gitignored — never commit the compiled output.

You rarely need to build by hand: `bin/dev` watches, and `css:build` is wired
into both `assets:precompile` and `test:prepare`. To build once:

```sh
npm run build:css
```

npm is the package manager — `package-lock.json` is committed. Customize Bulma
via `app/assets/stylesheets/application.bulma.scss`.

## Timezone

The app runs in UTC. Per-user timezone display is handled separately rather
than by setting `config.time_zone`.

## Deployment

Kamal to a single VPS, with the `storage/` volume holding both the SQLite
databases and Active Storage files. The deploy config is still scaffolding —
registry, host, SSL, secrets, and off-server backups are all outstanding. See
the "Production deploy config" task issue.
