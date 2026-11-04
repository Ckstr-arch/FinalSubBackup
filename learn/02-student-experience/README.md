# Stage 02 · Understand the student experience

A student clicks Apply, Register or Book. Your job is to explain the real journey from that control
through PHP to a MySQL row, and back to a truthful success state or useful refusal.
This is a study track around the existing application, not a second frontend or a module-copying task.

**Study roles:** C covers student screens/forms/feedback; D covers student workflow rules and regression
cases. These follow the [eight-member plan](../../docs/eight-member-plan.md); only accepted real work
belongs in the [contribution log](../../docs/contribution-log.md). H owns the auth/API/security review
boundary. Understanding the login screen does not make C/D the authors of the existing security code.

[Learning home](../README.md) · [Start here](../00-start-here.md) · [Solo path](../solo-path.md)

## Your first session · about 30 minutes

Do this in order. A useful first session ends with a correct trace, not a folder full of copied PHP.

1. **Minutes 0–5:** finish the setup checks in [start here](../00-start-here.md). Use the app's configured
   origin, whether native or optional Docker; do not switch localhost/127.0.0.1 or ports arbitrarily.
   If services are unavailable, continue the source-only route below rather than changing private config.
2. **Minutes 5–10:** open [public/index.php](../../public/index.php), then search
   [app/views.php](../../app/views.php) for `$page==='clubs'`. Find the GET filter and `action_button('join'`.
3. **Minutes 10–20:** read [the two membership transports](source-walkthrough.md#2-one-apply-button-two-transports).
   Write the exact form fields, REST method/path/body, PHP function and table on a private worksheet.
4. **Minutes 20–30:** if your development app is ready, use a synthetic student and a club with spare
   capacity. Apply once, observe pending status and the private-tab gate. Record only method/status and
   safe UI state. Do not expose cookies, CSRF, password or authenticator values in screenshots.
5. Explain aloud: **“The form requests membership. The server identifies the user, validates the
   request and stores pending status. An administrator must approve before private club access.”**

**Source-only fallback:** complete steps 2–3, predict the expected row/status and mark the observation
NOT RUN. You have learned a call chain, not executed a test. Return to [the first lab](exercises.md#lab-1--predict-before-clicking)
when local setup is ready. No reset test is necessary for this session.

## Read in this sequence

| Stop | Open | You should be able to explain |
|---|---|---|
| 1 | [Lessons 1–2](lessons.md#1-forms-read-state-or-request-a-change) | Fields, GET/POST and UI versus security ownership |
| 2 | [Lesson 3](lessons.md#3-membership-is-a-relationship-with-a-decision) | Pending/approved/rejected, reapply and leaving |
| 3 | [Walkthrough 2](source-walkthrough.md#2-one-apply-button-two-transports) | Both transports reaching the same service |
| 4 | [Lessons 4–5](lessons.md#4-an-entry-is-not-the-same-as-attendance) | Tournament/event separation and booking intervals |
| 5 | [Lesson 6](lessons.md#6-community-access-is-scoped-to-one-club) | Community ownership, moderation and privacy |
| 6 | [Exercises](exercises.md) | Predictions, focused changes and honest evidence |

Take one journey at a time. Read a short source section, predict, inspect the app, then explain any
mismatch. You do not need to understand the entire API router before tracing Apply to join.

## Source map for this track

| Actual file | Read these sections | Change review |
|---|---|---|
| [app/views.php](../../app/views.php) | Auth UI, clubs, schedule, tournaments, results, facilities, activity | C with A/B for presentation; H for field/action contract changes |
| [app/portal-views.php](../../app/portal-views.php) | Student dashboard, my_clubs, club tabs/community | C/D; E/F own its admin/moderation/audit branches |
| [app/actions.php](../../app/actions.php) | Join, enroll, booking and own cancellation | D with G/H; auth/2FA branches reviewed with H |
| [app/models.php](../../app/models.php) | `reserve()` and student impact of `review_request()` | D coordinates with F/G/H |
| [app/community.php](../../app/community.php) | Access, member writes, hidden/deleted behavior and REST mapping | D with F for moderation and H/G for security/data |
| [public/assets/app.js](../../public/assets/app.js) | `endpoint()`, submit enhancement, error/redirect behavior | B with C/D and H when routes/payloads change |
| [app/api.php](../../app/api.php) | Auth/session, enrollment, own collections and field allowlists | H; C/D verify student expectations |
| [database/schema.sql](../../database/schema.sql) | Users and student participation relations | G; no schema edits by guesswork |
| [community migration](../../database/migrations/20261017_club_communities.sql) | Post/comment/reaction/report relationships | G with D/F/H |

Some production files are densely written. Search by function/page branch rather than reformatting a
whole shared file to read it. [The full source map](../source-map.md) assigns surrounding responsibilities.

## Boundaries to keep clear

- Registration/login UI gathers input; H's server-side auth/session/CSRF/factor rules decide access.
- C/D study admin decisions as the other half of a student lifecycle; E/F own admin policy/presentation.
- A visible Register button cannot guarantee an eligible request after capacity, time or permission changes.
- Dashboard “Session registrations” counts attendance rows; it does not claim every row is attended.
- This app has individual tournament enrollment and generic club standings, not automatic team brackets.
- There is no email verification/password-reset flow to explain as if already implemented.

## If you are studying alone

Follow [the solo path](../solo-path.md). Read C then D's part, and use two browser profiles for your own
synthetic student/admin when practicing approvals. Learn the existing security boundary before crossing
into H's work; do not disable auth checks to simplify a demo. Keep one canonical runtime and one accepted
small task at a time. A source-only explanation can be valuable learning, but it is not a fake code diff.

## What “done” means here

- [ ] I can trace native and REST membership writes to the exact relation and user identity source.
- [ ] I can explain reapply/cancel, session versus tournament enrollment and registered versus attended.
- [ ] I can predict interval overlap, own cancellation and approved-only community access.
- [ ] I can distinguish hidden content from deleted content, and private reports from public comments.
- [ ] I completed an exercise or recorded a precise NOT RUN reason, without borrowing historic results.
- [ ] I can explain my own real work, review and assistance without claiming existing module authorship.

Start [lessons](lessons.md), then return to the [done rubric](exercises.md#done-rubric).
These pages document inspected source; execution evidence is tracked in [learning validation](../validation.md).
