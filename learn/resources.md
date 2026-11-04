# A focused resource shelf

[Learning home](README.md) · [Start here](00-start-here.md) · [Glossary](glossary.md)

Use a resource to answer the next question raised by the code. You do not have to finish every linked course. The **current project source** defines implemented behavior; official documentation explains the underlying technology. A tutorial for another stack is not a reason to introduce its frameworks here.

External sites can change, redirect, require access or update examples. A reachable URL does not establish technical compatibility. PHP examples should fit 8.2+; database behavior should be checked against actual MySQL 8+; the MySQL 8.4 manual is the reference version used here. Browser tools and Docker are optional development aids, not extra application requirements.

## First five references

| Resource | Read for this immediate purpose | Apply it here |
|---|---|---|
| [MDN: how the web works](https://developer.mozilla.org/en-US/docs/Learn_web_development/Getting_started/Web_standards/How_the_web_works) | Browser, server, request and returned files | Draw a page request using `public/index.php` |
| [PHP language reference](https://www.php.net/manual/en/langref.php) | Values, arrays, conditions, functions and includes | Explain one PHP view branch before editing it |
| [PDO prepared statements](https://www.php.net/manual/en/pdo.prepared-statements.php) | Separate SQL structure from bound values | Trace `query()` in `app/bootstrap.php` |
| [MySQL tutorial](https://dev.mysql.com/doc/refman/8.4/en/tutorial.html) | Read relational data and understand table definitions | Follow user → membership → club |
| [Pro Git](https://git-scm.com/book/en/v2) | Record actual scoped work and collaborate | Inspect `git status` and `git diff` before a commit |

## Stage 01 — HTML, CSS, JavaScript and usable interfaces

| Resource | Focus | Small project exercise |
|---|---|---|
| [MDN HTML learning](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content) | Semantic structure and documents | Identify header/nav/main/headings in a rendered page |
| [MDN forms](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms) | Labels, controls, validation and submission | Explain why `name` sends data but `id` alone does not |
| [WAI forms tutorial](https://www.w3.org/WAI/tutorials/forms/) | Accessible labels, instructions and errors | Inspect login and booking labels/focus |
| [MDN CSS basics](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics) | Cascade, inheritance, box model and selectors | Find which rule wins for a button color |
| [MDN layout](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/CSS_layout) | Flexbox, Grid and responsive layout | Explain a card grid at desktop and phone widths |
| [MDN JavaScript guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide) | Values, functions, control flow and collections | Explain a handler in `app.js` without rewriting it |
| [MDN DOM introduction](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction) | How JS finds and changes elements | Trace a theme-toggle click |
| [MDN Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) | Methods, headers, credentials, response handling | Relate a native form to its JSON enhancement |
| [MDN constraint validation](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Constraint_validation) | Browser validation and its limits | Compare browser refusal with server validation |
| [Chrome DevTools Network](https://developer.chrome.com/docs/devtools/network/) | Observe the actual request/response | Distinguish HTML navigation, asset and fetch requests |
| [WAI modal dialog pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/) | Focus, Escape and keyboard principles | Compare applicable focus ideas with this app's drawer; the pattern is not proof the drawer is a modal dialog |

Use the browser available to you; equivalent Network/Elements tools exist in Firefox and Safari. Automation does not replace keyboard and assistive-technology checks.

## Stages 02 and 03 — forms, workflows and application rules

| Resource | Focus | Apply it here |
|---|---|---|
| [MDN HTTP overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview) | Request/response messages and methods | Trace GET page versus POST action |
| [MDN HTTP status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status) | Distinguish success, redirect and failure | Explain this API's 401, 403, 409 and 422 |
| [PHP forms tutorial](https://www.php.net/manual/en/tutorial.forms.php) | Server receives submitted fields | Follow `$_POST` from a native form |
| [PHP HTML escaping](https://www.php.net/manual/en/function.htmlspecialchars.php) | Encode text for HTML output | Explain `e()` and why validation alone is insufficient |
| [MySQL foreign keys](https://dev.mysql.com/doc/refman/8.4/en/create-table-foreign-keys.html) | Parent/child relationships and delete actions | Predict what deleting a club can remove |
| [MySQL aggregate functions](https://dev.mysql.com/doc/refman/8.4/en/aggregate-functions.html) | Counts, sums and grouped reports | Explain what a participation count actually counts |
| [OWASP authorization](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) | Enforce permission on each request | Test exact-club membership and own cancellation |
| [OWASP input validation](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html) | Shape, syntax and business semantics | Distinguish a valid date string from a forbidden past booking |

Read the actual schema and actions before copying an online CRUD example. Capacity, audit atomicity, ownership and cancellation state are project rules beyond a generic INSERT/UPDATE form.

## Stage 04 — PHP, MySQL, API and security

| Resource | Focus | Apply it here |
|---|---|---|
| [PHP associative arrays](https://www.php.net/manual/en/language.types.array.php) | Keys, row values and iteration | Trace a fetched row into a template |
| [PHP Exception class](https://www.php.net/manual/en/class.exception.php) | Exceptions, error boundaries and cleanup | Follow a domain failure into native or JSON output |
| [PDO documentation](https://www.php.net/manual/en/book.pdo.php) | Connections, queries and transactions | Explain driver, exception mode and native prepares |
| [PHP session security](https://www.php.net/manual/en/features.session.security.management.php) | Session ID lifecycle, cookies and expiry | Compare with bootstrap and `finish_login()` |
| [PHP password hashing](https://www.php.net/manual/en/function.password-hash.php) | One-way password storage and Argon2id | Explain why password hashes cannot be decrypted |
| [PHP sodium extension](https://www.php.net/manual/en/book.sodium.php) | Authenticated encryption primitives | Read secretbox use; avoid inventing crypto replacements |
| [MySQL InnoDB locking](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking.html) | Locking reads and serialization | Explain why two approvals lock the same parent |
| [MySQL isolation levels](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html) | Snapshot/read visibility | Explain the READ COMMITTED regression fix |
| [MySQL implicit commits](https://dev.mysql.com/doc/refman/8.4/en/implicit-commit.html) | Why DDL is not ordinary transactional DML | Explain retry-safe additive migration statements |
| [OWASP SQL injection prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html) | Parameter values and allowlisted identifiers | Review `query()` and entity allowlists |
| [OWASP XSS prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) | Output-context handling | Review plain-text community rendering |
| [OWASP CSRF prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) | Session-bound mutation protection | Compare hidden native token and JSON header |
| [OWASP session management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) | Cookie flags, rotation and timeout | Explain why port alone is not cookie isolation |
| [OWASP authentication](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) | Credential errors, throttling and lifecycle | Trace login and missing recovery/reset capabilities |
| [OWASP multifactor authentication](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html) | Enrollment, recovery and operational risks | Explain the password-then-pending-factor state |
| [RFC 6238: TOTP](https://www.rfc-editor.org/rfc/rfc6238.html) | Time steps and standard test vectors | Relate vectors to `tests/security.php` |
| [RFC 9110: HTTP semantics](https://www.rfc-editor.org/rfc/rfc9110.html) | Precise method/status meaning | Understand PUT/PATCH and repeated mutations |
| [OpenAPI 3.1 specification](https://spec.openapis.org/oas/v3.1.1.html) | Documenting an HTTP contract | Compare `openapi.json` with implemented routes |
| [OWASP REST security](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html) | API input/access/error controls | Review actual route dispatch without disabling CSRF |

OWASP is guidance, not a certificate that this project is secure. Some recommended systems, such as email verification/reset, mandatory-admin 2FA and shared multi-node sessions, are not implemented here. Document gaps rather than claiming every linked recommendation is satisfied.

## Stage 05 — Git, local tooling, hosting and recovery

| Resource | Focus | Apply it here |
|---|---|---|
| [Git branch basics](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging) | Branches, merges and conflicts | Make a bounded task branch and review the actual diff |
| [Git diff reference](https://git-scm.com/docs/git-diff) | Inspect what will change | Check unintended changes and whitespace |
| [GitHub pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests) | Review before integration | Record real authors/reviewers and actual validation |
| [PHP development server](https://www.php.net/manual/en/features.commandline.webserver.php) | Local routing and development-only limits | Understand `-t public public/router.php` |
| [Docker Desktop installation](https://docs.docker.com/desktop/) | Supported local installation for your OS | Verify engine is running before Docker commands |
| [Docker Compose CLI](https://docs.docker.com/reference/cli/docker/compose/) | Select the intended project/env/file | Use the full isolated README prefix |
| [Docker volumes](https://docs.docker.com/engine/storage/volumes/) | Persistence outside containers | Explain why deleting a volume can erase the database |
| [Apache document roots](https://httpd.apache.org/docs/2.4/urlmapping.html) | Map URLs to public files | Verify only this package's `public/` is exposed |
| [Apache rewrite](https://httpd.apache.org/docs/2.4/rewrite/) | Route API URLs to PHP | Compare real host behavior with `.htaccess` |
| [MySQL backup and recovery](https://dev.mysql.com/doc/refman/8.4/en/backup-and-recovery.html) | Consistent backup strategies and restoration | Rehearse a restore into a separate scratch database |
| [mysqldump reference](https://dev.mysql.com/doc/refman/8.4/en/mysqldump.html) | Logical dumps and platform details | Use prompted passwords/private result files, not public exports |
| [Let's Encrypt documentation](https://letsencrypt.org/docs/) | Certificates and HTTPS operation | Use the hosting provider's supported TLS procedure |

Host product plans, free-tier limits, prices and lecturer access are not established by these resources. Verify the exact chosen plan before paying or publishing. GitHub Pages serves static sites and does not run this PHP/MySQL backend.

## How to use a reference without becoming overwhelmed

1. Write a specific question: “Why does this prepared query use `?`?”
2. Read the relevant section, not every page in the manual.
3. Locate the matching project fragment and explain its inputs/output.
4. Make one prediction, observe a synthetic example, record what happened.
5. Return to the stage exercise. Bookmark deeper topics for later.

If an example recommends a different stack, extract the concept and adapt it to the existing architecture. Do not add Laravel, React, Node, Django or a different database simply because a tutorial uses them.
