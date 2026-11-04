# Stage 03 · Admin operations

[Learning home](../README.md) · [Start here](../00-start-here.md) · [Solo route](../solo-path.md) · [Source map](../source-map.md)

The sports office makes decisions that affect students: approve a membership, publish a fixture, record attendance or hide a reported post. This stage teaches the difference between showing an administrator a control and safely applying their decision on the server.

**Outcome:** explain and test one complete administration journey from form fields through permission checks and SQL to the student's resulting view. Then make one bounded improvement you personally understand.

The existing application is a Codex-assisted baseline. These lessons describe existing code; receiving the lessons or copying a module does not establish your authorship. Record actual study, implementation, debugging, testing and review in the [contribution log](../../docs/contribution-log.md).

## Open these four pages in order

| Page | What you will do | Evidence you can produce |
|---|---|---|
| [Lessons](lessons.md) | Learn CRUD, approvals, locking, scores, reports and moderation | A prediction table and an explanation in your own words |
| [Source walkthrough](source-walkthrough.md) | Follow actual PHP branches and two request paths | A source-linked browser → service → SQL trace |
| [Exercises](exercises.md) | Inspect, test and propose a small reviewed improvement | Synthetic before/after evidence and actual results |
| This dashboard | Check boundaries, prerequisites and progress | A realistic next task, without claiming baseline authorship |

## Who coordinates this work?

The proposed eight-person split uses **E for admin screens** and **F for admin services**. These are responsibility labels, not names or proof of completed work. Confirm actual assignments with the team.

| Boundary | E: screens and explanation | F: services and rules | Required coordination |
|---|---|---|---|
| Management forms | `manage_*` branches and error/empty states | `entities()` and `save_entity()` behavior | A for shared PHP/layout; B for CSS/browser UI |
| Membership/booking review | Pending queues, status and decision controls | `review_request()` and transactional checks | C/D for student consequences; G for data relationships |
| Fixtures/results | Clear date/score fields and displayed results | Event validation and persisted paired scores | G for schema; H for REST/auth/permission contracts |
| Attendance/reports | Availability feedback, understandable count labels | Start-time checks and report query meaning | C/D for participation history; H for endpoint parity |
| Community moderation | Private queue, reason and decision feedback | `moderate_post()` and content-state invariants | C/D for member experience; H for privacy/authorization |

Neither person owns an entire shared file exclusively. Before editing, name the exact function or page branch, agree the changed behavior and reviewer, and inspect the current diff together. One canonical runtime remains in `app/`, `public/` and `database/`.

A solo learner follows both columns one journey at a time. The [solo route](../solo-path.md) explains how to narrow the workload instead of creating fictional teammates.

## What already exists?

- Separate admin overview, navigation, CRUD forms, approvals, reports, moderation and audit views.
- CRUD for clubs, tournaments, training/match events and facility/equipment resources.
- Pending membership/booking review under parent/request locks and READ COMMITTED isolation.
- Atomic administrative save/delete with their audit writes.
- Match-only paired scores, public results and generic club standings.
- Attendance recording after session start and reports derived from actual stored rows.
- Private community reports, reversible hide/restore and irreversible content deletion.

These are source-inspection facts at `f45557509e0202d4c93411db3a1237f44d9082ae`. They are not a claim that every exercise below was freshly executed. See [learning validation](../validation.md) and [application validation](../../docs/validation.md) for recorded checks and their limits.

## What you need first

1. Understand basic HTML forms, PHP arrays/conditions/functions and SQL SELECT/UPDATE.
2. Complete the [application setup](../../README.md), using actual MySQL 8+ and the configured exact origin.
3. Use a disposable local administrator and student, with synthetic records only.
4. Read [source ownership](../source-map.md) and agree a task before changing shared code.
5. Know that a destructive test runner is different from a harmless source-inspection exercise.

You do not need React, a PHP framework, Node or a second runtime. Native PHP and optional Docker run the same application.

## Ten-minute first inspection

From the repository root, inspect the guards and services:

```bash
rg -n 'require_admin|function entities|function save_entity|function review_request' public/index.php app/api.php app/actions.php app/models.php
```

If `rg` is unavailable, use your editor's search. Predict why sending `action=save` from a student's dashboard cannot grant admin privileges. Then locate the server guard that rejects it.

Expected: page and API guards protect reads; the shared action path checks the administrator role before core administrative writes. The early moderation dispatch checks the role inside its own service. No code or database change is needed.

## Progress dashboard

- [ ] I distinguish authentication, authorization, validation and CSRF.
- [ ] I can find all four CRUD entity descriptors and their actual inputs.
- [ ] I can trace native create and REST approval to a committed row.
- [ ] I explain why a pending request must be reread after waiting for a lock.
- [ ] I distinguish tournament enrollment, event registration and recorded attendance.
- [ ] I can calculate report/standings values without inventing analytics.
- [ ] I explain hide, restore, deletion and foreign-key cascade consequences.
- [ ] I tested an allowed and denied case safely and recorded actual evidence.
- [ ] My proposed or actual patch has a narrow shared-file review boundary.

## Suggested first three tasks

**E:** inspect a Reports attendance row; reproduce the future-session control mismatch with synthetic data; propose accessible availability feedback for review.

**F:** trace the same request to the start-time guard; identify its permitted/rejected states and transport statuses; review E's patch and preserve direct-request rejection.

These are proposals, not assigned or completed contributions. A useful result can be a genuine test, investigation, reviewed design or documentation improvement as well as a source change. The lecturer decides what satisfies assessment policy.

Continue with [Lessons](lessons.md). Keep the [official reference list](../resources.md) nearby when a term is unfamiliar.
