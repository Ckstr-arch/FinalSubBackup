# Eight students, four work tracks

The team has **eight students**. The four stages are work/learning tracks, so a practical arrangement is **two people per stage**. Give each person their own bounded task, own branch and own evidence, with their partner reviewing the work. The pair shares a feature outcome, not a fabricated joint claim to baseline authorship.

Names, GitHub handles and accepted assignments are intentionally unfilled in the [contribution log](contribution-log.md). A preference poll can show overlapping interests; several votes by one person do not create extra team members or establish accepted responsibilities. Agree one primary task per person and record secondary reviews separately.

[Detailed build sequence](build-order-and-contributions.md) · [Contribution log](contribution-log.md) · [Setup](../README.md) · [Report/video](report-and-video.md)

## 1. Proposed eight-person split

**A–H are eight planning slots, not real assigned identities or claims of completed work.** Match them to actual people after the team agrees.

| Slot | Track | Primary responsibility | Main source sections | Own evidence |
|---|---|---|---|---|
| A | Stage 01: frontend | Semantic layout and shared PHP-rendered HTML, navigation labels and accessible structure | `app/layout.php`, `app/ui.php`, agreed markup branches in `app/views.php` | Own scoped markup/design change, explanation and keyboard/semantics observations |
| B | Stage 01: frontend | CSS/responsiveness/themes and browser interactions | `public/assets/app.css`, `app.js`, `theme-init.js` | Own scoped style/interaction change, desktop/mobile and both-theme evidence |
| C | Stage 02: student | Student screens/forms, club discovery, membership/history feedback | Student branches of `app/views.php` and `app/portal-views.php`, agreed form mapping | Own student-facing patch or substantive investigated walkthrough with actual observations |
| D | Stage 02: student | Student workflow rules and regression cases for membership/enrollment/bookings/community | Student actions in `app/actions.php`, relevant `app/models.php`/`app/community.php` sections, agreed test cases | Personally implemented/tested domain change or meaningful regression evidence; Stage 04 review |
| E | Stage 03: admin | Admin management/approval/report views and validation feedback | Admin branches of `app/views.php` and `app/portal-views.php` | Own admin UI change and tests of valid/invalid/future/past states as applicable |
| F | Stage 03: admin | Admin CRUD/decision/result rules and report/audit checks | Admin actions, `save_entity()`/`review_request()`, relevant queries/tests | Own scoped service/test investigation or fix, including role refusal and data impact |
| G | Stage 04: backend/database | Relational schema, data dictionary, migration/seed safety and query review | `database/`, `bin/migrate.php`, `seed.php`, `app/bootstrap.php` PDO/query sections | Own verified schema/query documentation or reviewed migration/test change; no invented original schema authorship |
| H | Stage 04: backend/API | Authentication/security, REST contract and API integration/verification | `app/security.php`, API/auth sections of `app/actions.php`, `app/api.php`, `public/api.php`, API generator/tests | Own scoped auth/API/test work and evidence; coordinate with the affected UI owner |

This preserves work for frontend, backend, database and API interests. Database/API knowledge should be reviewed across tracks: D and F need G/H review for SQL/security, and A/B/C/E review the resulting browser behavior. Do not create a separate production application or database for each slot. Each developer may run their own private local development installation of the one canonical application.

## 2. Start in parallel, then integrate one journey

1. **All eight:** run the existing project from the same agreed commit, read the stack/architecture, confirm local configuration works, and acknowledge the assisted baseline.
2. **A+B and G+H start together:** inspect/improve the shared UI shell while the backend pair explains/validates schema, PDO, sessions, CSRF, roles and login. Existing protections stay enabled.
3. **C+D and E+F connect the first journey:** student applies to a club → pending status → administrator approval → approved student access. A/B support presentation; G/H review data/API/security boundaries.
4. **Extend by feature:** schedules/tournament entries, scores/results, bookings, attendance/history/reports and existing community/2FA review. Use the [per-stage implementation order](build-order-and-contributions.md).
5. **All eight:** demonstrate a real personal result, review another person's patch/evidence, and prepare truthful report/video contributions. Assign one actual integrator; that role does not acquire everyone else's authorship.

Before a shared feature, agree the exact input names, REST method/path/body, PHP action/function, tables/status rules, success state and refusal cases. API is a PHP interface; it is not an unrelated extra application to build in a different language.

## 3. Small task ideas when time is short

The baseline already works. These are **candidate tasks, not work completed by this document**. A student must reproduce a need, agree scope, personally implement or investigate it, and test/explain the result. Recheck the current branch because another teammate may already have addressed a candidate.

| Slot | Possible bounded first task | Evidence to save |
|---|---|---|
| A | Inspect the Results table and improve its semantic caption/header associations where needed | Actual markup diff, browser accessibility-tree/keyboard observations and preserved results data |
| B | Investigate keyboard focus after failed REST form submission and improve the feedback interaction if a real issue is reproduced | Own reproduction, handler diff, keyboard observations and preserved CSRF/error/retry behavior |
| C | Add/test a Clubs Clear filters control after confirming it resets both `q` and `sport` | Own branch/diff and filtered/reset/mobile/theme observations |
| D | Trace membership pending/approved/rejected/left access and add a useful missing student regression or improve an observed workflow issue | Own missing-case rationale, actual inputs/results and service/data explanation |
| E | Align future-session attendance feedback with the existing backend refusal | Future registered, past registered and attended-row observations; server rule retained |
| F | Investigate/test one admin write's rollback/audit or approval invariant, or improve a genuinely observed admin-rule issue | Own transaction/query trace, agreed disposable-data checks and precise outcomes |
| G | Personally produce a verified dictionary for agreed core tables, including field types, keys, relationships, state transitions and affected queries | Source/schema references, original explanations and peer review; avoid private identity exports |
| H | Add the missing raw-request parser-negative tests identified in the build guide | Malformed/array/nested JSON, oversized-body and Content-Type cases reaching the intended guarded parser |

A documentation or review task should be substantive and accepted under the course policy; it is not a substitute for claiming implementation that did not occur. Choose coding/testing work appropriate to each member's responsibility and the assessment's expectations. None of these tasks guarantees a grade or a fixed number of commits.

## 4. How each pair should cooperate

- Both people study the same feature and agree two distinct tasks. One person should not copy the pair's entire work and submit it under both identities.
- Each uses their own authentic account and a scoped feature branch. Keep the pair's changes independently understandable; stage only actual reviewed paths.
- The partner gives a real review, reproduces at least one relevant observation and records actual feedback. Service/data changes also receive the affected cross-stage review.
- For shared dense files such as `app/views.php`, agree the exact page branch before editing. Keep formatting changes local to the intended section so another person's patch remains reviewable.
- Use real commit/PR/review links after they exist. Unchanged baseline code remains baseline, even if both partners studied it.
- One coordinated integration review checks the complete student/admin journey. Individual records describe actual individual work and shared review separately.

Use the manual Git examples and PR template in the [build guide](build-order-and-contributions.md), and enter real names/tasks only after agreement in the [eight-slot log](contribution-log.md).

## 5. Eight-person demonstration and report evidence

Every student should be able to explain one actual contribution or investigation, a relevant source excerpt, one test/observation and what help they used. Prepare eight speaking slots with flexible timing, not a script claiming all eight independently developed pre-existing modules. The final video may combine pairs demonstrating a shared flow while each person explains their own work.

The leader/designated submitter should check that all eight real people have accepted roles, actual evidence and accurate assistance disclosure. The current log intentionally contains no fabricated identities, dates, successful tests, contributions or approvals. Keep private credentials, factor values, cookies and real student records out of logs/screenshots.
