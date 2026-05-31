# Chapter 19 — Content Security Policy in Depth

> Content Security Policy is the browser-enforced safety net for when output encoding fails.
> It can't fix a bug, but it can stop a bug from becoming a breach. Testers need to read CSP to
> judge a finding's real impact; defenders need to write CSP that actually mitigates rather than
> merely existing. This chapter does both.

---

## 19.1 What CSP is and what it is not

CSP is an HTTP response header (`Content-Security-Policy`) — or a `<meta http-equiv>` equivalent —
that tells the browser which sources of content are allowed and which behaviors are forbidden.
Its headline benefit for this book: a well-built CSP can **prevent injected script from
executing** even when an XSS injection point exists.

What CSP **is not**:
- It is **not** a substitute for output encoding (Chapter 18). It's *defense in depth*.
- It does **not** fix the underlying injection; it limits the consequences.
- A weak/misconfigured CSP can provide **false comfort** — and is itself a finding.

## 19.2 Syntax and the directives that matter

A CSP is a series of directives separated by semicolons; each names a directive and its allowed
sources:

```
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.example; object-src 'none'; base-uri 'none'; frame-ancestors 'none'
```

Directives most relevant to HTML attacks:

| Directive | Controls | Why it matters here |
|---|---|---|
| `default-src` | fallback for most fetch directives | baseline; doesn't cover everything (e.g., not `frame-ancestors`/`base-uri`) |
| `script-src` | where scripts may load/execute | the heart of XSS mitigation |
| `object-src` | `<object>/<embed>/<applet>` | set to `'none'` to kill plugin/legacy vectors |
| `base-uri` | allowed `<base href>` | set to `'none'`/`'self'` to stop `<base>` hijack (Chapter 5/7) |
| `frame-ancestors` | who may frame this page | anti-clickjacking (Chapter 12) |
| `frame-src`/`child-src` | what this page may frame | limits malicious embedding |
| `img-src`,`connect-src`,`style-src`,`font-src` | other resource types | constrain exfiltration (e.g., dangling-markup `<img>`, beacons) |
| `form-action` | where forms may submit | stop `formaction`/hijacked submissions exfiltrating |
| `require-trusted-types-for` / `trusted-types` | DOM sink enforcement | structurally blocks DOM XSS (Chapter 10) |

Source expressions you'll see: `'self'`, `'none'`, specific origins, `'unsafe-inline'`,
`'unsafe-eval'`, `'nonce-...'`, `'sha256-...'`, `'strict-dynamic'`.

## 19.3 The two big weaknesses: `'unsafe-inline'` and `'unsafe-eval'`

These two source keywords are where most CSPs fail to mitigate XSS:

- **`'unsafe-inline'`** allows inline `<script>` and inline event handlers (`onclick=...`) to run.
  A `script-src` that includes `'unsafe-inline'` provides **essentially no XSS protection**,
  because the classic injected `<script>`/`onerror`/`onload` payloads are exactly inline scripts.
  As a tester, if you see `'unsafe-inline'` in `script-src`, note that the CSP does **not**
  mitigate reflected/stored XSS.
- **`'unsafe-eval'`** allows `eval`, `Function`, and string `setTimeout`. Its presence keeps
  `eval`-based DOM XSS (Chapter 10) viable. Absence neutralizes those sinks.

So the *presence* of an XSS bug under a CSP with `'unsafe-inline'` is still a high-severity
finding; under a strict CSP (no `'unsafe-inline'`), the same injection may be **non-executing**,
which legitimately lowers severity — but report both the bug *and* the CSP weakness.

## 19.4 Doing it right: nonces, hashes, and `strict-dynamic`

A CSP that actually stops XSS avoids `'unsafe-inline'` and instead authorizes *specific* scripts:

- **Nonce-based:** the server emits a random per-response nonce and tags its legitimate scripts:
  ```
  Content-Security-Policy: script-src 'nonce-r4nd0m' 'strict-dynamic'; object-src 'none'; base-uri 'none'
  ```
  ```html
  <script nonce="r4nd0m"> /* trusted inline script */ </script>
  ```
  An attacker who injects a `<script>` can't guess the nonce, so their script won't run. The
  nonce **must be unpredictable and unique per response** — a static or reused nonce is broken
  (and a finding).

- **Hash-based:** instead of a nonce, allow specific inline scripts by their hash
  (`'sha256-...'`). Good for static inline scripts.

- **`'strict-dynamic'`:** lets a nonce/hash-trusted script load further scripts it creates, while
  ignoring host allow-lists. This makes nonce-based policies practical for apps that load scripts
  dynamically, and it sidesteps the well-known weakness of **host allow-list bypasses** (§19.5).

The modern recommendation (echoing Google's CSP guidance) is a **nonce + `'strict-dynamic'`**
policy with `object-src 'none'` and `base-uri 'none'`, rather than long host allow-lists.

## 19.5 Why host allow-lists are weak

Older CSPs allow-list domains: `script-src 'self' https://cdn.example ...`. These are commonly
bypassable because allow-listed origins often host dangerous content:

- **JSONP endpoints** on an allow-listed domain can be abused to execute attacker-chosen
  callbacks.
- **Open redirects** on an allow-listed origin can shuttle to attacker content.
- **Outdated libraries / Angular-style template gadgets** hosted on the allow-listed CDN can be
  leveraged.
- Large CDNs host *so much* that "allow this CDN" effectively allows a lot.

As a tester, when you see a host allow-list CSP, check the allow-listed origins for JSONP, open
redirects, and known gadgets — a CSP "bypass" is often just abusing what's already trusted. As a
defender, prefer nonces + `'strict-dynamic'` over host lists.

## 19.6 Beyond script: locking down the rest

A robust policy doesn't stop at `script-src`:

- **`object-src 'none'`** — removes `<object>/<embed>` vectors; recommended almost universally.
- **`base-uri 'none'` (or `'self'`)** — prevents injected `<base>` from hijacking relative URLs
  and (in some setups) defeating nonce strategies.
- **`frame-ancestors 'none'|'self'|<list>`** — clickjacking defense (Chapter 12).
- **`form-action 'self'`** — stops injected/overridden form actions from exfiltrating to
  attacker servers.
- **`connect-src`/`img-src`** — constrain where data can be sent, blunting exfiltration (including
  dangling-markup `<img>` and beacons, Chapter 7).
- **`require-trusted-types-for 'script'; trusted-types <policies>`** — DOM-XSS structural defense
  (§19.8).

## 19.7 Report-only mode and deployment

`Content-Security-Policy-Report-Only` enforces nothing but **reports violations** to a configured
endpoint (`report-to`/`report-uri`). It's the safe way to roll out a policy: deploy report-only,
watch what *would* break, tune, then switch to enforcing. For testers: a **report-only** header
provides **no protection** — if that's all that's present, the app is effectively unprotected
despite "having a CSP." Note that distinction explicitly.

## 19.8 Trusted Types via CSP (DOM-XSS structural defense)

The CSP directives:

```
Content-Security-Policy: require-trusted-types-for 'script'; trusted-types default dompurify
```

- **`require-trusted-types-for 'script'`** makes dangerous DOM sinks (`innerHTML`, `outerHTML`,
  `script.src`, `document.write`, etc.) throw when assigned a plain string.
- **`trusted-types`** names the allowed policy factories; code must route HTML through a defined
  policy (commonly one wrapping DOMPurify) that returns a `TrustedHTML` object.

The effect (Chapter 10): the diffuse, hard-to-audit DOM-XSS sink surface collapses into a few
central, reviewable policies the browser *enforces*. This is the most effective DOM-XSS mitigation
available. As a tester, its presence sharply reduces DOM-XSS exploitability; its absence on a
high-risk app is a defense-in-depth recommendation.

## 19.9 Reading a CSP as a tester (procedure)

When you find a CSP, evaluate it:

1. **Is it enforcing or report-only?** Report-only = no protection.
2. **`script-src`:** Does it include `'unsafe-inline'`? (→ no XSS mitigation.) `'unsafe-eval'`?
   (→ `eval` DOM XSS viable.) Nonce/hash present and **unpredictable/unique**? `'strict-dynamic'`?
3. **Host allow-list?** Inspect allow-listed origins for JSONP/open-redirect/gadget bypasses.
4. **`object-src`/`base-uri`/`frame-ancestors`/`form-action`** set restrictively?
5. **Trusted Types** required?
6. **Coverage:** is the header on *all* responses (including the ones with your injection), or
   just the home page?

Then state the impact precisely: e.g., "Stored XSS executes despite CSP because `script-src`
includes `'unsafe-inline'`," or "Reflected injection is present but non-executing under the
nonce-based CSP; however, `base-uri` is unset, enabling `<base>` hijack."

## 19.10 A solid baseline policy

A strong, modern starting point (tune to the app):

```
Content-Security-Policy:
  default-src 'self';
  script-src 'nonce-{RANDOM}' 'strict-dynamic';
  object-src 'none';
  base-uri 'none';
  frame-ancestors 'none';
  form-action 'self';
  require-trusted-types-for 'script';
  trusted-types default;
  report-to csp-endpoint
```

Notes:
- Replace `{RANDOM}` with a fresh, unpredictable per-response nonce; tag all first-party scripts.
- Add specific `img-src`/`connect-src`/`style-src` as the app requires, keeping them tight.
- Roll out via **report-only first**, then enforce.
- This single header mitigates the bulk of XSS execution, `<base>` hijack, clickjacking, and
  plugin vectors at once — which is why CSP is the capstone defense.

## 19.11 The limits of CSP (keep expectations honest)

- CSP doesn't stop **CSRF** (different mechanism — `form-action` helps with exfiltration, not the
  forged request itself).
- CSP doesn't stop **non-script HTML injection** outcomes like content spoofing/phishing text
  (though it constrains exfiltration channels).
- CSP can be **bypassed** when misconfigured (`'unsafe-inline'`, weak host lists, dangling-markup
  within allowed `img-src`, etc.).
- CSP is **defense in depth** — the bug still needs fixing at the source (Chapter 18).

State these limits when recommending CSP so it isn't treated as a silver bullet.

## 19.12 Lab exercise

Using your `lab.html` (Chapter 17) or a local app:

1. Add `script-src 'unsafe-inline'` and confirm your XSS still fires — observe that this CSP
   doesn't mitigate.
2. Switch to a **nonce-based** `script-src` (no `'unsafe-inline'`), tag the legitimate script with
   the nonce, and confirm injected inline scripts/handlers no longer execute.
3. Add `base-uri 'none'` and `object-src 'none'`; test a `<base>` hijack and an `<object>` vector
   being blocked.
4. Add `require-trusted-types-for 'script'` and watch `innerHTML` assignments throw; then route
   HTML through a DOMPurify-backed Trusted Types policy and see it work safely.
5. Try a **host allow-list** CSP and, conceptually, identify how a JSONP/open-redirect on an
   allowed origin would bypass it — reinforcing why nonces + `strict-dynamic` are preferred.

---

## Key takeaways

- CSP is **browser-enforced defense in depth**: it can make an XSS injection **non-executing**,
  but it doesn't fix the bug — encode at the source (Chapter 18).
- **`'unsafe-inline'`** (and `'unsafe-eval'`) gut CSP's XSS protection; their presence means the
  bug is still exploitable and is itself a finding.
- Strong policies use **nonces + `'strict-dynamic'`**, `object-src 'none'`, `base-uri 'none'`,
  `frame-ancestors`, and `form-action` — not fragile **host allow-lists** (JSONP/open-redirect/
  gadget bypasses).
- **`require-trusted-types-for 'script'`** is the strongest **DOM-XSS** structural mitigation.
- **Report-only** mode protects nothing; check enforcement and **coverage on all responses**.
  CSP doesn't stop CSRF or content-spoofing text.

---

[← Previous: Chapter 18 — Secure Output Handling and Sanitization](18-defense-remediation.md) | [Next: Appendix A — Payload and Encoding Cheat Sheets →](../appendices/A-cheatsheets.md)
