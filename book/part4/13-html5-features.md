# Chapter 13 — HTML5 Features and Their Security Impact

> "HTML5" loosely names the modern web platform: new elements, attributes, and a wave of
> JavaScript APIs. Each convenience added new attack surface. This chapter surveys the features
> a tester must understand — not as a list of trivia, but organized by *how each one expands what
> an attacker can do*.

---

## 13.1 New elements and attributes that carry event handlers

HTML5 added media and interactive elements, and crucially gave many of them **event handler
attributes** that fire without `<script>` — invaluable when `<script>` is filtered (Chapter 8):

- `<video>`, `<audio>` with `onerror`, `onloadstart`, etc.
- `<source>` inside media elements.
- `<details>`/`<summary>` with `ontoggle` (fires on open/close).
- `<svg>` with `onload` (a workhorse XSS vector — Chapter 14).
- Animation-driven handlers: CSS animations + `onanimationstart`/`onanimationend`,
  `<svg>` `<animate>`.

And the perennial favorites that need no user interaction:

- **`autofocus` + `onfocus`** on an `<input>`/`<textarea>`/`<select>` → zero-click trigger.
- **`<img onerror>`** with a broken `src` → fires immediately on parse.

For a tester, the takeaway is that the set of "things that can run code in markup" is far larger
than `<script>` and inline `on*` on a handful of tags. Modern filters that block `<script>` and
a few handlers routinely miss `ontoggle`, `onanimationstart`, media `onerror`, and
`autofocus`/`onfocus`.

## 13.2 `<iframe sandbox>` and `srcdoc` — both a defense and a vector

The `sandbox` attribute restricts what a framed document can do (disable scripts, forms,
popups, top-navigation, same-origin treatment) unless you re-enable capabilities with tokens
like `allow-scripts`, `allow-forms`, `allow-same-origin`, `allow-top-navigation`,
`allow-popups`, `allow-modals`.

Security relevance both ways:

- **As a defense:** sandboxing untrusted embedded content (ads, user HTML) limits blast radius.
  But a dangerous combination is **`allow-scripts allow-same-origin` together** — that lets the
  framed content run script *and* treat itself as same-origin, so it can reach out and remove its
  own sandbox or access the embedding origin's resources, defeating the point. Flag this combo.
- **As a vector:** `<iframe srcdoc="...">` lets you specify a whole document inline. If you can
  inject into a page, an injected `<iframe srcdoc="&lt;script&gt;...&lt;/script&gt;">` is a way
  to spawn an executing document — and `srcdoc` content is HTML-entity-decoded, which interacts
  with filters in useful (to an attacker) ways.

## 13.3 Web Storage: `localStorage` and `sessionStorage`

HTML5 added client-side key/value storage. Security implications:

- **No `HttpOnly` equivalent.** Anything in Web Storage is readable by *any* same-origin
  JavaScript, so **any XSS reads all of it.** Storing session tokens/JWTs in `localStorage` is
  convenient but means XSS = token theft. Cookies with `HttpOnly` at least resist JS reads.
- **Persistence.** `localStorage` survives browser restarts; a token or PII stored there lingers.
- **Stored DOM XSS.** If attacker-influenced data is written to storage and later read into a
  sink (Chapter 10), you get persistence without a server. Test read paths from storage.
- **Shared across tabs/subdomain nuances.** Storage is per-origin; understand the boundary when
  assessing leakage.

As a tester: inspect `localStorage`/`sessionStorage` (DevTools → Application) on authenticated
pages. Tokens or PII there is a finding, and it raises the severity of any XSS you find.

## 13.4 Cross-document messaging: `postMessage`

Covered as a DOM-XSS source in [Chapter 10](../part3/10-xss-dom.md), but it belongs in the HTML5
survey too. The two recurring bugs:

- **Receivers that don't check `event.origin`** → any page can drive the handler.
- **Senders that post to `'*'`** → messages (possibly containing secrets/tokens) are delivered
  to whatever origin currently occupies the target frame, leaking data to a malicious framer.

Audit both directions: who can send to this handler, and where does this sender's data go?

## 13.5 CORS: relaxing the Same-Origin Policy on purpose

Cross-Origin Resource Sharing lets a server opt in to cross-origin *reading* of its responses.
It's not HTML per se, but it changes the SOP boundary that all HTML attacks operate against.
Misconfigurations are common and impactful:

- **Reflective `Access-Control-Allow-Origin`** that echoes the request's `Origin` **plus**
  `Access-Control-Allow-Credentials: true` → any origin can read authenticated responses (mass
  data theft). This is a frequent critical finding.
- **`Access-Control-Allow-Origin: *` with credentials** is disallowed by spec, but **trusting
  `null`** (which sandboxed iframes and some redirects send as Origin) is a real bypass.
- **Weak origin matching** (substring/regex that allows `evil-target.example` or
  `target.example.evil.com`).

Test CORS by sending crafted `Origin` headers and observing the `Access-Control-*` response
headers. While adjacent to CSRF, CORS misconfig is about *reading*, which can be even worse.

## 13.6 Content embedding: `<object>`, `<embed>`, and media

Legacy-ish but still present:

- `<object>`/`<embed>` can load documents and historically plugins; they offer alternative
  vectors when `<iframe>`/`<script>` are filtered, and can load `data:`/SVG content.
- Media elements (`<video>`, `<audio>`) carry event handlers (§13.1) and can fetch cross-origin
  resources (CSRF-style GETs).

## 13.7 Navigation and link features: `target=_blank` and tabnabbing

A subtle but widespread issue:

```html
<a href="https://untrusted.example" target="_blank">open</a>
```

Historically, the newly opened page received a `window.opener` reference to the original page
and could redirect it (`window.opener.location = 'https://phish.example'`) — **reverse
tabnabbing**: the user clicks a link, a new tab opens, and meanwhile the *original* tab is
silently navigated to a phishing clone. Modern browsers now imply `rel="noopener"` for
`target=_blank`, mitigating this by default, but:

- Older browsers and some configurations still expose `opener`.
- Programmatic `window.open` may still pass an opener unless handled.

Recommend explicit `rel="noopener noreferrer"` on outbound `target=_blank` links and verify it's
present on links to untrusted destinations.

## 13.8 Form features: `formaction`, `autocomplete`, `pattern` (recap)

From Chapter 5, but reinforced as HTML5 surface:

- **`formaction`/`formmethod`/`formenctype`** on submit buttons override the form's target —
  injection or control of these can redirect submissions (credential theft) per button.
- **Client-side validation** (`pattern`, `required`, `type=email/url`) is advisory only.
- **`autocomplete`** behavior is a minor data-exposure consideration on sensitive fields.

## 13.9 Other capability APIs (relevant post-exploitation or for privacy)

These mostly require script execution to abuse (so they amplify XSS) or are gated by
permissions/secure contexts, but a tester should know they exist:

- **Service Workers** — scriptable network proxies that **persist**. An attacker who can run
  same-origin script (e.g., via XSS) can register a service worker for durable control of the
  origin's traffic, surviving the original XSS. Their availability dramatically raises XSS
  severity. Check whether the origin registers service workers and whether the registration
  surface is protected.
- **WebSockets / EventSource** — long-lived channels; check for missing origin validation on the
  server side (cross-site WebSocket hijacking) and for message-handling sinks.
- **Geolocation, Notifications, Camera/Mic (getUserMedia), Clipboard** — permission-gated; abuse
  generally needs script + user consent, but consent can be socially engineered.
- **Fullscreen API** — can aid UI-redress/phishing by hiding browser chrome.
- **Drag-and-drop API** — relevant to UI-redress (Chapter 12) data exfiltration.
- **`window.name`** — persists across navigations; a cross-origin source (Chapter 10).

## 13.10 The `<meta>` element's powers

`<meta>` is small but mighty and appears in injections:

- **`<meta http-equiv="refresh" content="0;url=...">`** — a script-free redirect; an open-redirect
  / phishing primitive when injectable.
- **`<meta http-equiv="Content-Security-Policy" content="...">`** — CSP can be set via meta;
  conversely, if an attacker can inject a `<meta>` CSP early, they might *weaken* policy in some
  cases, or you might use a meta CSP as a remediation where headers are hard to set.
- **`<meta name="referrer" content="...">`** — controls `Referer` sending; relevant to
  token/secret leakage and to suppressing `Referer` for `Origin`/`Referer`-based CSRF checks.
- **`<meta charset>`** — encoding control (Chapter 3 charset attacks).

## 13.11 Testing checklist for HTML5 surface

- [ ] Are non-`<script>` event-handler vectors filtered? (`ontoggle`, media `onerror`,
      `onanimationstart`, `autofocus`+`onfocus`).
- [ ] Any `<iframe sandbox>` with `allow-scripts allow-same-origin` together? Any injectable
      `srcdoc`?
- [ ] Are tokens/PII stored in `localStorage`/`sessionStorage`? (raises XSS impact)
- [ ] Do `postMessage` receivers check `origin`? Do senders post to `'*'`?
- [ ] Is CORS reflective/credentialed/`null`-trusting/weak-matching?
- [ ] Do `target=_blank` links to untrusted sites use `rel="noopener"`?
- [ ] Are `formaction`/`formmethod` overrides exploitable?
- [ ] Does the origin register **service workers**? Is that surface protected?
- [ ] Any injectable `<meta http-equiv>` (refresh/CSP/referrer)?

## 13.12 Fixing HTML5-feature risks (pointers)

- Filter/escape on output regardless of element type; don't enumerate "bad tags" — allow-list.
- Never combine `allow-scripts` and `allow-same-origin` for untrusted sandboxed content.
- Prefer **cookies with `HttpOnly`+`Secure`+`SameSite`** over `localStorage` for session tokens.
- Validate `event.origin` in `postMessage` handlers; target a specific origin when sending.
- Lock down CORS: explicit allow-list, never reflect arbitrary origins with credentials, don't
  trust `null`.
- Add `rel="noopener noreferrer"` to outbound `target=_blank` links.
- Protect service-worker and WebSocket surfaces (origin checks server-side).
- Set a strong CSP (Chapter 19) — it constrains many of these vectors at once.

---

## Key takeaways

- HTML5 multiplied the set of **markup-based code-execution vectors** beyond `<script>`
  (`ontoggle`, media `onerror`, `onanimationstart`, `autofocus`+`onfocus`, `<svg onload>`).
- **`localStorage`/`sessionStorage`** have no `HttpOnly` analog — any XSS reads them; storing
  tokens there raises XSS severity.
- **`postMessage`** and **CORS** redraw the cross-origin boundary; missing origin checks and
  reflective/credentialed CORS are high-impact misconfigurations.
- **`iframe sandbox`** helps, but `allow-scripts allow-same-origin` together defeats it;
  `srcdoc` is also an injection vector.
- Watch **tabnabbing** (`target=_blank` → `rel=noopener`), **`<meta>`** powers (refresh/CSP/
  referrer/charset), and **service workers** (durable post-XSS persistence).

---

[← Previous: Chapter 12 — Clickjacking and UI Redressing](12-clickjacking.md) | [Next: Chapter 14 — Dangerous Uploads, SVG, and Markup Tricks →](14-uploads-svg-markup.md)
