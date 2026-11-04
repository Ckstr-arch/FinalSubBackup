# Source walkthrough · Follow the real admin request

[Dashboard](README.md) · [Lessons](lessons.md) · [Exercises](exercises.md) · [Whole-project source map](../source-map.md)

Source inspected: `f45557509e0202d4c93411db3a1237f44d9082ae`. Snippets marked **actual source** are copied from this checkout. Other payloads/diagrams are labeled teaching examples, not extra production files.

## 1. Find the view before changing a label

| Actual source | Sections to open | E/F responsibility boundary |
|---|---|---|
| [public/index.php](../../public/index.php) | Page allowlist, native POST, admin page guard, layout include | Read with H; do not weaken route guards |
| [app/layout.php](../../app/layout.php) | Admin menu groups and shared shell | Coordinate E with A/B |
| [app/views.php](../../app/views.php) | `manage_*`, `approvals`, `reports`, public `results` | E primary operational markup; shared with student/layout owners |
| [app/portal-views.php](../../app/portal-views.php) | `admin`, `moderation`, `audit` | E primary admin sections; student/club sections shared |
| [app/models.php](../../app/models.php) | `entities`, `save_entity`, `review_request`; effects of `reserve` | F admin services; D/G/H review shared rules |
| [app/actions.php](../../app/actions.php) | `handle_action`, admin save/delete/review/attendance, early moderation | F with H; D shares personal-action sections |
| [app/community.php](../../app/community.php) | Admin access/deletion, `moderate_post`, admin API branches | F with D/H; E moderation UI |
| [app/api.php](../../app/api.php) | Admin route recognition, CRUD/review/report/attendance/audit | H adapter ownership; E/F verify domain contract |
| [database/schema.sql](../../database/schema.sql) | Relationships and cascade effects | G; all affected owners review changes |
| [Community migration](../../database/migrations/20261017_club_communities.sql) | Posts, comments, reactions and private reports | G; preserve applied migration history |

PHP files here contain compact lines with several statements or a whole template. A physical line count is not a useful measure of teaching or personal contribution. Follow the named branch/function.

## 2. Three authorization sites, one stored role

The page router requires admin for admin/approval/report/moderation/audit and `manage_*` pages. The API requires it for `admin/` routes. Core actions also enforce it after the personal actions.

**Actual source, `app/actions.php`:**

```php
 require_admin();
 if ($action==='save') { $entity=choice('entity',array_keys(entities())); $savedId=save_entity($entity,(int)($_POST['id']??0)); return action_result('manage_'.$entity,'Record saved.',['id'=>$savedId]); }
```

- `require_admin()` retrieves the signed-in account and checks its stored role.
- `choice()` accepts only keys returned by the fixed `entities()` map.
- `id=0` creates; a supplied nonzero ID selects update behavior.
- `save_entity()` owns field validation and persistence; the view does not implement SQL writes.
- `action_result()` returns structured navigation/message/data; it does not print the page itself.

Moderation dispatches earlier in `perform_action()`, so its service has its **own** `require_admin()`. Moving that guard below the update would introduce a serious defect. An admin-only menu is not a substitute.

## 3. One management branch serves four entity forms

**Actual source, `app/views.php`:**

```php
$entity=substr($page,7); $spec=entities()[$entity]; $edit=isset($_GET['edit'])?one("SELECT * FROM $entity WHERE id=?",[(int)$_GET['edit']]):null;
```

`substr` removes `manage_`; the page allowlist and descriptor map constrain the resulting identifier. `one()` executes a prepared-ID read. `null` means no edit row. The field loop chooses selects, textareas or inputs from `$spec['fields']`.

When validation returns `$error`, submitted values are preferred over stored values to help correction. Dates convert SQL values to `datetime-local`. Save carries action/entity/id; Delete carries the same routing fields plus a client confirmation attribute.

HTML current-record lists read all records in descending ID; they are not paginated or globally search-filtered. Do not promise those controls exist merely because API collections paginate.

## 4. Flow A · Native resource creation

**Goal:** create one expendable synthetic equipment resource, with JavaScript disabled so the native fallback is observable. Use your configured local origin and own disposable admin session.

Teaching example field set:

```text
page=manage_resources                 (URL query)
action=save, entity=resources, id=0    (hidden POST controls)
name=Study cones, kind=equipment
description=Isolated learning record
location=Practice cupboard
csrf=<the form's current hidden token; never record its value>
```

Enter values through the form; this display lists fields rather than a literal encoded request body.

1. GET `/?page=manage_resources` passes the page role guard and emits the generic form.
2. POST to that page enters `handle_action()`: hidden-field CSRF check, then `perform_action('save')`.
3. The shared action checks the user/admin role and fixes the allowlisted resource entity.
4. `save_entity()` validates name/kind/description/location, then begins its transaction.
5. It INSERTs a resources row, obtains its ID and inserts `save_resources` into audit history.
6. Commit publishes both writes; any exception before commit rolls the transaction back.
7. `handle_action()` stores the message as a flash notice and `go()` sends HTTP 303.
8. The browser GETs `/?page=manage_resources` and renders the actual saved record.

**Actual source, save completion in `app/models.php`:**

```php
  $savedId=$id ?: (int)db()->lastInsertId();
  audit('save_'.$entity,'Record '.$savedId);
  db()->commit();
  return $savedId;
```

`?:` uses the existing ID on update or the new database-generated ID on create. Audit follows the data write but precedes commit. Return happens only after successful commit.

**Observe:** 303 then a rendered GET, one synthetic resource card and a matching audit action. Do not claim that a success message alone proves persistence. Reread the record or inspect its safe synthetic ID/name.

The enhanced equivalent is POST `/api/v1/admin/resources` with JSON `name,kind,description,location`. It returns 201 and Location metadata; updating uses PUT with the complete field set. Both call the same model.

## 5. Flow B · REST membership approval

**Goal:** approve one synthetic pending request and show the student receives the correct club access. The request's membership ID is different from its club ID.

Teaching request contract, with no actual credentials:

```http
PATCH /api/v1/admin/memberships/{membership_id}
Content-Type: application/json
X-CSRF-Token: <current token from this admin session>
Cookie: <preserved privately by the browser/client>

{"status":"approved"}
```

Do not paste the placeholder header values into a real client or capture actual ones in evidence. The existing enhanced Approve button sends the request using its own session.

1. `app.js` maps native `review, entity=memberships, id=...` to the PATCH route.
2. It retains CSRF separately and removes routing-only native fields from the JSON body.
3. `public/api.php` loads shared services and invokes `api_dispatch()`.
4. The router checks method/admin, CSRF, supplied Origin, JSON and field allowlist.
5. Path-derived entity/ID become fixed action fields; the body cannot choose another owner.
6. `perform_action('review')` calls `review_request()` with allowlisted status.
7. The model requests READ COMMITTED and locks parent then request before final checks.
8. It checks current approved membership count, updates status and writes audit atomically.
9. The API returns 200 and redirect metadata for Approvals; that pending-only queue now omits the request.
10. A fresh student request sees approved status and can enter that exact club's private feed.

The review response is an action result/message, not a complete updated membership resource. Reread the relevant list/state to verify it. A full club or already reviewed request produces a domain validation failure rather than a second approval.

```mermaid
sequenceDiagram
    actor Admin
    participant UI as Browser
    participant API as REST transport
    participant Service as review_request
    participant DB as MySQL InnoDB
    Admin->>UI: Approve membership
    UI->>API: PATCH + JSON + cookie + CSRF
    API->>API: Verify admin and transport
    API->>Service: memberships, request ID, approved
    Service->>DB: READ COMMITTED; BEGIN
    Service->>DB: Read candidate; lock club; lock request
    Service->>DB: Reread pending; check capacity
    alt Eligible
        Service->>DB: UPDATE status; audit; COMMIT
        API-->>UI: 200 and redirect metadata
    else Full or stale
        Service->>DB: ROLLBACK
        API-->>UI: 422 validation response
    end
```

The native equivalent is POST `action=review, entity=memberships, id=<request>, status=approved` with hidden CSRF; it receives a flash/303 flow. Native bad CSRF is 419; REST bad CSRF is 403. The different transport does not grant different role permissions.

## 6. Read the critical review checks in source order

**Actual source, `app/models.php`:**

```php
  $parent=one("SELECT * FROM $parentTable WHERE id=? FOR UPDATE",[$candidate[$parentColumn]]);
  if (!$parent) throw new DomainException('The requested club or resource no longer exists.');
  $r=one("SELECT * FROM $entity WHERE id=? FOR UPDATE",[$id]);
  if (!$r || $r['status']!=='pending') throw new DomainException('Request is no longer pending.');
  if ((int)$r[$parentColumn] !== (int)$candidate[$parentColumn]) throw new RequestError(409,'The request changed. Reload it before reviewing.');
```

The candidate read supplies parent context. After any wait, the request is reread under lock: missing/no-longer-pending rejects, and changed parent identity rejects 409. Only after that does the model evaluate capacity/overlap and write status. The controlled parent/table names come from the entity choice, not arbitrary submitted SQL.

`tests/concurrency.php` holds a parent through one connection, proves a second worker waits at its lock, commits a competing approval, then checks the worker rejects. Read it as a focused regression, not a claim every possible race is solved. It resets only its guarded disposable schema when explicitly run.

## 7. Record scores through event Save

**Actual source, `app/models.php`:**

```php
  if (($data['home_score']===null)!==($data['away_score']===null)) throw new DomainException('Enter both scores.');
  if ($data['home_score']!==null && ($data['kind']!=='match' || $data['opponent']==='')) throw new DomainException('Scores require a match and an opponent.');
```

The first comparison computes two booleans: “home missing?” and “away missing?”. If they differ, exactly one score is missing. Both zero is not missing. The second line requires match+opponent whenever scores exist.

There is no separate `record_score` action. E improves the existing event form/display; F preserves `save_entity()` checks; H checks the full REST replacement contract. `results` and `standings` derive data on read.

## 8. Attendance and report reads

**Actual source, `app/actions.php`:**

```php
  $record=one('SELECT a.id,e.starts_at FROM attendance a JOIN events e ON e.id=a.event_id WHERE a.id=?',[positive('id')]);
  if (!$record || strtotime($record['starts_at'])>time()) throw new DomainException('Attendance can only be recorded after a session starts.');
```

A positive attendance ID selects its event; `strtotime()` compares the stored Sri Lanka datetime to current time. Future starts reject. The following UPDATE sets attended and writes audit; this action lacks the encompassing transaction used by CRUD save/delete, so do not claim universal audit atomicity.

The Reports page then counts attendance rows and attended subsets. Its UI row condition checks `registered`, which explains the proposed future-control feedback improvement. A direct request still needs this guard after any UI change.

Report printing uses `#print-report` → `window.print()`. Audit uses a LEFT JOIN so a removed user can leave a retained log row with a null actor. Log details are bounded operational metadata, not full request payloads.

## 9. Moderate without rewriting an author's words

**Actual source, `app/community.php`:**

```php
  $post=one('SELECT * FROM club_posts WHERE id=? AND status<>"deleted" FOR UPDATE',[$id]);
  if (!$post) throw new RequestError(404,'Post not found.');
  query('UPDATE club_posts SET status=?,moderation_reason=? WHERE id=?',[$status,$reason,$id]);
  query('UPDATE club_reports SET status="resolved" WHERE post_id=?',[$id]);
  audit('community_moderation','Post '.$id.': '.$status); db()->commit();
```

A nondeleted post is locked. Decision/reason, report resolution and audit commit together. Resolved reports remain records; hiding is not deletion. A tombstone cannot be selected here for restoration.

HTML moderation `status=all` means all nondeleted posts. The API uses an omitted/empty filter for that meaning; literal `all` is invalid there. Member-safe feed fields omit moderation reasons. Keep those distinctions when sharing UI/API work.

## 10. Diagnose before editing

| Symptom | Evidence to inspect | Safe interpretation |
|---|---|---|
| Student request 403 | Stored role and named guard | Expected access refusal, not missing UI permission |
| Native 419 / REST 403 | Fresh session/form; exact origin | Reload/refetch token; preserve CSRF |
| Validation 422 | Field/type/date/state message | Correct input or stale state; do not weaken the rule |
| Conflict 409 | Current request parent/state | Reread before retrying |
| Missing record 404 | Correct record ID and deletion state | Do not substitute club ID for membership ID |
| Generic 500/503 | Protected logs, doctor, request ID | Diagnose configuration/SQL privately |

At a breakpoint inspect action name, synthetic target ID and branch outcome. Never dump `$_POST`, session/cookie/token values, passwords, authenticator material, private report text or full user records. [PHP transactions](https://www.php.net/manual/en/pdo.transactions.php) and [OWASP CSRF guidance](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) help explain the boundary rather than bypass it.
