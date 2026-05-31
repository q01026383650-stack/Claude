# Chapter 17 — Hands-On Labs and Exercises

> Reading about XSS teaches you the words; breaking a lab teaches you the language. This chapter
> shows how to stand up safe, **legal** practice environments and gives a structured set of
> exercises mapped to every chapter — plus a tiny self-contained vulnerable page you can run
> locally to see the concepts with your own eyes.

---

## 17.1 The golden rule of practice

**Practice only on systems you own or are explicitly authorized to use.** Everything below is
either software you run on your own machine or a managed range that *grants* you permission. Do
not point these techniques at any other site. (See [Chapter 1](../01-legal-and-ethics.md).)

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
   [Chapter 3 §3.12](../part1/03-browser-parsing.md) and the source/sink drill from
   [Chapter 4 §4.11](../part1/04-the-dom.md). Explain, in your own words, why the DOM differs
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
build in the lab is necessary — the discipline from [Chapter 1](../01-legal-and-ethics.md) is
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

[← Previous: Chapter 16 — Tooling](16-tooling.md) | [Next: Chapter 18 — Secure Output Handling and Sanitization →](../part6/18-defense-remediation.md)
