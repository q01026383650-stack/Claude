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
only**. Read [Chapter 1: Legal Foundations and Ethics](book/01-legal-and-ethics.md) before
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
- [Preface](book/00-preface.md)
- [Chapter 1 — Legal Foundations and Ethics](book/01-legal-and-ethics.md)

### Part I — Foundations
- [Chapter 2 — HTML Refresher for Security Testers](book/part1/02-html-refresher.md)
- [Chapter 3 — How Browsers Parse HTML](book/part1/03-browser-parsing.md)
- [Chapter 4 — The DOM and the Rendering Model](book/part1/04-the-dom.md)

### Part II — Forms, Inputs, and the Attack Surface
- [Chapter 5 — HTML Forms and Input Controls](book/part2/05-forms-and-inputs.md)
- [Chapter 6 — Mapping the HTML Attack Surface](book/part2/06-attack-surface.md)

### Part III — Core HTML-Based Vulnerabilities
- [Chapter 7 — HTML Injection](book/part3/07-html-injection.md)
- [Chapter 8 — Reflected XSS](book/part3/08-xss-reflected.md)
- [Chapter 9 — Stored XSS](book/part3/09-xss-stored.md)
- [Chapter 10 — DOM-Based XSS](book/part3/10-xss-dom.md)

### Part IV — HTML-Enabled Attacks
- [Chapter 11 — CSRF and the Role of HTML Forms](book/part4/11-csrf.md)
- [Chapter 12 — Clickjacking and UI Redressing](book/part4/12-clickjacking.md)
- [Chapter 13 — HTML5 Features and Their Security Impact](book/part4/13-html5-features.md)
- [Chapter 14 — Dangerous Uploads, SVG, and Markup Tricks](book/part4/14-uploads-svg-markup.md)

### Part V — Methodology, Tooling, and Labs
- [Chapter 15 — A Testing Methodology for HTML Vulnerabilities](book/part5/15-methodology.md)
- [Chapter 16 — Tooling: Browser DevTools, Proxies, and Scanners](book/part5/16-tooling.md)
- [Chapter 17 — Hands-On Labs and Exercises](book/part5/17-labs.md)

### Part VI — Defense and Remediation
- [Chapter 18 — Secure Output Handling and Sanitization](book/part6/18-defense-remediation.md)
- [Chapter 19 — Content Security Policy in Depth](book/part6/19-csp.md)

### Appendices
- [Appendix A — Payload and Encoding Cheat Sheets](book/appendices/A-cheatsheets.md)
- [Appendix B — Glossary](book/appendices/B-glossary.md)
- [Appendix C — References and Further Reading](book/appendices/C-references.md)

---

## A note on responsible use

The payloads and techniques in this book work. Use them only against systems you own or are
explicitly authorized in writing to test. Unauthorized testing is illegal in most
jurisdictions and unethical everywhere. See [Chapter 1](book/01-legal-and-ethics.md).

---

## License

This text is provided for educational purposes. You are free to read, share, and learn from
it. The author and contributors accept no liability for misuse.
