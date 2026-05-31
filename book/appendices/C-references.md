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
  [Chapter 3](../part1/03-browser-parsing.md). <https://html.spec.whatwg.org/>
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
  [Chapter 15](../part5/15-methodology.md). <https://owasp.org/www-project-web-security-testing-guide/>
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
  ([Chapter 18](../part6/18-defense-remediation.md)). <https://github.com/cure53/DOMPurify>
- Browser **DevTools** (Chromium/Firefox) — the primary instrument
  ([Chapter 16](../part5/16-tooling.md)).

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

Pick based on your goals; the lab-driven practice in [Chapter 17](../part5/17-labs.md) underlies
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

---

[← Previous: Appendix B — Glossary](B-glossary.md) | [Back to the Table of Contents →](../../README.md)
