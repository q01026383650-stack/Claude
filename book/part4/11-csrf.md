# Chapter 11 — CSRF and the Role of HTML Forms

> Cross-Site Request Forgery turns the browser's own helpfulness against the user: because the
> browser automatically attaches cookies to requests, an attacker's page can make the victim's
> browser send an authenticated, state-changing request to a target site. HTML forms (and a few
> other elements) are the engine that fires those requests — which is why CSRF belongs in a book
> about HTML.

---

## 11.1 The core idea

Recall from [Chapter 4](../part1/04-the-dom.md) that the Same-Origin Policy stops a cross-origin
page from *reading* your responses, but it does **not** stop a cross-origin page from *sending*
requests — and the browser will attach your cookies to those requests anyway. CSRF exploits that
gap:

```
Victim is logged in to bank.example (has a session cookie).
Victim visits attacker.example (or any page the attacker controls content on).
That page silently submits a form to bank.example/transfer.
The browser includes the victim's bank.example cookie automatically.
bank.example sees a valid, authenticated request and performs the action.
```

The attacker never sees the response (SOP blocks that) — but they didn't need to. The point of
CSRF is to cause a **state change** (transfer money, change email/password, add an admin, delete
data), not to read data.

## 11.2 Conditions required for CSRF

A request is CSRF-able when all of these hold:

1. **It performs a meaningful state change** (otherwise there's nothing to forge).
2. **It relies on cookies (or other automatically-sent credentials)** for authentication, with
   no unpredictable per-request secret.
3. **The request parameters are predictable** — the attacker can construct the full request
   without anything they can't know.

If any condition fails — the action is read-only, or it requires a secret token the attacker
can't guess, or it isn't cookie-authenticated — classic CSRF doesn't work.

## 11.3 HTML elements that send cross-site requests

This is the HTML heart of the chapter. Several elements cause the browser to issue requests, and
many do so cross-origin with cookies attached:

**`<form>` — the primary weapon.** A form can auto-submit to any URL:

```html
<!-- Conceptual PoC: auto-submitting cross-site POST. Use only in authorized testing,
     targeting an in-scope, low-impact action you control. -->
<form action="https://target.example/account/email" method="POST" id="f">
  <input type="hidden" name="email" value="attacker@evil.example">
</form>
<script>document.getElementById('f').submit();</script>
```

Key HTML facts that make forms powerful for CSRF:

- Forms submit **`GET` or `POST`** natively (no JS needed for the request itself, though JS is
  used to auto-submit).
- With **`enctype="text/plain"`**, the body is sent almost verbatim, which lets an attacker
  approximate a JSON body (e.g., crafting `{"email":"x@evil.example","ignore":"=...` style
  payloads) to hit some JSON endpoints that don't strictly check `Content-Type`.
- Forms **cannot set custom headers** and can't send `application/json` with arbitrary
  structure — a limitation that underpins several defenses (§11.6).

**Other request-issuing elements** (mostly `GET`, useful when the target action is a GET):
- `<img src="https://target.example/action?x=1">` — fires a GET on load.
- `<iframe src=...>`, `<script src=...>`, `<link href=...>`, `<video>/<audio>/<source>`,
  `<object>/<embed>` — all fetch URLs.
- `<a>` with auto-click, `<meta http-equiv="refresh">`, CSS `url()` in injected styles.

**The practical rule:** if a state-changing action can be triggered by a simple GET, an `<img>`
tag is enough. If it requires POST, an auto-submitting `<form>` is the tool. If it requires
JSON + custom headers + a token, it's likely not CSRF-able.

## 11.4 GET-based CSRF is the easy mode

Any state-changing action exposed over **GET** is a gift to an attacker: it can be triggered by
an `<img>`, a link, or a redirect — no form, no JavaScript, embeddable anywhere user content
appears.

```html
<img src="https://target.example/account/delete?confirm=true" width="1" height="1">
```

This is why **state changes should never be exposed over GET** — a principle that's both a
finding when violated and a remediation (§11.7). During mapping (Chapter 6), flag every GET that
mutates state.

## 11.5 Building and testing a CSRF proof of concept

A responsible CSRF test (in scope, least-impact):

1. **Capture the legitimate request** in your proxy: method, URL, body, content type, and which
   tokens/headers are present.
2. **Identify the auth mechanism**: is it a cookie? Is the cookie `SameSite=Lax/Strict/None`?
3. **Strip non-essential parameters** and any apparent anti-CSRF token, then **replay** the
   request to see whether it still succeeds without the token. (If it succeeds without the
   token, the token isn't actually validated — a finding.)
4. **Construct a self-contained HTML PoC** (a form or `<img>`) that issues the request.
5. **Test as the victim**: in a separate logged-in session (an account you control), load the
   PoC and confirm the action occurs.
6. **Choose a low-impact target action** to demonstrate (toggle a benign preference) where
   possible, per [Chapter 1](../01-legal-and-ethics.md). Avoid destructive actions and never
   target real users.

Many proxies (Chapter 16) can auto-generate a CSRF PoC from a captured request, which is handy
but always verify it reflects the real request.

## 11.6 Why modern defaults reduced (but didn't eliminate) CSRF

Two big shifts changed the CSRF landscape:

**`SameSite` cookies.** Browsers now default cookies to `SameSite=Lax` when the site doesn't
specify otherwise. `Lax` means the cookie is **not** sent on most cross-site *subrequests*
(like the `<img>`/`<form POST>` an attacker uses), though it *is* sent on top-level GET
navigations the user initiates. This blunts a lot of classic CSRF automatically. But it's not a
complete fix:

- **`SameSite=None`** (often set for cookies that must work in third-party contexts) re-opens
  the door.
- **Lax allows top-level GET navigation**, so GET-based state changes triggered by a full-page
  navigation can still ride the cookie.
- **`SameSite` is per-cookie and per-browser**; older clients and misconfigurations exist.
- **Method/timing edge cases** and the ~2-minute "Lax+POST" intervals some browsers historically
  applied have created windows.

So a tester must still check `SameSite` *per cookie* and not assume the app is safe just because
modern browsers default to `Lax`.

**Token and header-based defenses.** Apps add unpredictable anti-CSRF tokens (synchronizer
tokens) or rely on the fact that forms can't set custom headers or send arbitrary JSON. If an
API requires `Content-Type: application/json` and a custom header (e.g., `X-Requested-With` or a
CSRF token header), a plain HTML form can't forge it — because forms can't set those headers and
can't produce arbitrary JSON bodies (§11.3).

## 11.7 Common CSRF defense flaws to test for

Defenses are often present but broken. Probe each:

- **Token not validated.** Remove it; does the request still work? (Astonishingly common.)
- **Token validated only when present.** Send an empty token or omit the parameter entirely.
- **Token tied to the wrong thing.** Is the token bound to the user's session? Try using your
  own valid token in a request "as" another flow, or reuse a token across users.
- **Token in a place the attacker can influence.** A token read from a cookie and compared to a
  body field (the "double-submit" pattern) can be defeated if the attacker can set the cookie
  (e.g., via a subdomain or a cookie-injection bug).
- **Token leaks** in URLs, `Referer`, or to third parties.
- **Method override.** The app checks the token on POST but accepts the same action via GET or a
  `_method` override field.
- **Content-Type not enforced.** A JSON endpoint that also accepts form-encoded or `text/plain`
  bodies can be hit by a form (§11.3). Test sending the action as `text/plain`/form-encoded.
- **`Referer`/`Origin` checks that fail open.** Some apps validate `Referer`/`Origin` but allow
  the request when the header is *absent*; an attacker can sometimes suppress it (e.g., via
  `rel="noreferrer"`, `meta` referrer policy, or `data:`/HTTPS→HTTP transitions).
- **`SameSite=None` or absent** on the session cookie.
- **CORS misconfig used as CSRF.** A reflective `Access-Control-Allow-Origin` with
  `Allow-Credentials: true` can turn into cross-origin *reading* — adjacent to CSRF and worth
  noting.

## 11.8 Login CSRF and logout CSRF

CSRF isn't only about authenticated victims acting on their own account:

- **Login CSRF**: the attacker forges a *login* request that logs the victim into the
  *attacker's* account. The victim then unknowingly operates in the attacker's account — saving
  payment details, search history, or files that the attacker later retrieves. Defenses (CSRF
  tokens) must cover the login form too, which developers often forget.
- **Logout CSRF**: forcibly logging the victim out (low impact alone, but useful in chains).

## 11.9 CSRF as a building block in chains

CSRF rarely stands alone in high-impact reports. It chains:

- **CSRF → self-stored-XSS escalation.** If an app has a "self-XSS" that only affects your own
  account, CSRF can place that payload into a *victim's* account, converting self-XSS into a
  real attack.
- **CSRF → account takeover.** Forge an email-change or password-reset-initiation request, then
  take over via the attacker-controlled email.
- **CSRF + clickjacking.** When a token blocks pure CSRF, clickjacking (Chapter 12) can trick the
  user into performing the action through the real UI, which carries the token.

## 11.10 Detecting CSRF (procedure)

1. From the Chapter 6 map, list **state-changing** actions.
2. For each: identify auth mechanism, presence/validation of anti-CSRF token, cookie `SameSite`,
   accepted methods, and accepted content types.
3. Replay the request **without** the token / with it emptied; observe.
4. Try **method/content-type downgrades** (POST→GET, JSON→form/`text/plain`).
5. Build a minimal HTML PoC for any action that still succeeds; verify as a controlled victim.
6. Record severity by **impact of the action** (email change, money, admin) and **ease of
   delivery**.

## 11.11 Fixing CSRF

Layered defenses (details in [Chapter 18](../part6/18-defense-remediation.md)):

- **Anti-CSRF tokens**: per-session (or per-request) unpredictable tokens, bound to the user's
  session, validated server-side on every state-changing request, and **rejected when missing**.
- **`SameSite` cookies**: set session cookies to `SameSite=Lax` (or `Strict` where UX allows);
  avoid `None` unless genuinely needed, and pair `None` with tokens.
- **Never expose state changes over GET**; require POST/PUT/DELETE with proper protections.
- **Validate `Origin`/`Referer`** as a secondary check, failing **closed** when the header
  indicates a cross-site origin.
- **Require non-forgeable request shapes** for sensitive APIs (custom header + JSON content type,
  enforced server-side) so HTML forms can't reproduce them.
- **Re-authenticate** for the most sensitive actions (password/email change, large transfers).

The deepest point: CSRF is the price of *ambient authority* — credentials the browser sends
automatically. Defenses work by adding something the attacker's HTML *cannot* supply (an
unpredictable token, a custom header, a `SameSite` restriction).

---

## Key takeaways

- CSRF abuses the browser auto-attaching cookies: a cross-site page makes the victim send an
  authenticated **state-changing** request. SOP blocks reading the response, but the action still
  happens.
- **HTML forms** (GET/POST, `enctype=text/plain`) and request-issuing elements (`<img>`,
  `<iframe>`, `<script>`...) are the engine. **State changes over GET are trivially forgeable.**
- `SameSite=Lax` defaults reduced classic CSRF but didn't kill it — check `SameSite` **per
  cookie**, test `None`, top-level GET navigations, and content-type/method downgrades.
- Test for **broken** defenses: unvalidated/empty/reusable tokens, cookie-settable double-submit,
  fail-open `Origin` checks, and JSON endpoints that accept form/`text/plain`.
- Fix with session-bound anti-CSRF tokens (rejected when missing), `SameSite` cookies, no
  state-change-over-GET, `Origin`/`Referer` validation, and re-auth for sensitive actions.

---

[← Previous: Chapter 10 — DOM-Based XSS](../part3/10-xss-dom.md) | [Next: Chapter 12 — Clickjacking and UI Redressing →](12-clickjacking.md)
