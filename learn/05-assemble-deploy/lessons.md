# Lessons · from a checkout to a reviewable release

[Stage dashboard](README.md) · [Source walkthrough](source-walkthrough.md) · [Practical labs](exercises.md) · [Solo path](../solo-path.md)

## 1. What “assemble” means in this repository

A working application depends on more than source files. It also needs a compatible interpreter, database, private configuration, writable private state and correct HTTP routing. Assembly is making those dependencies agree. The five learning stages are instruction packages, not five services, five webroots or five databases to merge.

A browser cannot execute PHP. The webserver runs a PHP entrypoint; PHP reads MySQL and renders HTML or JSON; the browser applies CSS and executes JavaScript. The runtime remains the existing `app/`, `public/`, `database/` and `bin/` source. Use the [source map](../source-map.md) when responsibility crosses a directory boundary.

**Predict:** if every PHP file is present but PDO MySQL is missing, will replacing CSS fix login? No. The failed dependency is the PHP database driver. Diagnose the layer that failed before changing another layer.

## 2. Establish a known local baseline

Future human setup commands, from a chosen private projects directory:

```bash
git clone https://github.com/Ckstr-arch/FinalSubBackup.git
cd FinalSubBackup
git status --short
git branch --show-current
git log -1 --oneline
php --version
php --ini
php -m
php -r 'var_export(password_algos());'
```

`clone` creates the source checkout and its Git history. It does not create `.env`, users, sessions or a database. `status` protects existing work by showing edits before you change anything. `--ini` identifies the PHP configuration actually used by the CLI; the webserver may use a different configuration or user.

PHP must be 8.2+ with PDO MySQL, mbstring, sodium and Argon2id. The database must be **actual MySQL 8+**. This edition intentionally rejects MariaDB even if a local control panel labels it “MySQL.” Check the actual server version through a private database client rather than trusting a button label.

Platform instructions are in [local setup](../../docs/local-setup.md): Windows uses compatible PHP plus genuine MySQL, macOS uses supported Homebrew PHP/MySQL packages, and Linux uses supported distribution or MySQL packages. Do not install Node, Composer or a framework to run this native application. Optional editor/test tools do not become runtime dependencies.

## 3. Configuration, schema and accounts are separate steps

Create a fresh separate `sportshub_backup` database and dedicated account using the setup guide. Do not point this package at the original `sportshub` database. Existing databases and keys are not disposable because their names look familiar.

```bash
php bin/setup.php
# Edit the new private .env in your editor with your own database settings.
php bin/migrate.php
# Optional development sports data only:
php bin/seed.php
php bin/doctor.php
```

These commands do different jobs:

| Command | Reads | Writes / result |
|---|---|---|
| setup | `.env.example`, existence of `.env` | Creates private config and random key only if absent |
| migrate | SQL source and migration ledger | Creates missing schema/applies intended unapplied versions |
| seed | Development mode and existing club count | Optional synthetic sports rows; no login accounts |
| doctor | PHP capabilities, key, database/schema and path checks | PASS/FAIL diagnostics; does not reset tables |

Native `.env` values are literal strings: do not add surrounding quotes or inline comments. Existing process environment values take precedence over file values. Investigate variable names/presence privately if a command unexpectedly targets a different server; never dump the full environment.

`DB_PASSWORD` authenticates a database account. A website password authenticates a row in `users`; the operating-system password authenticates the computer account. They are not interchangeable. `APP_KEY` is neither kind of password: it encrypts/decrypts enrolled authenticator secrets and must match its dataset.

Provision a real administrator with [the private CLI procedure](../../docs/local-setup.md), then enroll 2FA. It uses private process environment input and refuses an existing identity. Do not turn this into a web-accessible setup endpoint or paste passwords into command arguments.

For disposable development only, `php bin/demo-users.php --confirm-demo` creates random per-installation credentials in ignored private `var/demo-credentials.json`. Open it privately; do not print it into shared logs or commit it. Existing identities/credential records are preserved. The athlete's authenticator secret is unique; no universal code is shipped.

## 4. The document root and the origin

```bash
php -S localhost:8091 -t public public/router.php
```

`-t public` exposes only the public directory. The router argument handles REST routes and validated assets; omitting it can make `/api/v1/...` fail while ordinary pages still appear. Ctrl+C stops this particular development server. Keep MySQL running separately.

Use the exact configured origin: `http://localhost:8091`. An **origin** is scheme +host +port; changing localhost to 127.0.0.1 creates a different origin. The REST Origin check intentionally rejects mismatches. Do not “repair” a 403 by removing CSRF/Origin validation.

Health is `/api/v1/health`; it performs a database-backed readiness check. A listening port or an HTML screenshot alone does not prove MySQL, login or role enforcement works. Conversely, health success does not prove every business workflow, keyboard interaction or production backup.

## 5. Integrate one full membership journey

The first shared target is student register/sign in → apply → pending → administrator approves → approved student enters the private club space. It joins all required technologies and both roles.

```mermaid
sequenceDiagram
    actor S as Student browser
    participant Entry as Native page or REST entrypoint
    participant Service as Shared PHP action and model
    participant DB as MySQL
    actor A as Administrator browser
    S->>Entry: Apply with club ID and session CSRF
    Entry->>Service: Authenticated user and validated input
    Service->>DB: Lock club; check duplicate/capacity; write pending
    DB-->>Service: Commit succeeds
    Entry-->>S: Redirect or JSON with redirect metadata
    A->>Entry: Approve pending membership
    Entry->>Service: Require admin; review request
    Service->>DB: Parent lock; fresh checks; approve and audit
    S->>Entry: Read club community
    Entry->>DB: Require approved membership in this exact club
    Entry-->>S: Approved feed or authorization refusal
```

With JavaScript disabled, native POST includes hidden `action`, `csrf` and named inputs. `public/index.php` calls `handle_action()`, which checks CSRF and calls `perform_action()`. The result becomes a flash and 303 redirect.

With JavaScript enabled, the real form mapping in `app.js` sends same-origin JSON plus `X-CSRF-Token` to the REST route. `public/api.php` and `app/api.php` validate transport/fields/roles, then call the shared action/service. The response contains data, metadata and often a redirect. H reviews the server/API boundary; the affected UI contributor verifies browser/native behavior.

Do not count the two paths as two independently implemented membership rules. Their shared service is precisely why parity can be maintained. Test success, duplicate, pending access denial, approval and leaving; a button becoming visible is not sufficient proof of authorization.

## 6. Safe checks and destructive test boundaries

Read-only or non-reset verification first:

```bash
php tests/security.php
php tests/audit.php
git diff --check
```

Security exercises cryptography, escaping and engine-version boundaries without resetting SQL. Audit checks source distribution and local Markdown file targets; it does not certify browser rendering, external URLs or anchors. Source syntax checks are useful but cannot establish functional correctness.

Optional functional verification requires the **literal separate disposable** `sportshub_backup_test` database. Confirm the server, schema and ownership, prove no wanted data exists there, and give the appropriate test account schema access. A reassuring database name is not proof of disposability.

```bash
# Only after the disposable schema preflight; run one after the other:
php tests/integration.php --confirm-test-reset
php tests/concurrency.php --confirm-test-reset
```

Both runners rebuild that test schema. The HTTP runner owns port 8092 and generated synthetic credentials. The concurrency runner uses independent PHP/database connections and no HTTP port. They cannot safely run concurrently because they share their reset target. The fixture checks CLI, explicit flag, exact name, actual connected schema, development mode and genuine MySQL before destructive statements.

Record command, commit, environment, actual outcome and limitation. Historical 158/10/23 counts in [validation](../../docs/validation.md) are evidence from named past runs, not automatically your result today. Never write “all tests passed” when you ran only syntax checks or skipped the database suites.

## 7. Eight contributors can work in parallel

Use the [eight-person plan](../../docs/eight-member-plan.md). A–G are proposed slots until humans agree; H is the requester's selected API/authentication/security responsibility. The integrator checks connections and evidence without acquiring everyone else's authorship.

| Slots | Bounded work | Required neighboring review |
|---|---|---|
| A/B | Shell/semantic markup; CSS/theme/drawer interaction | C/E for changed pages; H for form/API boundaries |
| C/D | Student views; domain workflow/test cases | G/H for SQL/auth; E/F for approval consequences |
| E/F | Admin views; CRUD/decision/report rules | C/D for student results; G/H for invariants/security |
| G/H | Schema/query/migration; API/auth/security contract | All affected domain/UI contributors |

Shared files are not divided by fabricated ownership. Before editing dense `app/views.php`, identify the exact page branch; before changing `app/actions.php` or `app/models.php`, name the action/function and agree the expected HTTP/SQL outcomes. Keep formatting changes scoped, avoid competing edits to the same region and request review before changing the contract.

Future human Git sequence, after studying/testing an actual personally owned patch:

```bash
git status --short
git switch -c feat/h-csrf-parser-regressions
# Personally implement an agreed change and run its focused checks.
git diff --check
git diff
# Example paths only: stage the actual reviewed files you changed.
git add tests/integration.php
git commit -m "test(api): cover agreed parser refusal cases"
git push -u origin feat/h-csrf-parser-regressions
```

These are teaching instructions, not actions performed by this stage. Use your own authentic Git identity/account, never a teammate's identity. Open a real PR with purpose, source regions, reproduction, actual tests, assistance disclosure and reviewer feedback. Do not stage another person's edits or private state. Do not force push or manufacture activity commits.

Prefer a reviewed merge method that preserves actual individual commits where course evidence needs it; verify attribution and final behavior after integration. A squash can consolidate commits and change how authorship appears. No merge strategy guarantees marks. Use [the contribution log](../../docs/contribution-log.md) for actual work, not retroactive ownership of baseline modules.

## 8. Docker is an optional local route

The [root recipe](../../README.md#optional-docker-setup--isolated-local-development) defines four ignored helpers under `var/docker/`. Follow it completely; do not invent a tracked container stack or copy a different project's Compose file.

Its defaults are project `finalsubbackup-local`, actual MySQL 8.4 and loopback web 8095. The database has no published host port; app DB_HOST is `db`. The build context allowlist excludes private config/state before transfer; Git ignore alone does not protect a Docker context.

```bash
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml config --quiet
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml ps
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml exec app php bin/doctor.php
```

Keep the same project/env/Compose files for stop/start/restart and reviewed `up --build -d` updates. Never delete volumes or globally prune to update the app. `db_data` retains database rows; `app_data` retains private state; APP_KEY stays in private helper settings. Changing an env-file password does not change a user already initialized inside MySQL.

Docker Desktop installation is platform-specific; the recipe's Linux verification does not claim a Mac/Windows run. A local Docker success is still not public deployment.

## 9. Release go/no-go and recovery

A real host must run PHP 8.2+ and actual MySQL 8+, support required extensions/Argon2id, HTTPS, private source/config, writable durable private sessions, migrations/provisioning and trustworthy logs/backups. Provider names/prices are **UNVERIFIED** until checked on the current plan. Static-only GitHub Pages cannot run this backend.

Production settings include exact HTTPS APP_URL and APP_ENV=production. A fresh first production dataset gets a fresh private key; every later release retains the matching key. File sessions suit the current single-instance architecture; multiple replicas require a reviewed shared store. The current PDO constructor does not configure arbitrary provider-specific remote-database TLS options. Resolve and test that gap before crossing an untrusted network; never disable certificate checks as a shortcut.

Back up database and matching private key/config outside webroot/source control. A dump file existing is not a restore test. Restore only into a new isolated scratch database, verify target selection, keep the copied key private, then check relations/counts and safe login/2FA. Never rehearse by importing over valued live data.

A future human backup command, after choosing a private protected directory and the actual server/account, follows the existing hosting guide:

```bash
mysqldump --host=127.0.0.1 --port=3306 --user=sportshub_backup \
  --password --single-transaction --no-tablespaces \
  --result-file=/PRIVATE-BACKUP-DIRECTORY/sportshub-backup.sql \
  sportshub_backup
```

The directory above is a placeholder, not a public download location. The multiline example uses Bash continuation syntax; in Windows PowerShell, run the command with all arguments on one physical line and choose a protected Windows path for `--result-file`. A backslash is not PowerShell's line-continuation character. `--password` prompts instead of putting a value in command arguments. `--single-transaction` takes a consistent InnoDB data snapshot under suitable conditions; coordinate schema changes during backup. `--result-file` also avoids PowerShell text-redirection encoding surprises. Protect the output and matching key separately from source control.

For a restore rehearsal, create a **new** scratch schema with an authorized administrator, inspect the dump for `USE`/database-selection statements, and open a client explicitly targeting that scratch schema. Confirm `SELECT DATABASE();` before importing. A separate scratch app origin/config/key copy must remain private; do not let its browser cookies or writes target the live installation. Use the host/client's supported import procedure and verify the actual target rather than piping an unreviewed dump into a privileged connection. No backup or import command was executed to write this lesson.

Before rollback, identify old code's schema expectations and whether new writes would be lost. Keep prior reviewed code and a verified backup; do not automatically reverse migrations or replace a current dataset with an old snapshot. A code-only rollback may be possible for compatible additive schema, but requires review rather than assumption.

**No-go** if secrets/private paths are exposed, the engine is wrong, key/state is ephemeral, a required host capability is unverified, restore is untested, critical checks fail or release permission is absent. No fictional deployment date/host/URL belongs in evidence.

## 10. Report and video finish the assignment

Use [report/video guidance](../../docs/report-and-video.md) and [requirements traceability](../../docs/requirements-traceability.md). The rubric includes report, screenshots, accessible source/live links and teamwork video; code alone does not finish submission.

Capture actual final-release screens with synthetic identities, and record route/commit/environment/scenario. Never record setup keys, recovery codes, passwords, cookies or real student records. Each of eight speakers explains their actual patch, investigation, test or design work and assistance used. If H completes work alone, record that honestly rather than inventing seven contributions.

Names, accepted roles, release date, confirmed deadline, final hosted URL and video URL remain human evidence fields until known. The manual's deadline is TO BE CONFIRMED. Finish with a release decision and precise outstanding work, not a claim that completing a guide deployed the application.
