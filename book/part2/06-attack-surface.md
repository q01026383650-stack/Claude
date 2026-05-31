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

[← Previous: Chapter 5 — HTML Forms and Input Controls](05-forms-and-inputs.md) | [Next: Chapter 7 — HTML Injection →](../part3/07-html-injection.md)
