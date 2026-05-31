# Chapter 14 — Dangerous Uploads, SVG, and Markup Tricks

> This chapter collects the "markup that arrives through unexpected doors" attacks: files that
> the browser later parses as HTML, SVG's deceptively rich scripting surface, and a grab-bag of
> parser tricks that defeat naive filters. The unifying theme: **any path that lets attacker
> bytes be interpreted as markup in the target origin is an XSS path.**

---

## 14.1 Why uploads are an HTML problem

A file upload feels unrelated to HTML — until you ask: *how is the file served back, and how
does the browser interpret it?* If an attacker can upload content that the browser later renders
as HTML **in the target's origin**, that's stored XSS by another name. The danger points:

1. **Content type the server assigns when serving the file.** If an uploaded `.html` (or a file
   the server labels `text/html`) is served from the app's origin, opening it runs its script
   in that origin.
2. **MIME sniffing.** Without `X-Content-Type-Options: nosniff`, browsers may *sniff* a file's
   content and decide it's HTML even if the server said otherwise — turning an "image" that
   actually contains `<script>` into an HTML page.
3. **SVG and HTML-bearing formats** that are legitimately served inline (next sections).
4. **Filename and metadata** reflected into listings (stored XSS via filename).
5. **Storage location/origin.** Files served from the *same origin* as the app are dangerous;
   files served from a separate sandbox origin or with `Content-Disposition: attachment` are
   safer.

## 14.2 The `accept` attribute is not a control

`<input type="file" accept="image/*">` only *hints* to the file picker. It does nothing
server-side and is trivially bypassed by submitting any file via a proxy. Likewise,
client-side checks of file extension or the picker's MIME are advisory. **All upload validation
must be server-side**, and content type must be verified by inspecting bytes, not by trusting the
client-declared type or extension.

## 14.3 Polyglots and content-type confusion

Attackers craft files that are **valid as two formats at once** (polyglots) so they pass an
"is this a real image?" check yet still get interpreted as HTML/script when served or sniffed:

- A **GIF/JPEG with HTML/JS appended or embedded** that, if served as `text/html` or sniffed,
  executes. Image parsers ignore trailing junk; HTML parsers find the markup.
- A file whose **magic bytes** say "image" but whose body contains `<script>` — defeats checks
  that only validate the header.

The defense combinations (see §14.9) are: validate content properly, set correct
`Content-Type`, send `nosniff`, force `Content-Disposition: attachment` for downloads, and serve
user files from a separate, non-app origin.

## 14.4 SVG: an XML island that scripts

SVG is the single most important "markup upload" format for a tester, because **SVG is XML that
can contain script and event handlers**, and it's frequently allowed where other HTML isn't
(avatars, logos, diagrams) and served inline.

A minimal scripting SVG (lab demonstration only, against authorized targets):

```xml
<svg xmlns="http://www.w3.org/2000/svg">
  <script>/* runs when the SVG is rendered as a document in the app's origin */</script>
</svg>
```

Event-handler variants that work even where `<script>` is stripped:

```xml
<svg xmlns="http://www.w3.org/2000/svg" onload="/* code */"></svg>
<svg><a xlink:href="javascript:/* code */"><rect width="100" height="100"/></a></svg>
<svg><animate onbegin="/* code */" attributeName="x" dur="1s"/></svg>
```

Critical nuances:

- **Context of rendering matters.** An SVG loaded as an `<img src="x.svg">` does **not** run its
  script. But an SVG opened **as a top-level document** (navigating to `https://app.example/uploads/x.svg`)
  or embedded via `<object>`/`<iframe>`/inline `<svg>` in HTML **does** run script in that
  origin. So the question is always: *how does the app let users reach the SVG?*
- **`<use>` and external references** can pull in remote/embedded fragments.
- **Foreign-content parsing** (Chapter 3 §3.6) makes SVG a hotspot for **mutation XSS** and for
  bypassing HTML sanitizers that mishandle the HTML↔SVG boundary.

As a tester: if an app accepts SVG uploads (or SVG in rich text) and can serve/render it inline
or as a top-level document from its own origin, test for script execution. If it accepts SVG but
serves it only as `<img>` with `nosniff` and `Content-Disposition: attachment` from a separate
origin, the risk drops sharply.

## 14.5 XML-adjacent risks that ride on SVG/markup

Because SVG is XML, XML attack classes can appear:

- **XXE (XML External Entity).** If the server *parses* uploaded SVG/XML with a misconfigured
  parser, external-entity declarations can read local files or trigger SSRF. This is a
  server-side bug surfaced through a markup upload. Test SVG/XML inputs with entity declarations
  where parsing occurs.
- **Billion-laughs / entity-expansion DoS** on naive parsers.

These are out of the pure-HTML lane but commonly reached *through* an SVG upload field, so keep
them in mind when you see "upload an SVG/XML" features.

## 14.6 Filename and metadata injection

The file's **name** and **metadata** are attacker-controlled strings that often get rendered:

- A filename like `"><svg onload=...>.png` reflected unencoded into an upload listing or an
  admin file browser is stored XSS via filename. (Also test path traversal: `../../` in
  filenames.)
- **EXIF/metadata** fields (camera, comments, geotags) displayed in a gallery UI can carry
  payloads.
- **Content-Disposition filename** handling and download dialogs can be abused on some stacks.

Treat filenames and metadata as untrusted input subject to the same context/encoding analysis as
any other reflection (Part III).

## 14.7 Markup tricks that defeat naive filters (catalog)

A consolidated reference of parser-level tricks (mechanisms explained in Chapter 3). Use only
against authorized targets; these illustrate *why blocklist filtering fails*.

- **Case and whitespace:** `<ScRiPt>`, `<img/src=x/onerror=...>`, tabs/newlines/`/` as attribute
  separators.
- **Recursive strip:** `<scr<script>ipt>` against single-pass tag removers.
- **Attribute-context entity decoding:** `<a href="javascript&#58;...">`, `&#x6a;avascript:`.
- **Protocol obfuscation:** `java\tscript:`, `JaVaScRiPt:`, leading/trailing whitespace and
  control chars in URLs.
- **No-`<script>` vectors:** `<svg onload>`, `<img onerror>`, `<details ontoggle open>`,
  `<input autofocus onfocus>`, media `onerror`, CSS-animation `onanimationstart`.
- **Special elements / RCDATA breakouts:** `</textarea>`, `</title>`, `</style>`, `</script>`.
- **Foreign content / mXSS:** SVG/MathML structures and sanitizer round-trip mismatches.
- **Comment breakouts:** `--><svg onload=...>`.
- **`<base>` hijack:** inject `<base href="//attacker/">` to repoint relative URLs (Chapter 5).
- **`<noscript>`/`<template>` quirks:** parsing differs when script is disabled or content is
  inert-then-activated.
- **Charset/encoding:** UTF-7 and charset confusion on legacy stacks (Chapter 3 §3.9).
- **Dangling markup:** unterminated attribute to swallow page bytes (Chapter 7 §7.5), useful when
  script is blocked.

Each entry is a manifestation of the same root cause: a filter modeling HTML differently than the
browser parses it. A maintained allow-list sanitizer + CSP defeats the whole category far better
than chasing individual tricks.

## 14.8 Detecting these issues (procedure)

**Uploads:**
1. Find every upload feature; note allowed types and how files are *served back* (origin,
   `Content-Type`, `nosniff`, `Content-Disposition`).
2. Bypass `accept`/extension/MIME client checks via a proxy; submit a file the server shouldn't
   accept.
3. Try **HTML** and **SVG** uploads; if served inline from the app origin, test for script
   execution by navigating to the file or finding where it renders.
4. Try **polyglots** and content-type confusion; check whether sniffing turns "images" into HTML.
5. Inject payloads into **filenames** and **metadata**; check listings/admin views.
6. Where XML/SVG is parsed server-side, test for **XXE**.

**Markup tricks:** when you hit a filter (Part III), characterize it (Chapter 7 §7.7) and walk
the catalog above until you find an unhandled case — then report the *class* of weakness, not
just the one payload.

## 14.9 Fixing uploads and markup risks

**Uploads:**
- **Validate server-side by content**, not extension/`accept`/client MIME. Re-encode images
  through a trusted library to strip embedded payloads where feasible.
- **Set a correct, restrictive `Content-Type`** and **`X-Content-Type-Options: nosniff`** on all
  served files.
- **Serve user files from a separate origin** (a different domain, not just a path) so even if
  they're HTML/SVG they can't script the app origin; and/or force
  `Content-Disposition: attachment` for downloads.
- **Disallow inline rendering of SVG/HTML** unless absolutely required; if SVG must render
  inline, **sanitize it** with a vetted SVG-aware sanitizer and serve under a strong CSP.
- **Randomize stored filenames** and encode any displayed filename/metadata.
- **Harden XML parsers**: disable external entities and DTD processing to prevent XXE.

**Markup/sanitization:**
- Use a **vetted, maintained allow-list HTML sanitizer**; never hand-roll blocklists.
- Apply a strong **CSP** (Chapter 19) as defense in depth — it blunts most of the trick catalog.
- Encode output by **context** (Part III / Chapter 18).

## 14.10 Lab exercise

In your lab (Juice Shop and DVWA both have upload features; or build a small upload endpoint):

1. Upload an SVG containing a benign `onload` and try to reach it as a top-level document from
   the app origin. Note whether it executes. Then re-test when served from a separate origin /
   with `nosniff` + `attachment` and observe it neutralized.
2. Craft an `<svg onload>` payload to bypass a filter that strips `<script>`.
3. Upload a file with a payload in its **filename** and find where the listing reflects it.
4. If your lab has an XML/SVG parser, test a benign XXE entity that reads a harmless local file
   you placed for the test (lab only).
5. Take one filter from Part III and bypass it using three different entries from the §14.7
   catalog; note that each exploits a parser/filter mismatch.

---

## Key takeaways

- An upload is an XSS path whenever attacker bytes are later **interpreted as markup in the
  target origin** — driven by served `Content-Type`, **MIME sniffing**, inline SVG/HTML, or
  reflected filenames/metadata.
- `accept`/extension/client-MIME checks are **not controls**; validate by content, server-side.
- **SVG is XML that scripts**: it runs as a top-level document / inline embed (not as `<img>`),
  is a mutation-XSS hotspot, and can surface **XXE** when parsed server-side.
- The markup-trick catalog (case/whitespace, recursive strip, entity decoding, no-`<script>`
  vectors, RCDATA/comment breakouts, foreign content, `<base>` hijack, dangling markup) all
  stems from filter↔browser parsing mismatches.
- Fix with content-based validation, correct `Content-Type` + `nosniff` + `attachment`,
  **separate-origin serving**, vetted allow-list sanitizers, hardened XML parsers, and CSP.

---

[← Previous: Chapter 13 — HTML5 Features and Their Security Impact](13-html5-features.md) | [Next: Chapter 15 — A Testing Methodology for HTML Vulnerabilities →](../part5/15-methodology.md)
