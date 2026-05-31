# Chapter 2 — HTML Refresher for Security Testers

> You probably know HTML well enough to build a page. This chapter re-introduces it through
> the only lens that matters for testing: **where does untrusted data go, and what can it
> become once it's there?**

---

## 2.1 HTML is a set of *contexts*, not just tags

The single most useful mental model for HTML security is this: an HTML document is made up of
distinct **contexts**, and the rules for what is "dangerous" change completely depending on
which context a piece of data lands in. The same character — say, `"` or `<` — is harmless in
one context and an injection point in another.

The contexts you must be able to recognize on sight:

1. **HTML element content** — text between tags: `<p>HERE</p>`
2. **HTML attribute value** — inside a tag's attribute: `<img alt="HERE">`
3. **Unquoted attribute value** — `<img alt=HERE>`
4. **URL / `href` / `src` context** — `<a href="HERE">`
5. **Inline event handler** — `<button onclick="HERE">`
6. **Inline `<script>` context** — `<script>var x = "HERE";</script>`
7. **Inline `<style>` / CSS context** — `<div style="HERE">`
8. **Comment context** — `<!-- HERE -->`
9. **Tag name / attribute name context** — `<HERE href=...>`

When you find a reflection (your input showing up in the response), your first job is **not**
to fire payloads — it's to **identify which of these contexts you landed in.** The payload
that works is entirely determined by the context and by what the application did or didn't
encode. We'll return to this constantly; it's the backbone of [Part III](../part3/07-html-injection.md).

## 2.2 The anatomy of an element

```html
<a href="https://example.com" class="link" data-id="42">Visit</a>
```

- `<a ...>` — the **start tag**.
- `a` — the **tag name** (also called the element name).
- `href`, `class`, `data-id` — **attributes** (name/value pairs).
- `"https://example.com"` — an **attribute value** (here, quoted with double quotes).
- `Visit` — the **text content**.
- `</a>` — the **end tag**.

Security-relevant observations:

- **Attribute values can be quoted with double quotes, single quotes, or left unquoted.** Each
  has a different "escape character" you'd need to break out. Double-quoted: a `"` ends it.
  Single-quoted: a `'` ends it. Unquoted: a space, tab, newline, or several other characters
  end it. Knowing which the app used tells you exactly what character you need to inject.
- **Attribute names are case-insensitive** and **boolean attributes** (`disabled`, `checked`,
  `autofocus`) need no value. `autofocus` in particular is a building block of many XSS payloads
  because it triggers focus-related events without user interaction.
- **`data-*` attributes** hold arbitrary application data and are frequently read by JavaScript
  — a common source of DOM-based issues (see [Chapter 10](../part3/10-xss-dom.md)).

## 2.3 Document structure

A minimal modern document:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Example</title>
  </head>
  <body>
    <h1>Hello</h1>
  </body>
</html>
```

Why each part matters to a tester:

- **`<!DOCTYPE html>`** switches the browser into *standards mode*. Its absence triggers
  *quirks mode*, which changes layout and, historically, some security-relevant behaviors.
  You'll occasionally see quirks mode relevant to clickjacking and CSS-based attacks.
- **`<meta charset>`** sets the character encoding. **This is a security control.** If the
  declared charset differs from how the server actually encodes bytes, an attacker can
  sometimes smuggle characters through filters using multibyte/alternate encodings (e.g.,
  UTF-7 in legacy browsers, or charset confusion). Always note the declared charset.
- **`<head>`** is where `<meta>`, `<link>`, `<base>`, and `<script>`/`<style>` often live.
  The `<base href>` element is especially interesting: it rewrites how *relative* URLs on the
  page resolve, so an injected `<base>` can hijack every relative link/resource on a page.
- **`<title>`** is an interesting *reflection sink* because content inside `<title>` is parsed
  in a special way (it's "RCDATA" — more in [Chapter 3](03-browser-parsing.md)).

## 2.4 Elements that carry security weight

Some tags matter far more to a tester than others. Keep this short list in your head:

| Element | Why it matters |
|---|---|
| `<script>` | Executes JavaScript. The classic XSS payload host. |
| `<iframe>` | Embeds another document; central to clickjacking and `srcdoc` tricks. |
| `<img>` | `onerror` handler + arbitrary `src` = no-`<script>` XSS and SSRF-ish fetches. |
| `<svg>` | A whole XML island with its own script vectors and event handlers. |
| `<a>` | `href` can be `javascript:` or `data:`; target of open-redirect and tabnabbing. |
| `<form>` | The engine of CSRF; `action`/`formaction` redirection. |
| `<input>` | Wide attack surface via `type`, `value`, `autofocus`, `onfocus`, `formaction`. |
| `<base>` | Rewrites relative URL resolution for the whole page. |
| `<meta>` | `http-equiv="refresh"` enables redirects; CSP can be set here. |
| `<object>`/`<embed>` | Legacy plugin/content embedding; alternative script vectors. |
| `<link>` | Preload/prefetch/stylesheet loading; can leak and can be abused. |
| `<style>` | CSS injection, data exfiltration via selectors, UI redressing. |
| `<template>`,`<noscript>`,`<textarea>` | Special parsing modes that defeat naive filters. |

You don't need to memorize behaviors yet — just recognize that these elements are where the
action is. Each gets detailed treatment later.

## 2.5 Character references (HTML entities) — the heart of output encoding

HTML lets you represent characters by reference so they are treated as *text*, not *markup*:

- Named: `&lt;` (`<`), `&gt;` (`>`), `&amp;` (`&`), `&quot;` (`"`), `&#39;`/`&apos;` (`'`)
- Numeric (decimal): `&#60;` is `<`
- Numeric (hex): `&#x3C;` is `<`

This is the mechanism behind **output encoding**, the primary defense against HTML injection
and XSS. When an application correctly encodes `<` as `&lt;` before placing user data into
HTML element content, the browser renders a literal less-than sign instead of starting a tag.

The crucial subtleties a tester exploits:

- **Encoding must match the context.** HTML-entity encoding protects *element content* and
  *quoted attribute values*, but it does **not** make data safe inside a `<script>` block,
  inside an event handler, or inside a URL. Developers who "HTML-encode everything" still get
  popped because they used the wrong encoding for the context. (We'll exploit exactly this.)
- **Browsers decode entities in many places**, including some attribute values and `href`s.
  That means `javascript&#58;alert(1)` can decode to `javascript:alert(1)` *after* a naive
  filter looked for the literal string `javascript:`.
- **Double-decoding bugs.** If data is decoded twice (once by a framework, once by the
  browser), `&amp;lt;` can become `<`. Layered decoding is a rich source of filter bypasses.

If you take one thing from this chapter, take this: **encoding is context-dependent, and
mismatches between the encoding applied and the context the data lands in are where
vulnerabilities live.**

## 2.6 Comments are not a safe place to hide

```html
<!-- user input here -->
```

Developers sometimes drop data into comments thinking it's inert. But:

- A payload containing `-->` closes the comment early and escapes back into normal parsing.
- Conditional comments and malformed comment syntax have historically had quirky handling.

Treat comments as just another reflection context that can be broken out of.

## 2.7 Raw-text and escapable-raw-text elements

A few elements don't treat their contents as normal HTML:

- **`<script>` and `<style>`** are *raw text* elements. Inside them, `<` does **not** start a
  new tag. The only thing that ends a `<script>` block is the literal sequence `</script`
  (with some nuances covered in [Chapter 3](03-browser-parsing.md)). This is why "just strip
  `<` and `>`" fails to secure a value reflected inside a script, and why the real escape route
  out of a script context is often `</script>` itself.
- **`<textarea>` and `<title>`** are *escapable raw text* (RCDATA): they treat `<` as text but
  still decode character references. So `<textarea>` content is safer from tag injection but a
  `</textarea>` sequence breaks out, and entities are still decoded.

These special modes are a recurring theme: **the rules change inside certain elements, and
filters that assume "normal" HTML rules can be bypassed by getting into or out of these
modes.**

## 2.8 Void elements and self-closing confusion

Some elements never have content or an end tag — `<img>`, `<input>`, `<br>`, `<meta>`,
`<link>`, `<hr>`, and others. These are **void elements**. In HTML (as opposed to XML/XHTML),
the self-closing slash `<img />` is allowed but ignored — the element is void regardless.

Why a tester cares: developers and naive sanitizers sometimes reason about HTML as if it were
XML (strict nesting, mandatory closing). Browsers don't. The gap between "how a regex or
XML-minded filter thinks HTML works" and "how the browser actually parses it" is a primary
source of bypasses. Hold that thought — it's the whole point of [Chapter 3](03-browser-parsing.md).

## 2.9 URLs inside HTML

Many attributes take a URL: `href`, `src`, `action`, `formaction`, `cite`, `data`,
`poster`, `background`, and more. URLs have their own pseudo-schemes that matter:

- **`javascript:`** — executes script when navigated (e.g., a clicked `<a href="javascript:...">`).
- **`data:`** — inlines content; `data:text/html,...` can host a whole document, and was a
  classic phishing/XSS vector (now restricted for top-level navigation in modern browsers).
- **`vbscript:`** — legacy IE only; effectively dead but appears in old payload lists.
- **`blob:`** and **`filesystem:`** — programmatically created resources.
- **Relative vs absolute** — resolution depends on the document base (and on `<base href>`).

A reflection that lands in a URL context is dangerous in ways that HTML-entity encoding does
**not** fix: `&quot;` won't stop `javascript:alert(1)` from being a valid `href`. URL contexts
need URL-aware validation (scheme allow-listing), which we cover in
[Chapter 18](../part6/18-defense-remediation.md).

## 2.10 A first taste: the same input, six outcomes

Imagine an application reflects the parameter `q` into different places. Suppose `q` is the
string `"><svg onload=...>` (kept abstract here). Whether anything dangerous happens depends
entirely on context and encoding:

| Where `q` is reflected | Was it encoded? | Result |
|---|---|---|
| `<p>q</p>` | HTML-encoded | Safe — shows as literal text |
| `<p>q</p>` | Not encoded | HTML injection / XSS possible |
| `<input value="q">` | `"` encoded | Safe |
| `<input value="q">` | `"` not encoded | Break out of attribute → XSS |
| `<script>x="q"</script>` | HTML-encoded only | Still XSS — wrong encoding for context |
| `<a href="q">` | HTML-encoded | `javascript:` may still execute → XSS |

Notice that two rows are vulnerable *despite* the developer applying HTML encoding — because
the encoding didn't match the context. Internalize this table; the rest of the book is, in a
sense, an expansion of it.

## 2.11 Tester's HTML reading checklist

When you view source (or proxy a response), scan for:

- [ ] Where does my input appear? (Use a unique marker like `zqxj123`.)
- [ ] What **context** is each reflection in (content / attribute / script / URL / style / comment)?
- [ ] What **encoding**, if any, was applied? Test with `<`, `>`, `"`, `'`, `` ` ``, and see what survives.
- [ ] Are there special parsing modes nearby (`<script>`, `<textarea>`, `<title>`, `<svg>`)?
- [ ] Does the response declare a charset, and does it match the bytes?
- [ ] Is there a `<base>` tag, a CSP header/meta, or framing controls?

This checklist is your first move on every reflection. The next chapter explains *why* it
works — by getting inside the browser's parser.

---

## Key takeaways

- Think in **contexts**, not tags. The danger of a character depends entirely on where it lands.
- **Output encoding is context-specific.** HTML-entity encoding protects element content and
  quoted attributes but not scripts, event handlers, or URLs.
- Certain elements (`<script>`, `<style>`, `<textarea>`, `<title>`, `<svg>`) have special
  parsing rules that break naive filters.
- URL contexts need scheme allow-listing, not entity encoding.
- Always identify context + encoding *before* choosing a payload.

---

[← Previous: Chapter 1 — Legal Foundations and Ethics](../01-legal-and-ethics.md) | [Next: Chapter 3 — How Browsers Parse HTML →](03-browser-parsing.md)
