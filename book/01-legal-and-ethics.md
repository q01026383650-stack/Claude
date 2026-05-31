# Chapter 1 — Legal Foundations and Ethics

> **Read this chapter before any other.** Everything that follows assumes you are operating
> with explicit authorization and within a defined scope. The techniques in this book are
> powerful and, used without permission, illegal.

---

## 1.1 The single rule that matters most

**Only test systems you own or have explicit, written authorization to test.**

That is the rule the entire profession rests on. The same HTTP request can be a routine part
of a paid engagement or a criminal act — the only difference is authorization. A skilled
tester is defined less by the payloads they know than by the discipline to stay inside scope.

## 1.2 Why HTML testing feels deceptively harmless (and isn't)

HTML-centric attacks — typing `<script>` into a search box, embedding a site in an `iframe`,
crafting a form that submits cross-site — can feel trivial. There is no "hacking the
mainframe" drama. That low friction is exactly why people get into trouble: it's easy to
"just try" a payload on a live site you don't own.

Don't. Reflecting a payload that pops an alert on someone else's production site is still
unauthorized access in most legal regimes, and a stored payload can harm real users. Treat
HTML attacks with the same seriousness as any other intrusion technique.

## 1.3 The legal landscape (general orientation, not legal advice)

Laws vary by country, and this section is **not legal advice**. Consult a qualified lawyer
for your jurisdiction. That said, most places have computer-misuse statutes that criminalize
unauthorized access to computer systems. Commonly referenced examples include:

- **United States** — the Computer Fraud and Abuse Act (CFAA) and various state laws.
- **United Kingdom** — the Computer Misuse Act.
- **European Union** — national implementations of the Directive on attacks against
  information systems, plus data-protection law (GDPR) when personal data is involved.
- **Many other countries** — analogous "unauthorized access" and "data interference" laws.

The recurring themes across jurisdictions:

1. **Access without authorization is the crime**, independent of harm or intent to profit.
2. **Exceeding authorized access** (testing out of scope) can be treated the same as having
   no authorization at all.
3. **Data protection laws stack on top.** Even authorized testing can create liability if you
   mishandle personal data you encounter.

## 1.4 Authorization in practice

Verbal "go ahead" is not enough. Before testing, make sure you have:

- **A signed contract or engagement letter** with the asset owner.
- **A written scope** listing in-scope domains, IP ranges, applications, and explicitly
  out-of-scope items.
- **Rules of Engagement (RoE):** allowed techniques, time windows, rate limits, data-handling
  rules, and prohibited actions (e.g., no destructive payloads, no social engineering of real
  staff unless agreed).
- **An emergency contact** and an agreed escalation path if you find something critical or
  accidentally cause an outage.
- **A "get out of jail" letter / authorization memo** you can produce if challenged.

If you are doing bug bounty work, the program's **policy page is your scope and RoE.** Read it
in full. "In scope," "out of scope," "safe harbor," and "prohibited testing" sections are
binding. When in doubt, ask the program before testing.

## 1.5 Scope discipline for HTML testing specifically

HTML attacks have a habit of crossing boundaries you didn't intend:

- **Stored XSS reaches other users.** A payload you store in a comment may execute in an
  administrator's browser. That can constitute unauthorized access to *their* session even if
  the app is in scope. Keep stored payloads benign and self-identifying, and clean them up.
- **Cross-site requests reach third parties.** A CSRF or redirect proof-of-concept can fire
  requests at domains outside scope. Make sure your PoC targets only in-scope endpoints.
- **`iframe`/embedding tests can pull in external origins.** Be careful that clickjacking or
  framing demos don't interact with out-of-scope sites.
- **Email/notification triggers.** Injecting markup into a field that later renders in an
  email can spam real people. Confirm such side effects are permitted.

A good habit: **tag every payload** with something identifying, e.g.
`/* pentest-jdoe-2026-05 */` or a unique nonce string, so artifacts are easy to find, attribute,
and remove afterward.

## 1.6 The "non-destructive proof of concept" principle

Your job is to *demonstrate* risk, not to cause damage. Prefer the least invasive PoC that
proves the finding:

- To prove XSS, surface a harmless signal — render `document.domain`, change a heading's text,
  or log to the console — rather than stealing cookies or session tokens. If demonstrating
  impact (e.g., session theft) is required by scope, do it against a test account you control,
  with the client's written agreement, and never against real users.
- To prove CSRF, target a low-impact state change (toggle a non-critical preference) where
  possible.
- To prove clickjacking, show that the page frames and that a click *would* land on a sensitive
  control — you don't need to actually trick a real user.

Document exactly what you did so it can be reproduced and reversed.

## 1.7 Handling data you encounter

During HTML testing you may stumble onto other users' data, internal pages, or secrets.

- **Minimize.** Access only what's necessary to prove the finding.
- **Don't exfiltrate.** Don't pull databases or download personal data "to be thorough."
- **Record carefully.** Screenshots and request/response captures may contain sensitive data;
  store them encrypted and share them only through agreed channels.
- **Report and stop.** If you find evidence of a prior breach or material illegality, follow
  the RoE escalation path immediately.

## 1.8 Responsible disclosure

If you find a vulnerability outside a formal engagement (for example, you stumble on it as a
user), do not test further. Instead:

1. Stop probing.
2. Look for a published security/disclosure policy or `security.txt`.
3. Report privately with enough detail to reproduce.
4. Give the owner reasonable time to fix before any public discussion.
5. Don't demand payment; coordinate, don't coerce.

## 1.9 Building a personal practice lab (the legal way to learn)

You never need someone else's site to practice. Use intentionally vulnerable, self-hosted
targets and public training ranges:

- **OWASP Juice Shop** — modern, deliberately insecure web app.
- **DVWA** (Damn Vulnerable Web Application) — classic teaching app with adjustable difficulty.
- **PortSwigger Web Security Academy** — free, browser-based labs with a legal sandbox.
- **bWAPP, WebGoat, Mutillidae** — additional vulnerable apps.
- **HackTheBox / TryHackMe** — managed legal environments.

[Chapter 17](part5/17-labs.md) walks through standing these up. Every payload in this book is
meant to be run against targets like these or systems you are contracted to test.

## 1.10 A short ethics checklist

Before you send a payload, ask:

- [ ] Am I in scope, right now, for this exact asset?
- [ ] Is this the least invasive way to prove the point?
- [ ] Could this payload affect real users or third parties? If so, have I controlled for that?
- [ ] Is my payload tagged and reversible?
- [ ] If someone audited my traffic, could I justify every request?

If any answer is uncomfortable, stop.

---

## Key takeaways

- Authorization is the line between testing and crime. Get it in writing; honor scope.
- HTML attacks cross boundaries easily — to other users, other origins, and out of scope.
  Plan for that.
- Prefer benign, reversible proofs of concept. Demonstrate risk; don't realize it.
- Practice only on systems you own or legal training ranges.

---

[← Previous: Preface](00-preface.md) | [Next: Chapter 2 — HTML Refresher for Security Testers →](part1/02-html-refresher.md)
