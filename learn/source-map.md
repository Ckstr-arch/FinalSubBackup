# Source map: where each responsibility lives

[Learning home](README.md) · [Architecture](../docs/architecture.md) · [Requirement traceability](../docs/requirements-traceability.md)

This inventory covers all **49 tracked baseline files** at `f455575` before `learn/` was added. Hidden public configuration and images are included. Private `.env`, local `var/`, Git internals and ignored Docker helpers are deliberately excluded. The baseline contains **36 non-Markdown files**; fingerprint verification is recorded in [learning validation](validation.md).

The primary home names a learning stage, not a claim that a single student originally authored the file. Several responsibilities share functions inside one file. Agree the exact function or page branch before concurrent editing.

## Web entrypoints and assets

| File | Responsibility and what to inspect | Primary stage |
|---|---|---|
| [public/index.php](../public/index.php) | Native page allowlist, POST action, page role gates, flash/layout rendering and error boundaries | 02/03 with 04 request boundary |
| [public/api.php](../public/api.php) | REST request ID, JSON content type, dispatch and safe exception/status boundary | 04 |
| [public/router.php](../public/router.php) | Strict local PHP-server routing; allowlisted public assets and private-path rejection | 05 |
| [public/.htaccess](../public/.htaccess) | Apache directory/index/rewrite configuration; actual host must support it | 05 |
| [public/assets/app.css](../public/assets/app.css) | Tokens, shared components, role layouts, breakpoints and theme overrides | 01 |
| [public/assets/app.js](../public/assets/app.js) | Navigation/focus/theme controls plus known-form REST enhancement | 01, with 02/03/04 form mapping |
| [public/assets/theme-init.js](../public/assets/theme-init.js) | Apply stored/default theme before page display | 01 |
| [public/assets/openapi.json](../public/assets/openapi.json) | Generated static API contract; not the runtime dispatcher | 04 |
| [public/assets/nsbm.png](../public/assets/nsbm.png) | Logo asset; confirm permission before public use | 01/05 |
| [public/assets/track.jpg](../public/assets/track.jpg) | Supplied athletics image; confirm usage rights | 01/05 |

## Private application source

| File | Responsibility and reading anchors | Primary stage |
|---|---|---|
| [app/bootstrap.php](../app/bootstrap.php) | Private env loader; `config`, MySQL version guard, `db/query/rows/one`, output `e`, redirects, sessions/CSRF/roles, field helpers and audit | 04 |
| [app/security.php](../app/security.php) | Base32/TOTP, `app_key`, secretbox encryption, throttle, `finish_login`, locked factor/recovery verification | 04 |
| [app/actions.php](../app/actions.php) | Shared command dispatch: auth, enrollment, booking/cancel, admin CRUD/review/attendance and 2FA | 04 shared; 02 student branches, 03 admin branches |
| [app/models.php](../app/models.php) | Entity allowlist, `save_entity`, `reserve`, `review_request`; validation/transactions/parent locking | 04 data layer; 02/03 workflows |
| [app/community.php](../app/community.php) | Exact-club access, community reads/writes, ownership, reaction/report, moderation and API subdispatch | 02, with 03 moderation and 04 security |
| [app/api.php](../app/api.php) | Safe user projection, JSON envelopes, parser/field checks/pagination, route/method/auth/CSRF/Origin and shared command adaptation | 04 |
| [app/layout.php](../app/layout.php) | Document shell, theme/menu controls, public/role navigation and page-view include | 01 |
| [app/ui.php](../app/ui.php) | SVG/icon helper, initials, club URL, common community form and metric rendering | 01 |
| [app/views.php](../app/views.php) | Shared markup helpers plus public/account/student/admin page branches, including Activity, Approvals and Reports; delegates portal views | 01 structure, 02 student, 03 admin |
| [app/portal-views.php](../app/portal-views.php) | Student/admin dashboards, My clubs, club tabs/community, moderation and audit markup/data reads | 02/03 |

## Database and command-line tools

| File | Responsibility and important boundary | Primary stage |
|---|---|---|
| [database/schema.sql](../database/schema.sql) | Twelve core InnoDB tables; keys, enums/defaults, constraints and delete relationships | 04 |
| [community migration](../database/migrations/20261017_club_communities.sql) | Four additive community tables; do not infer a completed deployment from filename | 04 |
| [bin/setup.php](../bin/setup.php) | Create private local `.env` with a random key only when absent | 05 |
| [bin/migrate.php](../bin/migrate.php) | Advisory lock, core schema, migration ledger and sorted unapplied files; MySQL DDL can auto-commit | 04/05 |
| [bin/doctor.php](../bin/doctor.php) | CLI prerequisites/key/actual MySQL/schema/timezone/writability checks; not a full HTTP acceptance test | 05 |
| [bin/seed.php](../bin/seed.php) | Optional fictional sports data, only exact development mode; no user accounts | 05 |
| [bin/create-admin.php](../bin/create-admin.php) | Private CLI administrator provision from process variables; no public setup page | 04/05 |
| [bin/demo-users.php](../bin/demo-users.php) | Optional development-only random synthetic accounts, explicit flag and private credential file | 05 |
| [bin/build-openapi.php](../bin/build-openapi.php) | PHP-only generation of static contract; inspect diff after a reviewed route/payload change | 04 |
| [.env.example](../.env.example) | Tracked nonsecret setting names/defaults; actual private values go in ignored `.env` or process config | 04/05 |
| [.gitignore](../.gitignore) | Excludes root `.env` and local `var/`; does not remove already-tracked secrets from Git history | 05 |

There are **17 tables after migration**: twelve core, four community and the `schema_migrations` ledger created by the CLI. `login_attempts` is operational state; it is not a participation table. The ledger is not part of `schema.sql` itself.

## Tests and what they actually establish

| File | Responsibility | Boundary |
|---|---|---|
| [tests/security.php](../tests/security.php) | TOTP vectors/drift/replay, authenticated encryption/tamper, password handling, escaping and engine-version helper checks | Requires valid private process/config key; no DB reset; does not exercise full HTTP sessions |
| [tests/audit.php](../tests/audit.php) | Read-only native-stack/source inventory and local Markdown file-target checks | No DB; does not render Markdown/diagrams or verify anchors/external links |
| [tests/integration.php](../tests/integration.php) | Native+REST HTTP journeys using actual isolated MySQL and synthetic users | Explicit reset flag, exact disposable schema, temporary local server on 8092; not for valued data |
| [tests/concurrency.php](../tests/concurrency.php) | Independent DB connection regressions for competing approvals/capacity/overlap | Same disposable schema, explicit flag, runs sequentially with integration |
| [tests/fixtures.php](../tests/fixtures.php) | Guarded destructive setup used by the DB suites | Not a standalone seed or a tool to run against normal application data |

See [Stage 05](05-assemble-deploy/README.md) and [native setup/test instructions](../docs/local-setup.md) before executing any DB suite. Example commands in a document are not permission to destroy another environment's data.

## Existing documentation

| File | What to use it for | Learning home |
|---|---|---|
| [README.md](../README.md) | Root setup, optional isolated Docker recipe, capability/documentation index | 05 |
| [docs/api.md](../docs/api.md) | Current PHP REST contract, payloads, pagination/errors, community endpoints | 04 |
| [docs/architecture.md](../docs/architecture.md) | Native request flow, source roles, ER relationships and state | All |
| [docs/build-order-and-contributions.md](../docs/build-order-and-contributions.md) | Earlier detailed four-track implementation/contribution order | All |
| [docs/contribution-log.md](../docs/contribution-log.md) | Eight-person roster/evidence template; fill genuine names/work | 05 |
| [docs/eight-member-plan.md](../docs/eight-member-plan.md) | A–H responsibility proposal and shared-file coordination | 05 |
| [docs/guidelines-audit.md](../docs/guidelines-audit.md) | Manual requirements, conversion decisions, provenance and remaining human tasks | 05 |
| [docs/local-setup.md](../docs/local-setup.md) | Native Windows/macOS/Linux setup, CLI admin/demo and test safeguards | 05 |
| [docs/report-and-video.md](../docs/report-and-video.md) | Real screenshots, identities, source/live/video links and contribution explanations | 05 |
| [docs/requirements-traceability.md](../docs/requirements-traceability.md) | Topic 3 features → actual routes/source/data and evidence | All/05 |
| [docs/security-and-hosting.md](../docs/security-and-hosting.md) | Production prerequisites, key/state handling, backup/restore and release procedure | 04/05 |
| [docs/start-finish-guide.md](../docs/start-finish-guide.md) | Short A–H entry/finish table and your proposed first H task | All |
| [docs/validation.md](../docs/validation.md) | Historical source/export checks and explicit unverified limits | 05 |

The new `learn/` package is indexed by [its README](README.md). It teaches these files rather than duplicating executable modules into five stage folders.

## Find a responsibility quickly

With ripgrep available, these are read-only searches from the repository root:

```bash
rg -n 'function (query|e|require_admin|csrf)' app/bootstrap.php
rg -n 'function (reserve|review_request|save_entity)' app/models.php
rg -n 'function (api_payload|api_dispatch)' app/api.php
rg -n 'function (club_access|moderate_post)' app/community.php
rg -n 'CREATE TABLE' database
```

If `rg` is unavailable, use editor-wide search. Function names are more durable reading anchors than hard-coded line numbers after edits. In shared files, search for the exact action/page name as well as the helper it calls.

## Keep ownership precise

- A/B coordinate shared markup and CSS/JS controls.
- C/E coordinate student/admin page branches in both view files.
- D/F/H coordinate exact action branches and tests; G reviews schema/query implications.
- H/G coordinate bootstrap/session/PDO changes because one edit can affect every page.
- Stage 05 integration records accepted work and actual tests; it does not silently rewrite everyone's changes.

When the source changes, update the corresponding lesson, contract and evidence. Do not use a stale diagram to overrule code behavior.
