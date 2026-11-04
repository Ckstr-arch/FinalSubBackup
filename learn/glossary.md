# Project glossary

[Learning home](README.md) · [Source map](source-map.md) · [Resources](resources.md)

| Term | Plain meaning in SportsHub |
|---|---|
| HTML | The structure and meaning of headings, navigation, forms, tables and content returned to the browser |
| CSS | Rules that control layout, color, spacing, responsive behavior and themes |
| JavaScript | Browser code that manages interactions and enhances forms with `fetch`; server rules still apply |
| PHP | Server language that reads requests, executes application rules, queries MySQL and produces HTML or JSON |
| MySQL | The required relational database engine; this edition checks the engine/version and rejects MariaDB |
| HTTP | The request/response protocol between browser/client and server |
| HTTPS | HTTP over TLS, protecting traffic in transit and authenticating the server through certificates |
| Request / response | A client's method/URL/headers/body and the server's status/headers/body |
| Document root | The directory a web server makes public; only this repository's `public/` belongs there |
| Entrypoint | The PHP script first executed for a request: usually `public/index.php` or `public/api.php` |
| Router | Code or web-server rules selecting the entrypoint for a URL; `public/router.php` is development-only |
| Template | PHP/HTML that renders a page using current data, such as branches in `app/views.php` |
| Function | Named reusable behavior with inputs and a return value or side effect |
| Associative array | PHP mapping from keys to values, such as a database row's `id`, `name` and `status` |
| Superglobal | PHP request/environment arrays such as `$_GET`, `$_POST`, `$_SERVER` and `$_SESSION` |
| PDO | PHP's database interface used here with the MySQL driver |
| Prepared statement | SQL structure prepared separately from bound values; stops data being interpreted as SQL syntax |
| Allowlist | An explicit set of allowed routes, identifiers, fields or choices; used when values cannot safely select arbitrary structure |
| Primary key | Stable row identifier, usually `id` |
| Foreign key | Database relationship constraint connecting a child row to an existing parent |
| Unique constraint | Database rule preventing repeated values or pairs, such as one membership per user/club |
| Join | SQL that combines related rows, not the same as a student joining a club |
| CRUD | Create, read, update and delete operations |
| Transaction | A group of database operations committed together or rolled back; MySQL DDL has important auto-commit exceptions |
| Row lock | A database mechanism that serializes competing changes; this app locks parent rows before capacity checks |
| READ COMMITTED | Transaction isolation used in reservation/review flows so later checks can see other committed writes |
| Race condition | Incorrect behavior depending on timing, such as competing approvals both seeing a last free place |
| Idempotent | Repeating the same intended operation leaves the same desired state; reaction PUT is an example, arbitrary create POST is not guaranteed |
| Migration | An intended database schema change tracked by filename in `schema_migrations` |
| Seed | Optional fictional development sports data; not a production backup or an administrator account |
| Authentication | Establishing who the user is through password and, if enrolled, a second factor |
| Authorization | Checking what that user may do or read, including role, ownership and exact-club access |
| Session | Server-side state referenced by an opaque browser cookie; here it includes identity, CSRF and pending-factor state |
| HttpOnly | Cookie flag preventing browser JavaScript from reading that cookie; it does not prevent every XSS consequence |
| SameSite | Cookie control limiting some cross-site sending; it complements rather than replaces CSRF validation |
| Secure cookie | Cookie sent over HTTPS; production settings enable this |
| CSRF | Cross-site request forgery: unwanted state-changing requests using a browser's existing credentials |
| Origin | Scheme, host and port identifying the request's source; exact configured match is checked when mutation Origin is supplied |
| XSS | Untrusted text executing as browser code; escape user text for the actual output context |
| Escaping | Encoding output for its destination; `e()` uses HTML escaping for normal HTML contexts |
| Validation | Checking type, shape, length, range and business rules; performed on the server even when the browser also validates |
| Hashing | One-way transformation used for password/recovery verification; not encryption that can be decrypted |
| Argon2id | Password-hashing algorithm used by the application; supported PHP builds are required |
| Encryption | Reversible protection requiring a secret key; TOTP seeds use sodium secretbox and `APP_KEY` |
| TOTP / 2FA | Time-based one-time code as a second authentication factor; a seed is enrolled per account |
| Replay protection | Rejecting a factor counter already used successfully |
| Recovery code | One-use factor alternative shown once and stored as a hash |
| REST API | The HTTP/JSON interface to the same PHP services, under `/api/v1`; not another backend language |
| JSON object | Named fields in `{...}`; this API rejects an array `[...]` as its top-level mutation payload |
| Middleware | Checks around dispatch, such as method, session, Origin, CSRF and payload validation; no external framework is required here |
| Progressive enhancement | Keep ordinary HTML/native forms functional, then add browser behavior where available |
| PRG | Post/Redirect/Get: successful native writes redirect with 303 before the browser loads the next page |
| Audit log | Persisted record of an action; useful evidence but not a backup or automatically a full immutable forensic record |
| Tombstone | Retained record indicating a deleted community post; body and children are erased and cannot be restored |
| CLI | Command-line execution; private provisioning/migration tools should not become public web setup endpoints |
| Environment variable | Process configuration value; the local `.env` loader only fills names not already set |
| Container / image | Isolated running process environment / its packaged filesystem and startup definition |
| Volume | Docker-managed persistent state separate from a replaceable container; deleting it can erase wanted data |
| Staging | A separate production-like environment for acceptance checks with synthetic data |
| Production | Live service for actual users, requiring verified operational configuration and recovery |
| Smoke test | A small check that critical live paths work; it does not prove all business rules or security |
| Regression test | A repeatable check preventing a known behavior from breaking after a change |
| Integration test | A check across components such as HTTP, PHP sessions and an actual test database |
| Concurrency test | A check involving independent connections/timing, needed to expose transaction races |
| Branch / commit / PR | A development line / saved change / review proposal in Git and GitHub |
| Baseline | The existing assisted working application from which your actual work starts |
| Vertical slice | One user outcome implemented/traced across UI, server rules, database and response |

When a word is still unclear, find its actual source example in the relevant stage. Definitions are a starting point; explaining a real request is the stronger check.
