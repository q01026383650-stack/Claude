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

[← Previous: Chapter 15 — A Testing Methodology for HTML Vulnerabilities](15-methodology.md) | [Next: Chapter 17 — Hands-On Labs and Exercises →](17-labs.md)
