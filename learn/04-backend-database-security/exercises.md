# Exercises · predict, observe, explain, then contribute

[Dashboard](README.md) · [Lessons](lessons.md) · [Source walkthrough](source-walkthrough.md) · [Dictionary](data-dictionary.md)

These labs are instructions for your own future learning and contribution, not completed work credited to you.
Use the existing application and synthetic data. Do not replace it with a new framework or weaken a check to make a demo pass.
The hints are separated from the tasks; safety requirements apply before opening any answer.

## First: classify commands and protect state

Read-only source checks and syntax checks can run without a database reset:

```bash
php -l app/bootstrap.php
php -l app/api.php
php -l app/security.php
php tests/audit.php
```

`php tests/security.php` uses public vectors/synthetic cryptographic material without SQL access/reset.
It needs the required extensions/Argon2id and a valid private APP_KEY already configured or privately supplied.
Do not print/generate a replacement key over an existing dataset to satisfy a test.
Doctor queries the actual configured DB/schema and checks private storage; it is not a reset or full feature suite.

The following are **reset-capable** commands, not routine first inspections:

```bash
# Future learner execution only after every isolation check below passes:
php tests/integration.php --confirm-test-reset
php tests/concurrency.php --confirm-test-reset
```

Both require CLI and the explicit flag, force DB_NAME=sportshub_backup_test and APP_ENV=development,
and verify actual selected database/engine before rebuilding tables through fixtures.
The integration server uses port 8092 and refuses to replace an existing service on that port.
The concurrency suite uses independent PHP/MySQL connections to establish real lock waits.
Run them sequentially because they rebuild the same test schema; never keep wanted records there.

Before either reset command, personally inspect runner/fixtures and confirm the selected server/account,
actual SELECT DATABASE() target, actual MySQL 8+ engine, and absence of valued data in that exact test schema.
Do not display credentials or full process environments as “proof”. If separation cannot be established, skip and state why.
Do not run fixtures as a public setup endpoint or substitute the working database name to bypass a failure.
These docs do not authorize production resets, volume removal, deployment or GitHub writes.

## Lab 1 · Read a response before changing anything

**Objective:** separate service health from claims of full readiness.
**Prerequisites:** local app running at its exact configured APP_URL; read the health branch in app/api.php.
**Sources:** public/router.php,public/api.php,app/api.php,bootstrap db.

On Linux/macOS, request health without printing its response metadata:

```bash
curl --fail --silent --show-error --output /dev/null --write-out '%{http_code}\n' http://localhost:8091/api/v1/health
```

On Windows PowerShell use curl.exe and output NUL instead of /dev/null.
Use your real local configured port; this is a public GET, not a password-bearing request.
Predict 200 when SELECT 1 succeeds, and explain why an unavailable engine/config can produce an error.
Expected evidence: status, local environment and named source branch, with no cookies or CSRF values.
**Guiding question:** does this prove registration, administrator role enforcement, backups or HTTPS hosting?
**Undo:** no source/domain changes; ordinary HTTP may create a local anonymous session file.

## Lab 2 · H's first real contribution: raw parser-negative tests

**Objective:** add meaningful missing HTTP assertions for the existing input boundary.
**Prerequisites:** read api_dispatch→api_payload, VerificationClient and all isolation guards above.
**Sources:** app/api.php,tests/integration.php,tests/fixtures.php; do not rewrite the parser simply to create activity.

### Level 1 — understand and predict

Use this expected-results table before writing any assertion:

| Raw request condition | Expected status | Why it reaches that result |
|---|---|---|
| Content-Type text/plain |415| Media-type gate fails first |
| Content-Type application/json with malformed body `{` |400| JSON decoding throws |
| JSON top-level array `[]` |400| Object opening-brace requirement fails |
| JSON object containing a nested object/array field |422| Only scalar/null values are allowed |
| Body exceeding 65536 bytes |413| Bounded reader sees one extra byte before decode |
| Correct JSON `{}` to auth/login |422| Parser accepts object; required email validation then fails |

Confirm these raw parser cases are still missing before editing, since another member may add them first.
The existing suite already covers missing CSRF, foreign Origin, role escalation and unknown replacement fields;
relabeling those existing assertions as your newly implemented cases is not a real contribution.

### Level 2 — implement your scoped tests manually

Agree the tests/integration.php section with D/F/H reviewers before parallel edits.
Use VerificationClient::request for malformed raw input: api() always JSON-encodes a valid object.
First bootstrap GET /api/v1/auth/session through the same client so its cookie and CSRF belong together.
Then send raw POST /api/v1/auth/login with valid X-CSRF-Token and appropriate Content-Type.
If Origin is supplied, use the test runner's exact APP_URL, http://localhost:8092.

Missing CSRF yields 403 **before** parsing; a protected route may fail 401/403 before reaching the body check.
An early denial is not proof your malformed JSON assertion exercised api_payload.
Do not log the token or whole auth/session response; record route, intended branch, expected/actual status and safe error text.
Use synthetic body strings and assert the error envelope matches status without SQL/stack-trace disclosure.
Do not add a test helper that prints secret-bearing request/response headers.

The intended outcome is a test-only diff demonstrating the current server contract.
Run lint and the reviewed isolated integration command only after proving the destructive target is disposable.
Retain the actual exit status/count and investigate failures; do not copy historical totals as your own run.
A valid test should fail if its corresponding boundary is removed; explain that counterfactual in review.

### Level 3 — explain edge cases and trade-offs

Why read 65537 bytes to enforce a 65536-byte limit?
Why does malformed JSON have a different status from valid JSON with a nested value?
Why can browser maxlength or a frontend JSON serializer not replace these server checks?
Why should an invalid-body test avoid creating a real account or relying on login throttling?
Consider a boundary-size assertion only if it proves a useful distinct behavior and remains within the agreed scope.

**Evidence:** your prediction table, own focused diff, actual isolated result and peer feedback.
**Undo:** before editing keep a private before-copy outside public/; manually remove only your added lines after reviewing your diff
if the exercise is temporary. Do not reset/clean shared work. Retain accepted tests only through truthful reviewed integration.
Your legal name, GitHub handle, completion date and PR remain TODO until supplied/performed by you.

## Lab 3 · G explains relationships with a read-only query

**Objective:** predict why a LEFT JOIN retains clubs with no approved members.
**Prerequisites:** read memberships,clubs and the dictionary's two sample queries; use synthetic local data only.
**Sources:** database/schema.sql and report queries in app/views.php/app/api.php.

On paper predict the report for one club with zero requests, one with pending only and one with approved memberships.
Run only the read-only aggregate query on your approved local study dataset; do not export identities.
Move the status predicate from JOIN ON to WHERE **on paper**, then explain which rows disappear.
Expected evidence: a three-row prediction/observation table and your own explanation of COUNT(m.id) versus COUNT(*).
Challenge: find the relevant index/FK metadata and propose an EXPLAIN investigation without inventing performance numbers.
**Undo:** no source or SQL data mutation. Remove only your own scratch query note if desired.

## Lab 4 · Booking/approval concurrency without damaging data

**Objective:** explain why both isolation and locking matter.
**Prerequisites:** lessons on READ COMMITTED; inspect review_request and tests/concurrency.php.
**Sources:** models review_request/reserve, actions book, bookings/resources/memberships/clubs.

Write an interleaving table: A holds parent lock, B reads candidate and waits, A approves/commits,
B acquires parent then request lock, B reads current count/overlap and refuses an invalid competing approval.
Now draw the earlier repeatable-read snapshot possibility and explain the function's transaction-specific isolation change.
A paper trace is sufficient for first learning; never remove locks in the real app to create a demonstration failure.
Optional execution is the guarded concurrency command only after every isolation gate passes.
Expected result: the test's actual controlled races preserve one last-slot approval and one overlapping approved booking.
Do not claim this proves every deadlock schedule or distributed deployment.
**Evidence:** named lock order, invariant, actual result if run, or “not run; reasoning by source” if not.
**Undo:** paper-only work changes no state; reset suite leaves synthetic rows only in its intentionally disposable target.

## Lab 5 · Explain factor state without recording a secret

**Objective:** distinguish password success from fully authenticated 2FA access.
**Prerequisites:** use a disposable account you control; read security/actions and have private factor/recovery storage.
**Sources:** login/verify/enable_2fa,verify_second_factor,finish_login,bootstrap require_user.

Predict session-key names in anonymous, pending and authenticated states; never write their actual values.
Observe that pending login cannot reach /me or the rendered dashboard until factor completion.
If enrolling a disposable account privately, do not record the setup/recovery screen or code entry.
Explain a same-step replay refusal and a consumed recovery code using source/control-flow evidence.
Challenge: plan recovery when DB ciphertext and APP_KEY come from mismatched backups; do not rotate a live key as an experiment.
Expected evidence: state diagram, safe status observations and your own explanation of encryption versus hashing.
**Undo:** optionally disable 2FA through the supported password+factor UI on that disposable account; no direct secret SQL edits.
If you cannot protect the account/key/recovery artifacts, keep this lab read-only.

## Hints and answer checks

<details><summary>Open after writing predictions</summary>

Lab 1: health checks `SELECT 1` after connection, engine and timezone initialization; it cannot prove full feature/security/host acceptance.
Lab 2: middleware first chooses route/method and role, then CSRF/Origin, then body parsing; valid session continuity is essential.
The extra read byte distinguishes exactly-at-limit from too-large; scalar restriction is different from JSON syntax validity.
Lab 3: LEFT JOIN keeps unmatched parent clubs; WHERE on the child rejects their NULL rows, effectively changing the result.
Lab 4: current committed state after waiting plus a common parent serialization point protects aggregate decisions.
Lab 5: pending_user is not user_id; factor verification persists the accepted counter or consumes one hash under a user lock.
A key/data mismatch cannot be repaired by changing CSS or waiting for the next TOTP step.

</details>

## Self-check rubric: readiness for a real review

Score yourself: 0=unsupported, 1=partly explained, 2=demonstrated with correct source/evidence.
This is a learning rubric, not a course mark or claim that a student passed.

| Criterion | What a 2-point explanation includes |
|---|---|
| Request trace | Correct entrypoint, middleware, service, table and response |
| Trust boundaries | Distinguishes auth, role/ownership, CSRF, JSON validation and escaping |
| Data correctness | Names exact key/lock/state rule and a failure/rollback case |
| Security state | Explains pending login, counter/recovery and stable private key |
| Verification | Proves isolation, uses meaningful assertions and records actual outcome/skips |
| Contribution integrity | Own scoped work, real review, own explanation and honest assistance disclosure |

A maximum self-score does not certify production security.
Before asking for review, explain one central excerpt aloud without claiming you wrote the pre-existing module.
Record genuine implementation/debugging/tests/docs work in [the contribution log](../../docs/contribution-log.md).
Leave person/date/commit/PR/result/reviewer unknowns unfilled until they actually exist.
