# Chapter 18 — Secure Output Handling and Sanitization

> Every offensive chapter ended with "how to fix it." This chapter assembles those fixes into a
> coherent defensive doctrine. The single most important idea — repeated throughout the book — is
> finally stated in full: **treat data as data at the moment it becomes part of a page, in
> whatever context that is.**

---

## 18.1 The hierarchy of defenses

No single control stops everything. Defense is layered, in roughly this order of primacy:

1. **Context-appropriate output encoding** — the primary, near-universal fix for injection/XSS.
2. **Safe APIs by construction** — frameworks and sinks that don't allow injection in the first
   place (`textContent`, parameterized templates, Trusted Types).
3. **Input validation** — allow-list expected formats as a *secondary* measure.
4. **HTML sanitization** — for the special case where you must allow *some* user HTML.
5. **Content Security Policy** ([Chapter 19](19-csp.md)) — defense in depth that limits impact
   when the above fail.
6. **Security headers & cookie flags** — reduce blast radius and enable browser protections.

The rest of this chapter works through 1–4 and 6; CSP gets its own chapter.

## 18.2 Output encoding is contextual (the core doctrine)

From [Chapter 2](../part1/02-html-refresher.md): the right encoding depends entirely on the
context where data lands. "Encode everything" is not a plan; "encode *for the context*" is. The
canonical contexts and their encodings:

| Context | Example | Required encoding |
|---|---|---|
| HTML element content | `<div>DATA</div>` | HTML-entity encode `< > & " '` |
| Quoted attribute value | `<input value="DATA">` | HTML-entity encode (esp. the quote in use) |
| Unquoted attribute value | `<input value=DATA>` | **Don't.** Always quote attributes; then entity-encode |
| URL in `href`/`src` | `<a href="DATA">` | URL-validate (scheme allow-list) **then** attribute-encode |
| JavaScript string | `<script>var x="DATA"</script>` | JavaScript string escaping (`\xHH`/unicode), not HTML-entity |
| CSS value | `<div style="width:DATA">` | CSS escaping; better: don't put untrusted data in CSS |
| HTML comment | `<!-- DATA -->` | Don't place untrusted data here |

Two failure modes this table prevents:
- **Right idea, wrong context:** HTML-encoding a value that lands in a `<script>` or `href`
  (Chapter 8) — encoding the wrong characters does nothing useful there.
- **Unquoted attributes:** even encoded data can break out of an unquoted attribute via a space;
  always quote.

The practical rule for developers: **encode at the point of output, choosing the encoder for the
exact sink.** Mature template engines provide context-aware auto-escaping that does this for you
(§18.4).

## 18.3 Why "input sanitization" is the wrong primary defense

A common but fragile approach is to "clean" input on the way in (strip `<script>`, remove
quotes). This fails because:

- **Context isn't known at input time.** The same value may later be rendered in element content,
  an attribute, a script, and a URL — each needs different treatment.
- **Blocklists have holes** (Chapter 14's whole catalog).
- **It corrupts legitimate data** (a user named `O'Brien`, a comment about `<html>`).
- **Second-order bugs slip through** (Chapter 9): data deemed "clean" is later rendered unsafely.

Input *validation* (allow-listing expected formats) is valuable as defense in depth, but it is
**not a substitute** for output encoding. Encode on output, validate on input, never rely on
input cleaning alone.

## 18.4 Lean on framework auto-escaping — and don't disable it

Modern frameworks encode by default in templates:

- **React** escapes values in JSX by default; the danger is `dangerouslySetInnerHTML`.
- **Angular** treats interpolated values as untrusted and escapes them; danger is
  `bypassSecurityTrust*` and `[innerHTML]` with unsanitized data.
- **Vue** escapes `{{ }}` interpolation; danger is `v-html`.
- **Server-side templates** (Jinja2, Twig, Razor, ERB, Go `html/template`, Thymeleaf, etc.)
  auto-escape — *if* you don't use their "raw"/`|safe`/`{{{ }}}`/`Html.Raw` escape hatches.

**The recurring vulnerability is the escape hatch.** When you (as a tester) see
`dangerouslySetInnerHTML`, `v-html`, `|safe`, `{% autoescape false %}`, `Html.Raw`, or
`mark_safe` applied to data with any user influence, flag it. As a developer, never feed
untrusted data to these without sanitizing first (§18.6). Notably, Go's `html/template` is
*context-aware* — it picks the right encoding for element/attribute/JS/URL contexts
automatically — which is the gold standard to emulate.

## 18.5 Safe DOM APIs (client-side)

For client-rendered content (the DOM-XSS surface, Chapter 10), choose sinks that can't inject:

- **Use `textContent`/`innerText`** instead of `innerHTML` for text.
- **Build nodes with DOM APIs** (`createElement`, `append`, `setAttribute` for non-URL attrs)
  rather than HTML string concatenation.
- **For URLs**, validate the scheme before assigning to `href`/`src`/`location` (§18.7).
- **Never** pass untrusted strings to `eval`, `Function`, `setTimeout`/`setInterval` (string
  form), or `document.write`. Parse JSON with `JSON.parse`.
- **Validate `postMessage`**: check `event.origin` against an allow-list and validate the data
  shape.

The structural enforcement of all this is **Trusted Types** (§18.9 / Chapter 19), which makes the
dangerous sinks refuse raw strings.

## 18.6 Sanitizing rich text the right way

Some features legitimately need to render user HTML (comments, WYSIWYG, markdown). Here you can't
just encode — you must allow *some* tags. Rules:

1. **Never hand-roll a sanitizer.** Use a **vetted, actively maintained** library designed for
   this. On the client, the widely used choice is **DOMPurify**; server-side, use a
   well-maintained allow-list sanitizer for your language (e.g., OWASP Java HTML Sanitizer, or
   ammonia/bleach-style libraries — verify current maintenance status for your stack).
2. **Allow-list, don't block-list** tags and attributes. Start from "nothing allowed" and add the
   minimum (`<b>`, `<i>`, `<a href>` with scheme checks, `<p>`, lists...).
3. **Be aware of mutation XSS** (Chapter 3): choose a sanitizer hardened against mXSS and keep it
   updated; mXSS bypasses are found and fixed over time.
4. **Sanitize as close to output as possible**, and **re-sanitize if you transform** the HTML
   afterward (linkify, image-proxy rewriting) — post-processing can reintroduce vulnerabilities
   (Chapter 9).
5. **Handle SVG/foreign content explicitly** (Chapter 14) — many sanitizers need configuration to
   safely allow or strip SVG.
6. **Pair with CSP and Trusted Types** so a sanitizer bypass isn't game over.

For **markdown**, configure the renderer to **not** pass through raw HTML (or to run its output
through a sanitizer), and validate link schemes so `[x](javascript:...)` can't produce a
dangerous `href`.

## 18.7 URL/scheme validation

Whenever untrusted data becomes a URL (in `href`, `src`, `action`, `formaction`, redirects,
`location`):

- **Allow-list schemes**: typically permit only `http`, `https`, `mailto` (and `tel` where
  relevant); **reject** `javascript:`, `data:`, `vbscript:`, and unknown schemes.
- **Canonicalize first** (decode entities/encodings, trim whitespace/control chars) so tricks
  like `java\tscript:` or `javascript&#58;` can't slip through after your check.
- **For redirects**, allow-list destinations (or use server-side mapping keys) rather than
  reflecting an arbitrary `returnUrl` — this kills open redirect and its escalations
  (Chapter 10).
- **After** scheme validation, still apply the surrounding context's encoding (e.g.,
  attribute-encode the validated URL).

## 18.8 CSRF defenses (consolidated)

From [Chapter 11](../part4/11-csrf.md), the layered fix:

- **Anti-CSRF tokens:** unpredictable, **bound to the user's session**, validated server-side on
  **every** state-changing request, and **rejected when missing or empty** (not merely "checked
  if present").
- **`SameSite` cookies:** default session cookies to `SameSite=Lax` (or `Strict` where UX
  allows). Avoid `SameSite=None` unless genuinely cross-site; if used, pair with tokens.
- **No state changes over GET.** Use POST/PUT/DELETE with protection.
- **Validate `Origin`/`Referer`** server-side and **fail closed** on cross-site or missing-header
  conditions for sensitive actions.
- **Require non-forgeable request shapes** for sensitive APIs (enforced JSON content type + a
  custom header) so plain HTML forms can't reproduce them.
- **Re-authenticate** for the highest-impact actions (password/email change, large transfers).
- **Cover the login form too** (login CSRF).

## 18.9 Framing, headers, and cookie flags (consolidated)

Browser-enforced controls that reduce impact and block whole attack classes:

- **`Content-Security-Policy: frame-ancestors 'none'|'self'|<list>`** — anti-clickjacking
  (Chapter 12); add **`X-Frame-Options: DENY/SAMEORIGIN`** for legacy browsers.
- **`X-Content-Type-Options: nosniff`** — stops MIME sniffing turning uploads/echoes into HTML
  (Chapter 14).
- **Correct `Content-Type`** with explicit `charset=utf-8`; serve APIs as JSON, downloads with
  `Content-Disposition: attachment`, user files from a **separate origin** (Chapter 14).
- **Cookie flags:** `HttpOnly` (hide session cookies from JS — limits XSS looting),
  `Secure` (HTTPS only), `SameSite` (CSRF). Prefer cookies over `localStorage` for session tokens
  (Chapter 13).
- **`Strict-Transport-Security` (HSTS)** — enforce HTTPS (supports cookie `Secure` and prevents
  downgrade).
- **`Referrer-Policy`** — limit token/secret leakage via `Referer`.
- **CSP** generally (Chapter 19) — the big one for XSS impact reduction.

## 18.10 Trusted Types: structurally killing DOM XSS

Reiterated from Chapter 10 because it's the strongest client-side fix:
`Content-Security-Policy: require-trusted-types-for 'script'` makes dangerous DOM sinks
(`innerHTML`, `script.src`, etc.) reject plain strings; data must pass through a named
**Trusted Types policy** you define (typically wrapping a sanitizer like DOMPurify). This turns a
sprawling, hard-to-audit set of sinks into a small number of central, reviewable policies — the
difference between "hope no one wrote an unsafe `innerHTML`" and "the browser enforces that they
can't." Adopt it for high-risk apps; full coverage in [Chapter 19](19-csp.md).

## 18.11 Building secure features: worked patterns

**Displaying a username (element content):**
```jsx
// React — safe by default
<span>{username}</span>          // auto-escaped; no action needed
```
```python
# Jinja2 — safe by default
<span>{{ username }}</span>      {# autoescaped; do NOT add |safe #}
```

**Rendering a user-supplied link (URL context):**
```js
const ALLOWED = new Set(['http:', 'https:', 'mailto:']);
function safeHref(raw) {
  try {
    const u = new URL(raw, location.origin);   // canonicalize
    return ALLOWED.has(u.protocol) ? u.href : '#';
  } catch { return '#'; }
}
anchor.setAttribute('href', safeHref(userInput));
```

**Rendering rich text (must allow some HTML):**
```js
// Client-side, with DOMPurify + (ideally) a Trusted Types policy
container.innerHTML = DOMPurify.sanitize(userHtml, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'ul', 'ol', 'li'],
  ALLOWED_ATTR: ['href'],
});
// Plus: validate href schemes, serve under a strict CSP.
```

**Protecting a state-changing endpoint (CSRF):**
- Generate a session-bound token; embed it in the form; verify on the server; reject if
  missing/empty/mismatched. Set the session cookie `HttpOnly; Secure; SameSite=Lax`. Don't accept
  the action over GET.

## 18.12 Remediation advice for reports

When you write remediation in a finding (Chapter 15), be **specific to the stack and context**:

- Don't write "sanitize input." Write: "HTML-entity-encode this value, which is rendered in HTML
  element content; in this Jinja2 template, remove the `|safe` filter so auto-escaping applies."
- For DOM XSS: "Replace `el.innerHTML = userVal` with `el.textContent = userVal`; if HTML is
  required, run it through DOMPurify and adopt a Trusted Types policy."
- For CSRF: "Add a session-bound anti-CSRF token validated server-side and rejected when absent;
  set `SameSite=Lax` on the session cookie."
- For clickjacking: "Add `Content-Security-Policy: frame-ancestors 'none'` (and
  `X-Frame-Options: DENY`) to responses for this sensitive page."

Specific, contextual remediation is what gets bugs actually fixed.

---

## Key takeaways

- The primary fix for injection/XSS is **context-appropriate output encoding at the point of
  output** — not input cleaning, and not "encode everything."
- **Lean on framework auto-escaping**; the recurring bug is the **escape hatch**
  (`dangerouslySetInnerHTML`, `v-html`, `|safe`, `Html.Raw`). Go's context-aware `html/template`
  is the model.
- Use **safe DOM APIs** (`textContent`, DOM construction, no `eval`); validate `postMessage`.
- For rich text, use a **vetted allow-list sanitizer** (e.g., DOMPurify) close to output,
  re-applied after transforms, and pair it with CSP/**Trusted Types**.
- **Validate URL schemes** (allow-list, canonicalize first) to kill `javascript:`/open-redirect.
- Layer **CSRF tokens + `SameSite`**, **framing controls**, **`nosniff`/correct content types**,
  and **cookie flags**. Write **stack-specific** remediation in reports.

---

[← Previous: Chapter 17 — Hands-On Labs and Exercises](../part5/17-labs.md) | [Next: Chapter 19 — Content Security Policy in Depth →](19-csp.md)
