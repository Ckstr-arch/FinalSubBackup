# Lessons · from PHP syntax to preserved invariants

[Track dashboard](README.md) · [Actual source](source-walkthrough.md) · [Exercises](exercises.md)

## 1. PHP produces the response

PHP runs on the server; the browser receives HTML or JSON, not the source implementation.
`public/index.php` coordinates page requests and `public/api.php` coordinates JSON requests.
Both load bootstrap and shared actions. The homepage is emitted by PHP views, not a production home.html.

`<?php` enters PHP mode. A semicolon ends a statement; braces group function/condition bodies.
A template may switch out of PHP to emit literal HTML, then use `<?= e($value) ?>` to echo escaped text.
A server-side condition can decide whether a form exists; CSS merely changes the appearance of existing elements.
Neither a hidden form nor a disabled button enforces server access.

### Read a real helper, one operation at a time

Actual source from [bootstrap](../../app/bootstrap.php):

```php
function query(string $sql, array $args = []): PDOStatement { $s = db()->prepare($sql); $s->execute($args); return $s; }
function rows(string $sql, array $args = []): array { return query($sql,$args)->fetchAll(); }
function one(string $sql, array $args = []): ?array { return query($sql,$args)->fetch() ?: null; }
```

- `$sql` is a local parameter; `$args` defaults to an empty array.
- `string`, `array` and `PDOStatement` describe parameter/return types.
- `?array` permits an array or null, allowing callers to detect a missing row.
- `db()` returns an object; `->prepare()` and `->execute()` call methods on objects.
- `$s` is local to query; another function does not automatically see it.
- `return` gives a result to the caller; it does not automatically print JSON or HTML.
- `fetchAll()` consumes all result rows; `fetch()` consumes one row.
- `?: null` turns a falsey fetch result into null, distinct from a valid row containing a zero-valued field.

An associative array maps keys to values, such as `$u['role']` or `['id'=>17,'status'=>'pending']`.
A list holds ordered values such as `[$uid,$clubId]` matching SQL placeholder order.
`foreach ($spec['fields'] as $field=>$type)` visits metadata keys and their validation/UI descriptions.
Array unpacking `[...array_values($data),$id]` preserves values then appends an UPDATE row ID.
Changing field order without corresponding value order can write correct values into incorrect columns.

`===` compares type and value; `??` supplies a value when absent/null; `??=` assigns only when absent/null.
Use `!== null` for TOTP counter results, because zero can be a valid number rather than failure.
`fn($k)=>"$k=?"` is PHP's short arrow-function syntax; here it builds assignments from trusted field metadata.

### Includes, scope and exceptions

`require` loads another PHP file and fails when unavailable; `require_once` avoids loading it twice in a request.
`__DIR__` resolves the current source directory independently of the shell working directory.
`ROOT` points to this package's private root above public/.
It controls .env, SQL and private session/key paths, so changing it casually affects much more than page routing.

Some files declare `strict_types=1`; that affects scalar calls made from those files.
It does not validate HTTP fields or automatically turn request strings into safe domain objects.
The current actions/models files do not uniformly declare it, so explicit validators remain essential.

`throw` stops normal execution and passes an exception to a matching catch.
`RequestError` extends DomainException with an explicit HTTP status; other DomainException failures become 422.
Unexpected errors become generic responses with private diagnostics, rather than exposing SQL or stack traces.
Only an active transaction that is rolled back undoes its SQL writes; a catch alone is not rollback.

## 2. Requests, forms and response types

`$_GET` contains query parameters, `$_POST` contains native form values, and `$_SERVER` contains request context.
All browser-supplied data is untrusted even when sent by this application's own form.
`$_SESSION` contains server-side session state referenced by a browser cookie.
REST JSON is read from php://input, then adapted into the existing action field interface.

| Transport | Request | Shared rule path | Success response |
|---|---|---|---|
| Native form | POST with action/id/csrf plus fields | index → handle_action → perform_action → model/service | Flash notice +303 redirect, then rendered GET |
| Enhanced form | Fetch with route, method, JSON, cookie and X-CSRF-Token | public API → api_dispatch → perform_action → same service | JSON data/meta, then frontend navigation when appropriate |

303 means the next navigation uses GET, avoiding a refresh that resubmits the same POST.
JSON `meta.redirect` is a UI hint; it does not grant access or force every API client to navigate.
Fetch resolving is not proof of HTTP success: JavaScript must check response.ok/status and read the error envelope.
If a response is lost after a mutation, SQL may already have committed; inspect state before retrying.

### Actual REST input contract

All route names below are relative to `/api/v1`.
GET `/auth/session` returns safe current identity/pending flag and current CSRF metadata with a session cookie.
Every non-GET request requires JSON Content-Type, an object body, cookie continuity and X-CSRF-Token.
Supplied Origin must match exact APP_URL scheme, hostname and port after trimming trailing slash.
localhost and 127.0.0.1 are different origins even when they reach the same machine.

The parser accepts at most 65536 bytes, depth 32 and scalar/null field values only.
It rejects [] as a top-level JSON array even though PHP decodes both objects and arrays into arrays.
Per-route allowlists then reject unexpected field names; privileged IDs/entity/user identity are fixed by server code.

| Operation | Method/path | Body shape |
|---|---|---|
| Student signup | POST /auth/register | name,email,student_id,faculty,password; no role |
| Password/factor login | POST /auth/login; POST /auth/2fa | email,password; code |
| Enroll/disable factor | POST /me/2fa/setup; POST/DELETE /me/2fa | {}; password,code |
| Apply/enroll | POST /clubs/{id}/memberships, /tournaments/{id}/registrations, /events/{id}/attendance | {}; route owns ID |
| Booking | POST /bookings | resource_id,starts_at,ends_at,purpose |
| Admin CRUD | GET/POST /admin/{entity}; GET/PUT/DELETE /admin/{entity}/{id} | Metadata fields; deletion {} |
| Review | PATCH /admin/memberships/{id} or /admin/bookings/{id} | status approved/rejected |
| Attendance | PATCH /admin/attendance/{id} | status attended |
| Own cancellation | DELETE /me/{collection}/{id} | {} |

Admin entities are clubs,tournaments,events,resources; personal collections are memberships,registrations,attendance,bookings.
PUT validates a complete editable record rather than arbitrary partial fields.
Community routes have their own approved-club/ownership rules in community.php; see the walkthrough.

| Status | Actual meaning to predict |
|---|---|
| 200/201 | Successful response / newly created resource |
| 400 | Malformed JSON or top-level object requirement |
| 401 | Sign-in required or failed/expired authentication challenge |
| 403 | Role/club denial, CSRF mismatch or supplied-Origin rejection |
| 404/405 | Unknown route/record or unavailable content / recognized route with wrong method |
| 409 | Integrity/state conflict, repeated cancellation or duplicate report |
| 413/415 | Body exceeds 64 KiB / unsupported Content-Type |
| 422 | Field/domain validation, including nested values, dates, capacity or overlap |
| 429 | Attempt throttle; Retry-After: 900 |
| 500/503 | Unexpected failure / unavailable database service |

Native CSRF uses 419, an application convention distinct from the REST 403 check.
A request failing an early gate never reaches a later parser/validator; test the intended boundary explicitly.
An API error envelope contains error.status/message and meta.request_id/csrf_token.
Do not copy full session/2FA responses into evidence because they can contain live secrets or tokens.

## 3. PDO, prepared values and trusted identifiers

PDO connects using the MySQL driver, utf8mb4, exceptions, associative fetches and native prepares.
The connection checks SELECT VERSION() before caching; MariaDB or a major version below 8 is rejected.
This implements the assignment's actual-MySQL requirement rather than treating a compatible brand as sufficient.

Actual source from registration in [actions](../../app/actions.php):

```php
query('INSERT INTO users(name,email,student_id,faculty,password_hash) VALUES(?,?,?,?,?)',[$name,$email,$sid,$faculty,password_hash($password,PASSWORD_ARGON2ID)]);
```

SQL structure is fixed first; five separate values fill five placeholders.
An apostrophe in a name remains data rather than ending a SQL string.
Do not concatenate a password/email into SQL then try to clean punctuation.
Placeholders represent values, not SQL table/column names: `SELECT * FROM ?` cannot bind a table.
Dynamic table/field fragments therefore come from entities(), fixed regexes or choice allowlists.
SQL binding and identifier allowlisting protect different boundaries; both are needed.

`e()` instead escapes HTML output using htmlspecialchars with quote handling and invalid UTF-8 substitution.
It is not a SQL defense and is not a general JavaScript-string encoder.
Plain-text community content remains untrusted when rendering; storing it in SQL does not make it safe HTML.

## 4. Authentication, authorization and CSRF

Authentication identifies an account; authorization decides what that account may do to a record now.
require_user checks user_id and fetches the current account, not a browser-supplied user ID.
require_admin adds a role check; ownership is repeated in cancellation WHERE clauses.
club_access adds approved membership in that exact club, or administrator permission.
The browser can display controls, but only these server rules authorize operations.

The cookie identifies private server state under var/sessions.
Strict sessions, HttpOnly, SameSite=Lax, private storage and authentication-time rotation reduce session risks.
Production sets Secure/HSTS and requires an HTTPS APP_URL; the host still needs a real certificate/HTTPS redirect.
After successful login finish_login replaces session state and generates a fresh CSRF token.
Thirty minutes of authenticated inactivity clears the session and rotates its identifier.

CSRF means forging a write through another person's authenticated browser cookie.
A random session token in legitimate forms/headers prevents an outside site guessing a valid write token.
Origin checking adds another boundary; SameSite cookies are defense in depth, not the only protection.
HttpOnly stops JS reading the session cookie, but malicious same-origin JS can still send requests.
That is why output escaping and CSP matter: XSS can undermine otherwise-correct CSRF defenses.
No passwords or authentication tokens belong in localStorage; this app stores theme preference there only.

## 5. Password hashes, TOTP secrets and the key

Passwords use Argon2id hashes; password_verify checks an input without recovering the original password.
Registration enforces 12–128 bytes with strlen; most ordinary text limits use mb_strlen characters.
Public signup inserts no role field, leaving the schema's student default intact.
CLI provisioning creates an administrator privately; no public setup endpoint should be exposed.

The server must recompute TOTP using the shared authenticator secret, so a one-way hash is insufficient for that secret.
It encrypts the secret with sodium secretbox and a private 32-byte APP_KEY, encoded as base64 in private config.
A fresh 24-byte nonce accompanies each ciphertext; nonce is public encryption input, not the encryption key.
Base32 makes the 20 random secret bytes convenient to type into an authenticator; it is encoding, not encryption.

The moving counter is floor(Unix seconds/30).
HMAC-SHA1 plus dynamic truncation produces six decimal digits, including leading zeros.
totp_counter accepts previous/current/next time steps and requires a counter newer than the stored accepted value.
A successfully used code cannot be replayed during that same interval; synchronize clocks and use a fresh interval.

An enrolled account's correct password creates only a five-minute pending_user/pending_until challenge.
It has no authenticated user_id and cannot enter protected portals.
Successful factor verification rotates into full authenticated state and fresh CSRF.
Setup enrollment is a separate ten-minute setup_secret/setup_until state requiring current password plus valid code.

verify_second_factor locks the user in a transaction before updating a counter or consuming a recovery-code hash.
Eight random recovery values are displayed once on enrollment and stored only as Argon2id hashes.
A matching recovery row is deleted before commit; concurrent requests cannot reuse it through the same user lock.
Disabling requires password plus a second factor. Its factor verification and later removal are not one enclosing transaction.
Never show authenticator seeds, recovery values, session IDs or APP_KEY in screenshots/logs.
Keep an existing dataset's key stable and recover it with its corresponding encrypted data.
A new random key after an update makes old TOTP secrets unreadable; it is not ordinary password rotation.

## 6. Transactions, isolation and parent locks

An invariant is a rule that must remain true, such as no two active bookings sharing a unit's interval.
A transaction groups writes so COMMIT makes them durable together and ROLLBACK undoes them together.
A unique constraint prevents duplicate relationships, but does not alone enforce aggregate capacity or interval overlap.

A naive check-then-insert can interleave: A sees no booking, B sees none, A inserts, B inserts.
Lock the shared resource parent before checking children, even when there is no booking row yet.
Current booking/reservation/review services request READ COMMITTED **before** beginning their transaction.
After waiting for the parent lock, fresh nonlocking reads see other committed approvals rather than an earlier repeatable-read snapshot.
review_request then locks the specific request, rechecks pending state/unchanged parent and checks capacity/overlap before update.
Community writes similarly use READ COMMITTED and lock the post before visibility-dependent child changes.

READ COMMITTED alone does not serialize contenders; the shared parent lock is also required.
A lock alone with an earlier consistent-read snapshot can still lead to a stale decision; isolation choice matters too.
Use consistent parent→request lock ordering to reduce avoidable deadlocks; errors still need rollback/retry policy.
The existing concurrency suite proves two specific approval races, not every possible schedule or deadlock.

Datetime-local values use Asia/Colombo in PHP; PDO sets SQL session timezone +05:30.
Mixing a new UTC conversion into existing forms without a contract can shift a reservation by 5½ hours.
Bookings intersect if existing.start<requested.end and existing.end>requested.start.
Strict comparisons allow intervals meeting exactly at an endpoint; pending and approved reservations block a new request.

## 7. Migration safety and deployment limits

A migration changes schema; it is not a reset or a second production database.
bin/migrate.php takes a database-name-derived advisory lock, creates missing base tables/history and applies sorted unapplied SQL files.
MySQL DDL can implicitly commit, so PHP rollback does not universally undo a partly applied schema migration.
The shipped CREATE TABLE IF NOT EXISTS statements are retry-safe; future ALTER/data migrations need their own plan.
Do not rewrite an applied version's meaning or run fixture resets against valued data.
Use separate reviewed migration privileges and runtime DML privileges where the host supports them.

Optional 2FA, no email verification/reset/SSO, file sessions and basic DB throttles remain explicit product limits.
The throttle count+insert is not an atomic distributed limiter; proxy REMOTE_ADDR and edge controls need deployment review.
Current PDO has no explicit remote-TLS/CA options; do not claim remote MySQL encryption is already configured.
Real hosting additionally needs public-only webroot, persistent private state, HTTPS, protected errors/logs and verified backups/restoration.
A working local health check does not certify production readiness.

## Official references and a prediction check

Use these primary references to deepen the exact concept, then return to the corresponding SportsHub function.
External resources are references, not claims that a host/provider has been verified.

- [PHP language basics](https://www.php.net/manual/en/langref.php), [PDO prepared statements](https://www.php.net/manual/en/pdo.prepared-statements.php), [session security](https://www.php.net/manual/en/features.session.security.management.php).
- [PHP password_hash](https://www.php.net/manual/en/function.password-hash.php), [sodium secretbox](https://www.php.net/manual/en/function.sodium-crypto-secretbox.php).
- [MDN HTTP methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods), [HTTP status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status), [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch).
- [MySQL 8.4 isolation](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html), [locking reads](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking-reads.html), [implicit commits](https://dev.mysql.com/doc/refman/8.4/en/implicit-commit.html).
- [OWASP authentication](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html), [CSRF](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html), [SQL injection](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html).
- [RFC 6238 TOTP](https://www.rfc-editor.org/rfc/rfc6238), [RFC 4226 HOTP](https://www.rfc-editor.org/rfc/rfc4226).

Predict before opening exercises: why does missing CSRF produce 403 before malformed JSON can produce 400?
Why does a six-digit TOTP need to remain a string, and why cannot a foreign key replace authorization?
The answers require naming the actual gate, representation and relationship rule, not memorizing acronyms.
