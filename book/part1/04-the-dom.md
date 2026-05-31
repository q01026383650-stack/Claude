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
validation/encoding.** Finding these data flows is the heart of [Chapter 10](../part3/10-xss-dom.md);
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
  relevant, not just cosmetic. (See [Chapter 12](../part4/12-clickjacking.md).)
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
  (the basis of **CSRF** — [Chapter 11](../part4/11-csrf.md)). It just can't read the responses.
- A cross-origin page can **embed** your page in an `<iframe>` (the basis of **clickjacking** —
  [Chapter 12](../part4/12-clickjacking.md)) unless you forbid framing.
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

[← Previous: Chapter 3 — How Browsers Parse HTML](03-browser-parsing.md) | [Next: Chapter 5 — HTML Forms and Input Controls →](../part2/05-forms-and-inputs.md)
