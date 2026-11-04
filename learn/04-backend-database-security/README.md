# 04 · PHP, MySQL, REST and security

[Learning home](../README.md) · [Start here](../00-start-here.md) · [Solo path](../solo-path.md) · [Source map](../source-map.md)

The backend decides whether a request may change data; the browser displays the outcome.
Your goal is to explain that decision from its HTTP input to the exact SQL row and response.
This track studies the **existing** SportsHub application; it does not create another backend.

The inspected baseline is `f455575` on `main` in `Ckstr-arch/FinalSubBackup`.
Examples below link to this standalone repository, not the earlier TestENV layout.
No learning exercise is precredited as completed student work.
Current execution results belong in [learning validation](../validation.md), not copied historical totals.

## Your route through this track

| Read in order | Learn to explain | Evidence you produce yourself |
|---|---|---|
| [Lessons](lessons.md) | PHP syntax, requests, sessions, PDO, authentication and security | Predict a request before executing it |
| [Source walkthrough](source-walkthrough.md) | Native and REST call chains; actual functions and branches | Annotate one real flow in your own words |
| [Data dictionary](data-dictionary.md) | All 17 tables, keys, states and migration history | Draw relationships and predict a row change |
| [Exercises](exercises.md) | Safe observation, parser tests, lock reasoning and recovery | Actual observations, focused diff and genuine review |

You should already recognize HTML forms, variables, conditions and loops.
When a PHP or HTTP term is unfamiliar, follow the short explanation in the lessons before continuing.
No Composer framework, Node build or Python test runner is required by the distributed application.
PHP 8.2+, required extensions and actual MySQL 8+ are required; MariaDB is deliberately rejected.

## H and G: separate tasks with shared understanding

You chose **H: API, authentication and security**.
G is the proposed database/query/migration track; a real person and accepted task remain human choices.
H should understand the schema without claiming to have authored it.
G should understand sessions and API identity without rewriting H's work.

| Area | Lead learning responsibility | Review across the boundary |
|---|---|---|
| Session, CSRF, auth actions, REST parser/contract | H | G for data; D for student effects; F for admin effects |
| Tables, keys, PDO connection, migrations, data dictionary | G | H for identity/privacy; D/F for behavior |
| Student membership/enrollment/booking/community rules | D with domain reviewers | G/H for SQL, authorization and HTTP contract |
| Admin CRUD, approval, result/report rules | F with domain reviewers | G/H for transactions, privileges and response behavior |
| Browser form mapping and error/focus handling | Frontend owner | H for method/payload/CSRF and affected domain owner |

These are review boundaries, not exclusive file ownership or retroactive authorship.
Agree exact functions, branches or assertions before editing dense shared files.
Use [the eight-person plan](../../docs/eight-member-plan.md) and [actual contribution log](../../docs/contribution-log.md).

## A practical learning order

1. Read the relational schema and identify required MySQL/extension support.
2. Follow private config, stable key and PDO initialization in bootstrap.
3. Follow migration setup, then session/CSRF/role helpers **before** protected writes.
4. Trace registration/login and the first membership → approval journey.
5. Study transactional reservations, ownership and cancellation rules.
6. Compare native HTML POST with REST parsing and JSON response behavior.
7. Review existing TOTP enrollment, pending login, replay and recovery handling.
8. Verify a bounded improvement with meaningful tests and truthful evidence.

This is an order for understanding or a reviewed rebuild exercise.
The working application already has these protections; never remove them while studying later lessons.
Tests should accompany each meaningful change, rather than be postponed until every screen is finished.

## First safe observation

Once your local app is running with its configured origin, use a public GET:

```bash
curl --fail --silent --show-error --output /dev/null --write-out '%{http_code}\n' http://localhost:8091/api/v1/health
```

Expected: printed HTTP status 200. The health endpoint returns JSON containing data.status=ok, but this command discards its body so no CSRF metadata is printed.
The command above uses Linux/macOS output syntax; on Windows use curl.exe with --output NUL.
Use your actual configured local port if it differs; do not expose the development server publicly.
Health checks a database request; it does not verify login, every feature or production hosting.
Avoid `curl -v` and exporting auth-session bodies as learning evidence: cookies/CSRF may be present.
This guide contains no password-bearing mutation commands.

## Finish when you can demonstrate these outcomes

- [ ] Explain native POST → shared service → 303 and REST JSON → same service → JSON.
- [ ] Distinguish session identity, role, ownership, club approval and CSRF.
- [ ] Explain prepared values versus allowlisted SQL identifiers.
- [ ] Predict the SQL state of a successful and rejected reservation/review.
- [ ] Explain password hashes, encrypted TOTP secrets, APP_KEY and one-use recovery codes.
- [ ] Name the exact disposable test database and every reset guard before using reset-capable suites.
- [ ] Present a personally understood, genuinely reviewed contribution and actual evidence.

A guide, a folder, a checked box or copying the existing code is not proof of independent authorship.
A failed investigation can still be meaningful work when its method, finding and uncertainty are recorded honestly.

## Limits you must be able to state

2FA is optional; password reset/email verification, institutional SSO and cross-device session revocation are absent.
File sessions suit a single durable instance; scaling needs a reviewed shared-session design.
PDO currently supplies no explicit remote database TLS/CA options.
A host requiring verified remote database TLS needs a reviewed connection adaptation; do not claim it is already enabled.
Local operation and tests do not certify production security or an externally accessible deployment.
See [security/hosting](../../docs/security-and-hosting.md) and [historical validation](../../docs/validation.md).
