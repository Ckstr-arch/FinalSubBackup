# Eight-student start → finish guide

Eight students total; four tracks with two people each. **You chose H: REST API, authentication and security.** Names/GitHub handles and work evidence remain to be filled in the [contribution log](contribution-log.md); the other role allocations are proposals until accepted.

Use the working application as the disclosed assisted baseline. Each person agrees one scoped task, learns the relevant flow, makes or investigates a real change, tests it, and explains their own result.

## A–H: where to start and where to stop

| Part | Start here | Work in this order | Finish when |
|---|---|---|---|
| **A — shared HTML/PHP layout** | [layout.php](../app/layout.php), [ui.php](../app/ui.php) | Semantic HTML → header/sidebar/navigation → shared PHP template/helper → labels and keyboard structure | Shared pages render with correct role navigation and accessible structure; keep domain rules with D/F/H |
| **B — CSS and browser UI** | [app.css](../public/assets/app.css), [app.js](../public/assets/app.js), [theme-init.js](../public/assets/theme-init.js) | Tokens/spacing → Grid/Flex → responsive rules → themes/menu/focus JavaScript | Agreed component works on desktop/mobile, both themes and keyboard; coordinate API mappings with H |
| **C — student screens** | Student branches of [views.php](../app/views.php) and [portal-views.php](../app/portal-views.php) | Lists/cards → search/filter/forms → PHP data/status display → success/error/history feedback | Student can operate the agreed journey and see its real state after refresh; D/H enforce the rules |
| **D — student services** | Student branches of [actions.php](../app/actions.php), [models.php](../app/models.php), [community.php](../app/community.php) | Input validation → session owner → membership/enrollment/booking rules → prepared SQL/transaction → regression cases | Agreed student action persists correctly and invalid/duplicate/other-user actions are refused; G/H review shared boundaries |
| **E — admin screens** | Admin branches of [views.php](../app/views.php) and [portal-views.php](../app/portal-views.php) | Management lists → create/edit forms → approvals → results/report/attendance feedback | Admin can use the agreed screens and see accurate validation/status; F/H enforce permission and writes |
| **F — admin services** | `save_entity()`/`review_request()` in [models.php](../app/models.php), admin [actions](../app/actions.php) | CRUD validation → atomic write/audit → approvals → scores → attendance/report rules | Agreed CRUD/decision works, bad or unauthorized input fails, and database/audit state agrees with the UI |
| **G — MySQL and data layer** | [schema.sql](../database/schema.sql), [migration](../database/migrations/20261017_club_communities.sql), PDO/query sections of [bootstrap.php](../app/bootstrap.php) | Relationships/keys → private MySQL config/PDO → migration → seed/query review → data dictionary/tests | Actual MySQL/schema/queries are verified, needed changes are additive, and valued data is preserved; H reviews auth/API impact |
| **H — your part: API/auth/security** | [API guide](api.md), then [public/api.php](../public/api.php) | Request/error boundary → sessions/CSRF/roles → login/2FA → payload/routes/shared service → focused API tests | Agreed requests return correct safe JSON/status, access checks hold, and you can explain and demonstrate your own reviewed work |

These are handoff checkpoints, not claims that anyone newly authored or completed the existing modules. Shared files are divided by exact function/page branch; agree before editing the same section.

## Your H order — follow this first

1. **Read the contract:** [docs/api.md](api.md). Understand GET/POST/PUT/PATCH/DELETE, cookie sessions, CSRF, JSON and status codes.
2. **Read the entrypoint:** [public/api.php](../public/api.php). Trace setup, dispatch, safe errors and request IDs.
3. **Read authentication helpers:** session/CSRF/roles in [bootstrap.php](../app/bootstrap.php), [security.php](../app/security.php), and register/login/verify/2FA branches in [actions.php](../app/actions.php). Keep the working protections intact; agree PDO/schema edits with G.
4. **Trace one request:** follow [app/api.php](../app/api.php) → shared action/service → MySQL → JSON. Start with login/session, then membership application. Check the matching browser mapping in [app.js](../public/assets/app.js) with B/C.
5. **Take one bounded first task:** on your own branch, add missing raw-request parser tests in [tests/integration.php](../tests/integration.php). Read `api_payload()` before writing expectations. This is proposed work for you; the guide does not implement it.
6. **Test and hand off:** use the [setup/test safety instructions](local-setup.md) and [detailed build guide](build-order-and-contributions.md). Save actual outcomes, obtain peer review and explain the flow in your own words before claiming completion.

### A useful first H task

Use `VerificationClient::request()` for raw bodies: its `api()` helper always encodes a valid object. Fetch `/api/v1/auth/session`, keep the same cookie and current CSRF header, then exercise POST `/api/v1/auth/login`:

| Raw request case | Existing parser expectation |
|---|---|
| Malformed JSON | 400 |
| Top-level JSON array `[]` | 400 |
| A nested object/array field | 422 |
| Body larger than 65,536 bytes | 413 |
| Unsupported Content-Type | 415 |

Without valid CSRF you may receive 403 before reaching the parser, so that would not prove the intended case. Check the safe error envelope as well as status. Keep credentials/tokens out of screenshots/logs. Security checks need a valid private application key; integration/concurrency suites reset **only the deliberately disposable `sportshub_backup_test` schema**, require the confirmation flag and must run sequentially. Provision and verify that test schema first; never use wanted data.

**Your first-task finish line:** a personally understood test diff, actual results (or a clearly recorded blocker), review by the affected owner, and a real commit/PR/evidence entry. Larger API/security changes need their own agreed scope.

## Shared order and contribution handoff

Start **A+B and G+H in parallel**, then connect **C+D and E+F** around:

**Register → login → student applies → admin approves → student sees approved access.**

Each student finishes their task with: **own branch → actual work → focused checks → reviewed PR → genuine contribution entry → one-minute explanation**. Use the [eight-member plan](eight-member-plan.md) for responsibility boundaries and the [longer guide](build-order-and-contributions.md) when a step needs more explanation. Existing working features remain in the one canonical application.
