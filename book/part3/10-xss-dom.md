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
covered in [Chapter 16](../part5/16-tooling.md).

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
structural defense against DOM XSS and is covered in [Chapter 19](../part6/19-csp.md). As a
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

[← Previous: Chapter 9 — Stored XSS](09-xss-stored.md) | [Next: Chapter 11 — CSRF and the Role of HTML Forms →](../part4/11-csrf.md)
