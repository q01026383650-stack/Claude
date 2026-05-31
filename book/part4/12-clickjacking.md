# Chapter 12 — Clickjacking and UI Redressing

> Clickjacking weaponizes a single HTML element — the `<iframe>` — plus a little CSS. The
> attacker frames a real, sensitive page invisibly over (or under) decoy content, so the
> victim's genuine clicks land on the target's controls. No script injection into the target is
> needed; the target's own UI is the attack surface.

---

## 12.1 The concept

"UI redressing" is the umbrella term; clickjacking is the best-known form. The attacker:

1. Loads the target site inside an `<iframe>` on a page they control.
2. Uses CSS to make the frame **transparent** and **positioned** over enticing decoy content
   (a game, a "claim your prize" button, a video).
3. Aligns the decoy so the victim's clicks/keystrokes actually hit a sensitive control in the
   hidden target frame (a "Delete account," "Confirm transfer," "Grant permission," or
   "Authorize app" button).

The victim believes they're interacting with the attacker's harmless page; in reality they're
operating the real target site **while logged in**, so the actions are authenticated and
genuine. Because the click happens in the real UI, it carries real anti-CSRF tokens — which is
why clickjacking can succeed where CSRF is blocked.

## 12.2 The minimal anatomy

A conceptual clickjacking page (for use only against an in-scope target in your lab):

```html
<style>
  iframe {
    position: absolute; top: 0; left: 0;
    width: 1000px; height: 800px;
    opacity: 0.0;            /* invisible target frame */
    z-index: 2;              /* on top */
  }
  #decoy {
    position: absolute; z-index: 1;   /* underneath */
  }
  #bait {
    position: absolute; z-index: 3;   /* visual lure, positioned under the real button */
  }
</style>

<div id="decoy">Win a free prize! Click the button below 🎁</div>
<button id="bait">CLAIM PRIZE</button>
<iframe src="https://target.example/account/delete"></iframe>
```

The craft is in the **alignment**: positioning the frame so the invisible "Delete account"
button sits exactly under the visible "CLAIM PRIZE" lure. Attackers tune `top`/`left`/scale and
sometimes scroll the frame to bring a specific control into the click zone.

Two opacity strategies:
- **Transparent overlay** (`opacity: 0`): the real frame is on top but invisible; the user sees
  the decoy and clicks "through" to the frame.
- **Decoy on top with a hole / partial opacity**: variations where the bait is layered to direct
  the click precisely.

## 12.3 Variants of UI redressing

- **Classic clickjacking** — hijacking a click on a sensitive button.
- **Likejacking** — historically, tricking users into clicking social "Like"/follow buttons.
- **Cursorjacking** — using a custom CSS cursor to misrepresent where the real pointer is, so the
  user aims wrong.
- **Keystroke/strokejacking** — framing a target field and luring the user to type (e.g., into a
  decoy form) so input lands in the framed page.
- **Content redressing / partial overlays** — covering parts of a legitimate page with attacker
  content to change its apparent meaning (e.g., hiding the real recipient of a payment).
- **Drag-and-drop attacks** — luring the user to drag a token/text from one frame to another to
  exfiltrate or fill fields.
- **Double-clickjacking** — timing a UI change between the two clicks of a double-click so the
  second click lands on a control that appeared mid-gesture (a more recent twist that can evade
  some framing protections by acting within a user-initiated gesture window).

## 12.4 Why it works: framing is allowed by default

By default, the browser will happily render one origin's page inside another origin's
`<iframe>`. The Same-Origin Policy stops the *parent* from reading the *child* frame's contents,
but it does **not** stop the framing itself, and it does **not** stop clicks from reaching the
framed page. Clickjacking lives precisely in this allowed-by-default behavior — the same "SOP
restricts reading, not embedding" asymmetry from [Chapter 4](../part1/04-the-dom.md).

So unless a site *opts out* of being framed, it's potentially clickjackable. That opt-out is the
whole defense (§12.7).

## 12.5 What makes a page a good clickjacking target

When triaging, prioritize pages where:

- A **single click performs a sensitive, irreversible, or valuable action** (delete, confirm,
  authorize, transfer, grant OAuth scope, change a setting).
- The action is **idempotent from one click** (no multi-step confirmation that breaks alignment).
- The page **can be framed** (no `X-Frame-Options` / CSP `frame-ancestors`).
- The user is **likely logged in** when they'd encounter the lure.

OAuth/consent screens, "delete/disable" actions, payment confirmations, and admin toggles are
classic high-value targets.

## 12.6 Detecting clickjacking (procedure)

1. **Check framing headers** on the target response:
   - `X-Frame-Options: DENY` or `SAMEORIGIN` → framing restricted.
   - `Content-Security-Policy: frame-ancestors 'none'` (or a specific allow-list) → modern,
     stronger control (and the one that takes precedence on modern browsers).
   - **Neither present** → likely frameable.
2. **Try to frame it** in a local test page in your lab. If the page renders inside your
   `<iframe>`, it's frameable. (Some pages use *frame-busting JavaScript* instead of headers; see
   §12.8.)
3. **Identify a sensitive single-click control** on the framed page.
4. **Build an alignment PoC** that visually demonstrates the lure overlapping the control. To
   keep it ethical, you generally **do not** need to actually trick a real person — proving the
   page frames *and* that a decoy can be aligned over a sensitive button is sufficient evidence.
   Capture a screenshot showing the overlap (e.g., with the frame at partial opacity so the
   reviewer can see both layers).
5. **Report** with the headers (or their absence), the frameability proof, and the specific
   sensitive action at risk.

> Many proxies include a clickjacking helper that frames a captured page so you can confirm
> frameability and produce a PoC quickly (Chapter 16).

## 12.7 The real defense: tell the browser not to frame you

Two mechanisms (use the CSP one as primary on modern browsers; keep XFO for legacy):

- **`Content-Security-Policy: frame-ancestors 'none';`** — forbids *all* framing.
  `frame-ancestors 'self';` allows only same-origin framing; `frame-ancestors https://trusted.example;`
  allows a specific list. This is the modern, flexible, and recommended control, and it
  supersedes `X-Frame-Options` where both are present.
- **`X-Frame-Options: DENY`** (or `SAMEORIGIN`) — the older header. Still useful for older
  browsers, but it can't express multiple allowed origins cleanly and is effectively legacy.

Apply these to **every** sensitive page (and ideally site-wide as a default), and verify they're
actually emitted on the responses that matter — not just the home page.

## 12.8 Why frame-busting JavaScript is not enough

Before headers existed, sites used "frame busters" — JS that detects framing
(`if (top !== self) top.location = self.location`) and breaks out. These are historically
**bypassable**:

- `sandbox`ed iframes can disable the framed page's scripts (`<iframe sandbox>` without
  `allow-scripts`), so the buster never runs — yet clicks may still register depending on the
  configuration; at minimum, script-based busting is neutralized.
- Various navigation/`onbeforeunload` tricks and timing races defeated older busters.

The lesson: **frame protection must be enforced by the browser via headers/CSP**, not by
JavaScript that the framing page can suppress. If you find a target relying solely on
frame-busting JS, that's a finding — note that it's bypassable and recommend `frame-ancestors`.

## 12.9 Interaction with cookies and CSRF

Clickjacking and CSRF are cousins:

- Both are **cross-site** attacks that don't require breaking SOP's read restriction.
- CSRF forges the *request*; clickjacking induces the *user* to make the request through the
  real UI.
- This is why clickjacking can defeat **anti-CSRF tokens**: the framed real page includes its
  own valid token, and the victim's genuine click submits it.
- `SameSite` cookies interact here too: because clickjacking often involves the user clicking in
  a framed top-... actually a subframe, cookie behavior depends on `SameSite` and whether the
  action is a navigation or subrequest. Test the actual flow rather than assuming.

For sensitive actions, the robust answer combines **`frame-ancestors`** (stop framing) with
**re-authentication or explicit confirmation** that's resistant to single-click hijacking.

## 12.10 Lab exercise

In your lab (use a local app you control, e.g., DVWA which includes a clickjacking-friendly
target, or your own page framing a local sensitive form):

1. Check whether the target page sends `X-Frame-Options`/`frame-ancestors`.
2. Build a local page that frames it; confirm it renders.
3. Overlay a decoy button and align it (visually) over a sensitive control; set the frame to
   `opacity: 0.3` so you can see the alignment, then to `0` to show the finished illusion.
4. Add `Content-Security-Policy: frame-ancestors 'none'` to the target's responses (if you
   control it) and confirm the frame now refuses to load — observe the fix.
5. Write up the finding as you would in a report: header status, frameability proof, the
   at-risk action, and the remediation.

---

## Key takeaways

- Clickjacking frames a real, sensitive page invisibly and lures the victim's genuine clicks
  onto it — abusing the fact that **framing is allowed by default** and SOP doesn't stop clicks
  reaching a framed page.
- Because the click happens in the **real UI**, clickjacking can succeed where CSRF tokens block
  request forgery.
- Detect by checking for `X-Frame-Options` / CSP `frame-ancestors`; if absent, confirm
  frameability in a lab. A benign PoC just needs to **show the overlap**, not trick a real user.
- The real defense is **`Content-Security-Policy: frame-ancestors`** (with `X-Frame-Options` for
  legacy), applied to every sensitive page. **Frame-busting JavaScript is bypassable** and not a
  substitute.

---

[← Previous: Chapter 11 — CSRF and the Role of HTML Forms](11-csrf.md) | [Next: Chapter 13 — HTML5 Features and Their Security Impact →](13-html5-features.md)
