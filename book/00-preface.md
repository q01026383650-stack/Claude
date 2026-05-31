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
  [Chapter 17](part5/17-labs.md)).

## Acknowledgements and standing on shoulders

The web security community is unusually generous with knowledge. This book leans on the
collective work of OWASP, the PortSwigger Web Security Academy, the WHATWG (which maintains
the HTML standard), browser security teams, and countless researchers who have documented
parser quirks and bypasses over the years. Specific resources are collected in
[Appendix C](appendices/C-references.md).

Now, before any technique: read the next chapter.

---

[Next: Chapter 1 — Legal Foundations and Ethics →](01-legal-and-ethics.md)
