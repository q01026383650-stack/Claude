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

[← Previous: Appendix A — Payload and Encoding Cheat Sheets](A-cheatsheets.md) | [Next: Appendix C — References and Further Reading →](C-references.md)
