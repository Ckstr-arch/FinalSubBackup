# Practical exercises · assemble with evidence

[Stage dashboard](README.md) · [Lessons](lessons.md) · [Source walkthrough](source-walkthrough.md) · [Solo path](../solo-path.md)

These are **human learning tasks**, not commands executed by producing this guide. Before every lab, write a prediction. Save only safe evidence: commit, route, synthetic scenario, command, outcome and explanation. Never save secrets, cookies, recovery/setup codes, real student rows or private dumps.

## Lab 1 · Level 1: classify the setup before running it

**Objective:** distinguish source, configuration, schema, accounts and process state. **Sources:** [setup](../../bin/setup.php), [migration](../../bin/migrate.php), [seed](../../bin/seed.php), [doctor](../../bin/doctor.php), [admin provisioning](../../bin/create-admin.php). **Prerequisites:** basic terminal navigation; no database is needed for this reading lab.

1. Draw five boxes: checkout, private config/key, database schema, account rows and running server.
2. Place clone, setup, migrate, seed, doctor, create-admin and `php -S` beside the box each affects.
3. Predict what exists immediately after clone and what a second setup run should preserve.
4. Explain why a sports seed can create four clubs without making any password work.
5. Compare the actual template and script guards to your predictions without opening a private `.env`.

**Expected outcome:** source alone is insufficient; setup creates config once, migration creates schema, seed creates optional sports rows, provisioning creates admin, and the server is a separate process. **Evidence:** your annotated map and one explanation in your own words. **Revert:** none; this lab changes nothing.

<details><summary>Hint and answer</summary>

Treat doctor as a prerequisite/readiness diagnostic rather than a fixture loader. A persistent key belongs with a dataset; a process can stop/restart without requiring a new key. Re-cloning does not transfer private state automatically.

</details>

## Lab 2 · Level 1→2: run a native vertical slice

**Objective:** operate one student/admin workflow and explain its dependencies. **Sources:** [local setup](../../docs/local-setup.md), [index](../../public/index.php), [actions](../../app/actions.php), [models](../../app/models.php). **Prerequisites:** compatible PHP, genuine MySQL 8+, fresh separate local database and privately configured account/key.

Follow the exact platform setup first, then from repository root:

```bash
php bin/setup.php
# Privately configure the generated .env if this is a new installation.
php bin/migrate.php
php bin/doctor.php
php -S localhost:8091 -t public public/router.php
```

Optional development sports seed and administrator provisioning follow the setup guide; this lab does not print credentials. Register a synthetic student and sign in as your own administrator in a separate private browser context.

Before applying, ensure at least one fictional club with spare capacity exists. A fresh migration creates no clubs: either create one through the administrator interface or run the optional sports seed in development mode following the setup guide. Keep existing wanted records.

1. Student browses a club and applies; predict the membership state and community access.
2. Administrator approves; predict which table/field changes.
3. Student refreshes the club community; verify approved access and safe feedback.
4. Explain the session-derived user ID, CSRF and parent-lock boundary.
5. Verify `/api/v1/health` and distinguish its proof from this workflow's proof.

**Expected:** pending membership initially denies feed access; approval persists and grants access for that exact club. **Save:** source links, actual statuses and synthetic screenshots with no factor/credential data. **Safety:** this is local synthetic data, never production; parent deletion may cascade.

**Revert:** stop only your PHP server with Ctrl+C. Keep private settings/key/database for further study. Use ordinary own-record cancellation if cleaning up agreed synthetic records; do not reset a wanted database.

<details><summary>Hint and answer</summary>

A club card link or theme state is not an authorization grant. Trace the membership query in community.php after approval, then explain why an approved membership in another club is insufficient.

</details>

## Lab 3 · Level 2: compare native POST and REST

**Objective:** prove progressive enhancement rather than assuming all buttons use the API. **Sources:** [app.js](../../public/assets/app.js), [actions](../../app/actions.php), [API](../../app/api.php). **Prerequisites:** Lab 2, disposable local accounts, browser DevTools; never export a secret-bearing HAR file.

1. With JavaScript enabled, inspect one membership form submission in Network. Record only method, path, status, field **names** and the fact CSRF/cookies are present; do not copy their values.
2. Follow the mapping to the shared action and reserve service. Predict a duplicate request's failure.
3. Using a separate agreed synthetic scenario, disable JavaScript through your browser's supported setting and submit the native form.
4. Observe 303 navigation/feedback and identify the same server service. Re-enable JavaScript afterward.
5. Explain one native/REST status difference without weakening either protection.

**Expected:** enhanced JSON and native URL-encoded POST reach shared business behavior, with appropriate transport handling. **Save:** a redacted flow diagram and observed result; a disabled-JavaScript screenshot alone is not enough. **Safety:** do not replay a potentially committed mutation blindly after a network failure; first inspect current state.

**Revert:** restore your browser setting, close the isolated test context, and undo only your own temporary source edit if you made one. No source edit is required.

<details><summary>Hint and answer</summary>

Native handle_action checks hidden CSRF then performs the action and redirects. REST parses JSON, checks header/origin/roles and adapts allowlisted inputs before calling shared actions. Frontend validation improves usability; it does not replace server enforcement.

</details>

## Lab 4 · Level 2→3: choose meaningful checks safely

**Objective:** classify test evidence and, only with proven isolation, run the appropriate suites. **Sources:** [tests/fixtures.php](../../tests/fixtures.php), [integration](../../tests/integration.php), [concurrency](../../tests/concurrency.php). **Prerequisites:** compatible PHP; optional reset suites additionally need the confirmed empty/disposable `sportshub_backup_test` on the intended MySQL server and appropriate grants.

First read the fixture guards and runner overrides. Name the actual server/schema and confirm no wanted data is present. If you cannot prove this, stop at the non-reset checks and record the reset suites as SKIPPED with the precise reason.

```bash
php tests/security.php
php tests/audit.php
git diff --check
```

Only after completing the disposable target preflight:

```bash
php tests/integration.php --confirm-test-reset
# Wait for that process to finish before starting this one:
php tests/concurrency.php --confirm-test-reset
```

**Predict:** which runner occupies 8092? Which uses independent database processes without an HTTP port? Why does the shared reset target prevent parallel execution? **Expected:** checks fail loudly on a broken invariant; successful output and exit status are retained accurately. **Save:** command/commit/environment, safe summary, failures/skips and your explanation of one assertion.

**Safety:** both reset suites destroy their test schema's prior rows, even though they refuse other names. Never weaken the guard or borrow a production dataset/account to make them run. **Revert:** no restoration of production is involved; synthetic test rows may remain in the designated disposable schema. Remove only your own temporary notes/files, and do not execute a broad cleanup.

<details><summary>Hint and answer</summary>

The integration runner starts its own 8092 server; concurrency controls two connections/locks. A syntax pass says code parses, not that an authorization refusal or booking collision happened. Historical counts are not fresh results.

</details>

## Lab 5 · Level 3: eight-person integration rehearsal

**Objective:** plan independently reviewable work around one shared feature. **Sources:** [eight-member plan](../../docs/eight-member-plan.md), [build guide](../../docs/build-order-and-contributions.md), [contribution log](../../docs/contribution-log.md). **Prerequisites:** agreement by real team members; this is a planning lab until a human adopts a task.

1. Give A/B, C/D, E/F and G/H distinct proposed patches or investigations for the membership journey.
2. Name the exact page/action/function region, input contract, required review and focused evidence for each.
3. Identify two patches touching the same dense file and agree edit/review order before either begins.
4. H explains parser/auth/CSRF expectations; the UI owner explains rendered/native behavior; G explains the relevant relation/invariant.
5. Draft a PR description and a personal explanation for actual work, leaving unknown identities/links/results unfilled.

**Expected:** a dependency/review matrix, not eight claims to the existing application. **Save:** accepted tasks only after agreement, personal diffs/tests/explanations as they actually occur, and real review feedback. **Safety:** use truthful personal Git identity; no fake commits, force pushes, blanket staging or remote changes merely to complete this lab.

**Revert:** an unused plan needs no Git operation. If abandoning a personal patch, inspect its diff and remove only your edits through the editor; preserve other contributors' work.

<details><summary>Hint and solo adaptation</summary>

If H is working alone, retain the same review questions and record that actual fact. Use [the solo path](../solo-path.md) to prioritize: one complete flow, required behavior checks, operational safety, then optional polish. A solo fallback cannot invent teammate contributions or replace the lecturer's team policy.

</details>

## Lab 6 · Level 3: production release and recovery simulation

**Objective:** reach a defensible go/no-go decision without live deployment. **Sources:** [hosting/security](../../docs/security-and-hosting.md), [report/video](../../docs/report-and-video.md), [validation](../../docs/validation.md). **Prerequisites:** understand private config, webroot, migrations and key/dataset pairing. No host purchase, credential entry or deployment is required for this paper exercise.

Draft a capability matrix: PHP/extensions/Argon2id, actual MySQL 8+, HTTPS, public root, private durable sessions/config, migration/provisioning access, database TLS, logs and backups. Mark unknown provider features **UNVERIFIED**; do not select a fictional working host.

Draw separate private application and public document-root paths. Explain why `.env`, bin/, tests/ and var/ must not be downloadable. Plan fresh production configuration/admin provisioning without sample users. Describe how you would test actual HTTPS health, roles, CSRF, CRUD, bookings, 2FA and mobile behavior using synthetic accounts after authorization.

For recovery, name a private encrypted backup location and a **new isolated scratch database**. Describe checking dump target-selection statements, restoring with the matching protected key, verifying counts/relations and safe 2FA, then recording evidence. Do not import over a valued database to rehearse. Explain how code rollback differs from restoring data, and what newer user work could be lost.

**Expected:** a release decision with unresolved blockers and a reversible review plan. **Save:** actual capabilities/evidence when later verified; release SHA/approver/date/live URL remain TODO until real. **Revert:** no runtime action was taken. A future scratch rehearsal needs a separate approved cleanup plan for its own scratch state only.

<details><summary>Hint and answer</summary>

Missing public-root control, ephemeral keys/state, unverified remote-DB TLS or an untested restore are no-go issues, not documentation inconveniences. Keeping the previous code archive is useful but does not prove it can safely read the newer schema.

</details>

## Lab 7 · Final evidence and oral check

**Objective:** turn genuine work into a truthful submission. **Prerequisites:** your own observed results from earlier labs or implementation work. Use [the report/video template](../../docs/report-and-video.md), not a scripted claim of baseline authorship.

- [ ] I can explain one native/REST→service→SQL flow without reading a canned paragraph.
- [ ] My personal contribution record names actual work, assistance, reviewed source and real evidence.
- [ ] Each required sports feature has implementation/test/demo evidence or a stated limitation.
- [ ] Actual screenshots use synthetic data and contain no credentials/factors/tokens.
- [ ] Real source, hosted application and video links are accessible to the intended lecturer.
- [ ] Release date, host, team roles and confirmed deadline are actual facts, not guesses.
- [ ] The final report separates inspected, executed/passed, failed, skipped and planned items.

**Oral questions:** Why does doctor success not establish deployment? What breaks if APP_KEY is lost? Why does running a reset suite need more than its reassuring schema name? How would you preserve a teammate's work during a conflict? What evidence shows the student's action persisted rather than just changing a button?

**Done means:** another person can follow your safe procedure, reproduce a stated result and understand what remains unfinished. Writing this checklist does not perform the release or submit the report.

## Official references for a question you can name

Use [the shared resource index](../resources.md) for the full list. These official pages are supporting learning material; provider plan availability and your actual host configuration remain unverified until checked.

| Question | Official resource |
|---|---|
| What does the PHP development server do? | [PHP built-in server](https://www.php.net/manual/en/features.commandline.webserver.php) |
| How are PDO values bound? | [PDO prepared statements](https://www.php.net/manual/en/pdo.prepared-statements.php) |
| How do PHP session settings work? | [PHP session security configuration](https://www.php.net/manual/en/session.security.ini.php) |
| What does a MySQL logical dump contain? | [MySQL 8.4 mysqldump](https://dev.mysql.com/doc/refman/8.4/en/mysqldump.html) |
| What does Git record and how do branches work? | [Pro Git](https://git-scm.com/book/en/v2) |
| What is an actual pull request? | [GitHub pull requests](https://docs.github.com/en/pull-requests) |
| What determines Apache's public root? | [Apache DocumentRoot](https://httpd.apache.org/docs/2.4/mod/core.html#documentroot) |
| Why does Nginx need its own PHP configuration? | [Nginx FastCGI module](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html) |
| How do Compose project names and volumes work? | [Compose project name](https://docs.docker.com/compose/how-tos/project-name/) and [Docker volumes](https://docs.docker.com/engine/storage/volumes/) |
