# Source walkthrough · what is actually assembled

[Stage dashboard](README.md) · [Lessons](lessons.md) · [Exercises](exercises.md) · [Complete learning source map](../source-map.md)

This walkthrough follows current source inspected at `f455575`. The small excerpts below are copied from actual source; operational plans are labeled separately. Do not substitute an invented `home.html` or a second service for the existing PHP application.

## 1. Inspect the private template, not a secret file

Open [`.env.example`](../../.env.example). Its development defaults select native 8091, TCP127.0.0.1:3306, database/user `sportshub_backup`, and blank secret fields. A blank template password is an instruction to configure your own value, not a usable universal credential.

The native loader in [bootstrap](../../app/bootstrap.php) reads lines at the first equals sign and sets only values absent from the process environment. It deliberately does not behave like every Docker/env library. Values are literal; a quoted password would include the quote characters. The same module sets Asia/Colombo time and the PDO connection timezone, native prepares, actual MySQL version check, sessions and response protection.

**Input → result:** private configuration values → PHP environment/config helpers → database/session/security behavior. No configuration is embedded in a frontend build. Source should never need a developer's real password committed into a file.

## 2. Setup preserves existing encryption state

Actual [setup](../../bin/setup.php) excerpt:

```php
if (is_file($file)) { echo ".env already exists; preserved unchanged.\n"; exit; }
$key=base64_encode(random_bytes(32));
$data=file_get_contents($root.'/.env.example');
$data=preg_replace('/^APP_KEY=.*$/m','APP_KEY='.$key,$data);
```

- `is_file` is the early preservation gate: an existing file prevents the later key/write path.
- `random_bytes(32)` supplies the 256-bit random material; base64 represents it as text. Base64 itself is not encryption.
- `file_get_contents` reads the tracked template, not somebody else's local config.
- `preg_replace` changes the template's APP_KEY line. The multiline flag makes anchors apply to each line.

The following source writes with `LOCK_EX`, checks failure and requests 0600 permissions. On Windows, confirm actual ACLs; chmod alone does not establish the same access model. The helper is a local CLI convenience, not a concurrent public provisioning service.

**Failure reasoning:** if the original dataset already has enrolled users, generating a new key can make their authenticator secrets unreadable. That is why setup preserves an existing file and why backups must include the matching key.

## 3. Migration is not a reset

Open [migration runner](../../bin/migrate.php), [core schema](../../database/schema.sql) and [community migration](../../database/migrations/20261017_club_communities.sql).

Source order:

1. Require bootstrap and restrict use to CLI.
2. Derive a database-specific advisory lock and acquire it before schema work.
3. Execute core `CREATE TABLE IF NOT EXISTS` statements.
4. Ensure the `schema_migrations` ledger exists.
5. Sort additive migration files; skip recorded filenames.
6. Execute each unapplied file, record its version, and release the advisory lock in `finally`.

Actual excerpt:

```php
  if (one('SELECT version FROM schema_migrations WHERE version=?',[$version])) continue;
  // DDL may auto-commit. Each statement must be safe to retry after partial failure.
  foreach (explode(';',file_get_contents($file)) as $sql) if (trim($sql)!=='') db()->exec($sql);
```

A prepared filename lookup protects the ledger query. `continue` skips work already recorded. Semicolon splitting is deliberately simple; it is not a parser for stored procedures or arbitrary SQL containing semicolons in literals. MySQL DDL can commit implicitly, so a failure is not equivalent to rolling back normal row changes.

An applied file edited later is still skipped by filename. Add a new reviewed migration instead. `IF NOT EXISTS` also does not transform existing column definitions automatically. The advisory lock coordinates migration runners; row locks in domain services separately protect reservations and approvals.

## 4. Sports seed, demo accounts and real admin are different tools

| Actual source | Preconditions / behavior | What it does not do |
|---|---|---|
| [seed.php](../../bin/seed.php) | Exact development mode; skips if clubs exist; synthetic sports inserts in a transaction | Does not create login credentials or refresh old dates on every restart |
| [demo-users.php](../../bin/demo-users.php) | Development +`--confirm-demo`; unique generated passwords/key; private exclusive credential record; existing identities preserved | Does not publish fixed passwords, reset users or provision production |
| [create-admin.php](../../bin/create-admin.php) | Private CLI email/password; format/length checks; duplicate email refuses; Argon2id insert with admin role | Does not accept a browser role field or become a public account-reset script |

Demo provisioning creates its own private file under `var/`, verifies storage safety and saves credential material before committing new account rows. Its output points to that file rather than printing secrets. If the first sports club exists, the synthetic student/athlete get pending/approved memberships respectively; otherwise ordinary join/approval is still required.

These tools are not interchangeable. A production runbook must choose real private administrator provisioning and omit development seeding. A laptop demonstration can use synthetic data while keeping every secret off-screen.

## 5. Doctor proves prerequisites, not the whole product

Open [doctor](../../bin/doctor.php). It checks PHP version, PDO MySQL/mbstring/sodium, Argon2id, key validity, SELECT 1, actual MySQL, connection timezone, all 17 table names and private-path writability. It prints named PASS/FAIL outcomes and exits nonzero after any failure.

A CLI doctor can pass while Apache fails under a different user or PHP configuration. It checks whether `var` or root is writable; verify the actual web user's durable session path on a production host. It does not certify HTTPS, remote database TLS, every column/index or live business behavior.

**Safe debugging:** compare CLI/web configuration and permissions through protected administration tools. Never publish phpinfo, stack traces, `.env` or environment dumps to make troubleshooting easier.

## 6. Native HTTP route into a shared mutation

Read [index](../../public/index.php) → [actions](../../app/actions.php) → [models](../../app/models.php).

Actual native action wrapper:

```php
function handle_action(): never {
 check_csrf();
 $result=perform_action((string)($_POST['action']??''));
 if ($result['message']!=='') flash($result['message']);
 go($result['redirect']);
}
```

`check_csrf()` runs before dispatch. `perform_action()` selects a supported action and applies its own authentication/role/domain checks. A flash is short-lived session feedback. `go()` returns 303 so the browser follows with a GET, reducing accidental repost on refresh. The `never` return type says normal control does not return to this caller.

For `join`, `register_tournament` and `attend`, the service maps the action to memberships, registrations or attendance and calls `reserve()` with the validated ID and authenticated user ID. A student cannot turn a hidden field into an arbitrary SQL table name; the mapping is fixed.

## 7. REST route reaches the same service

Read [API entrypoint](../../public/api.php) and [dispatcher](../../app/api.php), then the [browser handler](../../public/assets/app.js). The entrypoint supplies an opaque request ID, JSON output/error handling and shared bootstrap/actions. The dispatcher validates route/method, Origin/CSRF and JSON shape before mutations.

Actual membership route excerpt:

```php
  $actions=['clubs'=>'join','tournaments'=>'register_tournament','events'=>'attend'];
  api_fields($payload,[]); $_POST=['id'=>$m[2]]; $result=perform_action($actions[$m[1]]);
```

The route regex has already captured an allowlisted entity and positive ID. These operations accept an empty JSON object; they do not accept a user ID or administrator role. `api_fields` rejects extra fields. The dispatcher deliberately adapts validated transport input into the shared action interface; it does not create a second copy of the business rule.

The server reads the persisted membership for the authenticated user, returns 201 and supplies redirect metadata. JSON errors use status/envelope semantics; native forms use rendered error feedback or redirects. A native 419 and REST 403 can both represent CSRF refusal, so parity means the same protection, not identical presentation/status in every case.

## 8. Approval, community gate and the database

`review_request()` in [models](../../app/models.php) requires allowlisted review targets/states and runs a fresh READ COMMITTED transaction. It reads the candidate, locks the shared parent, locks/rereads the request, checks capacity or overlap, writes status and audits before commit. Consistent parent-first ordering avoids incompatible lock sequences between reviewers/reservers.

The [community service](../../app/community.php) rechecks approved membership in that exact club or administrator status on every request. It is not enough to cache “joined a club” once in a browser. Leaving removes membership and should revoke the next feed request. Private report reasons remain separate from member feed output.

See [schema](../../database/schema.sql): the user/club pair is unique and its foreign keys maintain relationships. Those keys do not decide whether a particular browser user may approve someone else. SQL constraints and PHP authorization solve different problems.

## 9. Routing defines what can be downloaded

For native development [router.php](../../public/router.php) routes `/api/`, normal entry pages and validated public assets; private/unknown paths get 404. It checks decoded paths, real locations, hidden segments and allowed asset types before delegating files.

For Apache [`.htaccess`](../../public/.htaccess) disables indexes, selects index.php and rewrites `api/` to api.php. It works only when the server's document root is public/ and Apache permits the required rewrite overrides. Nginx ignores this file and needs verified equivalent configuration.

A correct rewrite does not rescue a repository-root webroot. If `.env` or `var/demo-credentials.json` is directly downloadable, release is blocked even if the home page looks correct. Fixed-public_html hosting requires a supported private-project/public mapping; review/test any path adaptation rather than blindly moving files.

## 10. Actual test boundaries and evidence

Open [security](../../tests/security.php), [audit](../../tests/audit.php), [fixtures](../../tests/fixtures.php), [HTTP integration](../../tests/integration.php) and [concurrency](../../tests/concurrency.php).

- Security checks helpers without resetting SQL.
- Audit checks distribution/local Markdown targets without opening private `.env`/`var` or resetting data; its scope excludes anchors/external availability/rendering.
- Fixtures require exact disposable schema, development mode, actual MySQL, CLI and explicit reset flag before table drops.
- Integration owns an HTTP server on 8092, separate cookie clients, synthetic credentials and real SQL assertions.
- Concurrency proves a worker reaches a held parent lock, commits a competing approval, then checks the waiter refuses an over-capacity/overlapping approval.

Those are concrete guarantees under the recorded environment, not blanket production certification. Use [application validation](../../docs/validation.md) for historical runs and [learning validation](../validation.md) for this documentation task's actual checks. No new execution count is established by the excerpts in this walkthrough.

## One-minute explanation to personalize

Point to the configuration gate, migration ledger, actual document root and one native/REST call chain. Explain which invariant the service and SQL each protect. Show one personally observed result, state its environment and limitation, and name what you changed or studied honestly. Do not say that receiving this guide made you the author of the baseline.
