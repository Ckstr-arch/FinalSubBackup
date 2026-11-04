# Source walkthrough · from PHP to the visible page

[Track home](README.md) · [Lessons](lessons.md) · [Exercises](exercises.md) · [Complete source map](../source-map.md)

This is a guided trace through the current files, inspected at `f455575`.
The code fences labelled **source excerpt** are selected real fragments, sometimes taken from a longer template line.
Illustrative emitted HTML is labelled separately; it is not another production source file.
No runtime test or hosting result is implied by reading these excerpts.

## 1. Locate the controller before the template

Open [public/index.php](../../public/index.php).
It requires bootstrap and actions, selects the allowlisted `page`, handles POST through `handle_action()`, resolves the current user and gates protected routes.
Then it includes the fixed layout path.
The layout includes views, which delegates dashboard, admin, my_clubs, club, moderation and audit to portal views.
A page name does not become an arbitrary include filename.

```mermaid
sequenceDiagram
    participant Browser
    participant Index as public/index.php
    participant Boot as app/bootstrap.php
    participant View as layout.php + views.php
    participant DB as MySQL
    Browser->>Index: GET /?page=home
    Index->>Boot: load helpers and session; resolve user
    Index->>View: render fixed template includes
    View->>Boot: one()/rows() for counts and clubs
    Boot->>DB: prepared queries
    DB-->>View: result rows through PDO helpers
    View-->>Browser: HTML with escaped values
    Note over Browser: Parse DOM; apply CSS; execute deferred app.js
```

Input to the layout includes `$page`, `$current`, `$notice`, `$error` and `$clubView`.
Do not invent these variables independently in a copied header to bypass the controller.
If protected content is absent, first check the selected page and authenticated role; CSS cannot fix a server access refusal.

## 2. Read the document shell as HTML and PHP

Open [app/layout.php](../../app/layout.php).
The file derives `$isAdmin` from the current server user and chooses a grouped admin sidebar or student workspace links.
Public visitors receive the public navigation branch instead of an authenticated sidebar.
`aria-current="page"` describes the active link; it does not grant permission to open its destination.

**Source excerpt:**

```html
<a class="skip-link" href="#main">Skip to content</a>
```

The fragment targets the unique `id="main"` later in the document.
The main has `tabindex="-1"`, so it can receive focus without adding another normal Tab stop.
CSS places the skip link above the screen until focus; keyboard users can then bypass repeated navigation.

The head loads assets in this order:

1. `theme-init.js` synchronously, before CSS.
2. `app.css` as the stylesheet.
3. `app.js` with `defer`, so it executes after HTML parsing.

This is deliberate: the early script chooses the saved palette before paint, while the main script needs the parsed controls/forms.
The theme toggle precedes the profile menu in the emitted header.
Profile `details`/`summary` gives a native disclosure widget, enhanced by outside-click/Escape behavior.
Sign out is a CSRF-protected POST form rather than a GET navigation link.

## 3. Identify the actual homepage DOM

Open the `home` branch of [app/views.php](../../app/views.php).
It selects database counts, selects the next future match and emits hero/card/banner markup.
The counts are not random frontend statistics; the visible plus sign next to each count is literal punctuation.

```text
main#main
├── section.hero
│   ├── div.hero-copy → eyebrow, h1, p, hero-actions, hero-note
│   └── div.hero-photo → img, photo-tag, photo-caption
├── div.stats-strip → four count cells
├── section.content
│   ├── div.section-heading
│   └── div.grid.three → article.card.club-card repeated
└── section.feature-banner → next match or empty copy
```

| Actual node | Selector to find in CSS | Visible responsibility |
|---|---|---|
| Hero wrapper | `.hero` | Two-column desktop composition, stacked at mobile width |
| Headline | `.hero h1` | Responsive size and tracking |
| Colored headline span | `.hero h1 span` | Theme link/accent color |
| CTA link row | `.hero-actions` | Wrapping Flex layout |
| Image wrapper | `.hero-photo` | Sized relative container and clipping |
| Track image | `.hero-photo img` | Absolute cover crop and brightness filter |
| Statistics | `.stats-strip` | Four tracks, then two at narrow width |
| Club wrapper | `.grid.three` | Three/two/one responsive card tracks |
| Club article | `.card` | Surface, border, padding, shadow and minimum width |
| Next-fixture section | `.feature-banner` | Text/action row that stacks below 800px |

The homepage card's link goes to club browsing; the Clubs page supplies a specific club overview link with `club_url()`.
Do not assume every similar-looking card has the same destination or JavaScript listener.

## 4. Trace one database value into HTML

**Source excerpt from the homepage loop:**

```php
<?php foreach (rows('SELECT * FROM clubs ORDER BY id LIMIT 3') as $c):?>
```

`rows()` returns an array of records through the PDO helper.
`foreach` assigns each record to `$c` and repeats the following markup.
The alternate `:` template syntax is paired with `endforeach` later in the template.
The query is fixed and has no user-supplied SQL expression.

**Source excerpt from inside the article:**

```php
<h3><?=e($c['name'])?></h3><p><?=e($c['description'])?></p>
```

`$c['name']` indexes one record's name; `<?=...?>` echoes its result.
`e()` calls `htmlspecialchars` with quote handling and UTF-8 substitution in [bootstrap.php](../../app/bootstrap.php).
It converts characters that would otherwise affect HTML syntax into safe text representations.

**Illustrative emitted HTML with synthetic data:**

```html
<h3>Track &amp; Field</h3><p>Meet &lt;practice&gt; partners.</p>
```

The browser displays `Track & Field` and literal angle brackets around `practice`.
It does not create a `practice` element from that escaped text.
Prepared SQL protects the query boundary; escaping protects the output boundary.
Deleting `e()` because a query is prepared would confuse two different protections.
The empty-state branch explains what appears if no clubs exist; inspect it instead of clearing a shared database.

## 5. Match CSS to the component

Open [app.css](../../public/assets/app.css), first at `:root`, then `.grid` and `.card`.

**Source excerpt:**

```css
.grid.three {
  grid-template-columns: repeat(3, minmax(0, 1fr));
}
```

The element also has `.grid`, which supplies `display:grid` and the gap.
A `grid-template-columns` declaration on a non-grid element would not create the intended layout.
Three tracks share remaining width; `minmax(0,1fr)` allows them to shrink rather than forcing a long content minimum.
The `.card` class adds `min-width:0` and uses theme variables for surface/border.

**Source excerpt:**

```css
:root[data-theme="light"] {
  color-scheme: light;
  --bg: #f1f5f9;
```

This is a fragment of a larger block, not a complete stylesheet.
The root attribute selects the light token set; `color-scheme` also informs browser-native controls.
A variable is only a stored value until another declaration consumes it, for example `background:var(--bg)` on the body.
Dark and light do not need duplicate copies of every card rule.

At 1050px the three-card grid becomes two; at 500px it becomes one.
At 800px the authenticated sidebar is translated offscreen and hidden until `.is-open` applies.
At the end of the file, late `.metric small` and `.session-item p` declarations win against earlier equal-specificity values.
Inspect Computed rather than guessing from the first matching rule.

## 6. Read UI helpers by what they return or print

Open [app/ui.php](../../app/ui.php).
These are rendering helpers, not independent authorization or persistence services.

| Helper | Input → output | Important contract |
|---|---|---|
| `icon()` | Known icon name → static SVG string | Unknown names fall back to home; SVG is decorative with `aria-hidden` |
| `initials()` | Name → first letters of up to two words | Uses mbstring; callers escape display text |
| `club_url()` | Integer ID and tab → internal URL | ID is typed and tab is URL-encoded; current templates generally echo this constrained internal URL directly |
| `community_form()` | Action and IDs → opening POST form plus hidden fields | Emits the opening only; caller supplies controls and closes the form |
| `metric()` | Label, integer value, icon and URL → linked metric card | Caller supplies real counts; helper escapes its URL with `e()` but does not calculate counts |

`rawurlencode()` and `e()` are not interchangeable: one protects a URL value's syntax, the other protects the HTML context where the URL is emitted.
The SVG fragments are controlled static markup; arbitrary names/descriptions are text and must remain escaped.
The brand image has empty alt because adjacent brand text already names the same link.
The homepage photo has descriptive alt; CSS background images are decorative and need meaningful surrounding content where appropriate.
A supplied photo does not prove its location or publication rights; retain the existing asset provenance checks before public hosting.

## 7. Follow the menu and theme listeners

Open the first half of [app.js](../../public/assets/app.js), then [theme-init.js](../../public/assets/theme-init.js).

**Source excerpt:**

```javascript
const menu = document.querySelector(".menu-toggle");
const navigation = document.getElementById("navigation");
const scrim = document.querySelector(".sidebar-scrim");
```

These selectors match nodes supplied by the shell.
Renaming the ID or class in PHP requires coordinated JS/CSS updates.
The menu click updates `aria-expanded`, open classes and, for a sidebar, dialog/modal state, body scroll and scrim visibility.
Two animation-frame callbacks defer initial focus until the drawer is visibly presented; a guard avoids taking focus after a quick close.
`closeNavigation()` reverses those states and restores focus only if the menu was open.
The media-query listener closes mobile state above 800px.

The early theme script accepts only the stored string `light`; absent, invalid or unavailable storage falls back to dark.
The main script changes `document.documentElement.dataset.theme`, persists `sportshub-backup-theme` when possible and updates the button's next-action label/pressed state.
Another tab's change arrives through the `storage` event.
A storage error does not prevent the current page from changing its palette.
Do not use this storage channel for passwords, session tokens, authenticator keys or recovery codes.

## 8. Follow a form through both paths

Start at the Clubs branch of [views.php](../../app/views.php): `action_button('join', ...)` emits a native POST form with `action`, `id` and `csrf`.
Without JavaScript, [index.php](../../public/index.php) calls `handle_action()` in [actions.php](../../app/actions.php).
It checks CSRF, calls `perform_action()`, then returns flash/303 navigation through `go()`.
`perform_action('join')` requires a user and calls `reserve()` in [models.php](../../app/models.php), where the database rules apply.

**Source excerpt from the enhancement:**

```javascript
    const fields = Object.fromEntries(new FormData(form));
    const route = endpoint(fields.action, fields);
    if (!route) return; // Existing native form behavior remains for any unmapped extension.
    event.preventDefault();
```

The route is identified before native submit is suppressed.
For join it becomes `POST /api/v1/clubs/{id}/memberships`.
Routing-only fields are removed from the JSON object; CSRF moves into a header, while the browser sends the session cookie using same-origin credentials.
[public/api.php](../../public/api.php) and [app/api.php](../../app/api.php) validate the REST request and call the shared action/service layer.

A successful response may refresh CSRF fields and supply an internal `/?page=` redirect.
A failed HTTP response becomes a safe text alert, with a sign-in link for 401.
A parse/network failure produces a different warning: inspect current activity before retrying because a write may already have committed.
Buttons are restored in `finally`.
For recovery-code responses, the script shows an ephemeral modal and awaits acknowledgment before navigating; do not capture or log those values.

## 9. Know the scope of this walkthrough

You have traced the shared shell, homepage card/data loop, tokens/grid, helpers, menu/theme and one native/REST form.
Student-specific eligibility, admin CRUD and security internals need their own source walkthroughs; this is not a claim that every domain rule lives in the frontend.
Use the [source map](../source-map.md) and [exercises](exercises.md) to continue with one testable question.
