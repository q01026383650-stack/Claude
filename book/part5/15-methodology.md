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

[← Previous: Chapter 14 — Dangerous Uploads, SVG, and Markup Tricks](../part4/14-uploads-svg-markup.md) | [Next: Chapter 16 — Tooling →](16-tooling.md)
