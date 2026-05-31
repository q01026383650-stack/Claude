# Chapter 3 — How Browsers Parse HTML

> Filters and sanitizers fail because their authors imagine a tidier HTML than the one
> browsers actually parse. This chapter gets inside the parser so you can predict what a
> browser will *really* do with your payload — and why so many "clean" inputs still execute.

---

## 3.1 The big idea: HTML parsing is forgiving and that's the problem

HTML is not XML. The HTML standard (maintained by the WHATWG) specifies an extraordinarily
**lenient** parser whose job is to render *something* no matter how broken the markup is.
Missing quotes, missing end tags, mis-nested elements, stray `<` — the parser has a rule for
all of it. Browsers will silently "correct" markup into a valid DOM.

For a tester, this lenience is the gift that keeps giving. Two reasons:

1. **A payload that looks malformed to a human (or a regex) may be repaired by the browser
   into something that executes.** The browser is, in effect, on your side.
2. **Sanitizers that don't parse exactly like a browser will disagree with the browser about
   what the markup means.** Every such disagreement is a potential bypass — this is the root
   cause of an entire class of *mutation XSS* (mXSS) bugs.

## 3.2 The parsing pipeline at a glance

When a browser receives an HTML byte stream it goes through roughly these stages:

```
bytes → [decoding to characters] → [tokenizer] → [tree construction] → DOM
                                       │                │
                                  (states &        (insertion modes &
                                   character          the "stack of
                                   references)        open elements")
```

- **Decoding** turns bytes into characters using the character encoding (the charset). Get the
  charset wrong and everything downstream shifts.
- **Tokenizer** is a *state machine* that reads characters and emits tokens: start tags, end
  tags, text, comments, etc.
- **Tree construction** takes tokens and builds the DOM, applying rules about where elements
  are allowed and auto-correcting when they aren't.

Most "interesting" parser behavior comes from the tokenizer's **states** and the tree
builder's **insertion modes**. You don't need to memorize the spec, but you do need the
intuition.

## 3.3 The tokenizer is a state machine

The tokenizer is always in some *state*, and the current character plus the state decides what
happens next and which state comes after. A few states are worth knowing by name because
payloads live in the transitions between them:

- **Data state** — normal text. A `<` moves toward "tag open."
- **Tag open / tag name states** — after a `<`, the parser decides if this is a start tag, end
  tag, comment, or just stray text.
- **Attribute name / before attribute value / attribute value (double-quoted, single-quoted,
  unquoted) states** — this is where quoting rules come from.
- **RAWTEXT state** (`<style>`, and historically others) — `<` is just text.
- **RCDATA state** (`<textarea>`, `<title>`) — `<` is text, but character references are
  decoded.
- **Script data state** (`<script>`) — special rules (see §3.7).

Two practical consequences:

**(a) A lone `<` is often not a tag.** In the data state, `<` followed by something that isn't
a letter, `/`, `!`, or `?` is treated as literal text. So `a < b` renders fine. Filters that
panic at every `<` over-block; parsers are pickier about what actually opens a tag.

**(b) Whitespace and quoting in attributes are flexible.** The parser accepts spaces, tabs,
newlines, and form feeds as separators, and accepts `=` with surrounding whitespace. This is
why payloads like the following are all equivalent to the browser even though they look odd:

```html
<img src=x onerror=alert(1)>
<img src = x onerror = alert(1)>
<img/src='x'/onerror='alert(1)'>
<img
  src=x
  onerror=alert(1)>
```

The slash as a separator (`<img/src=...>`) surprises people: in the "before attribute name"
state a `/` is consumed and parsing continues, so `/` works as a separator between the tag
name and attributes. Naive filters that split on spaces miss this.

## 3.4 Tree construction: the browser fixes your mess

After tokenizing, the tree builder decides where each node goes. It maintains a *stack of open
elements* and follows *insertion modes* (e.g., "in head," "in body," "in table"). When markup
violates the rules, the builder auto-closes, re-parents, or relocates nodes.

Classic examples a tester should recognize:

- **Implicit tag closing.** `<p>one<p>two` becomes two sibling paragraphs — the first `<p>` is
  auto-closed. Many elements auto-close previous ones.
- **Foster parenting in tables.** Content that isn't allowed inside a `<table>` (like stray
  text or certain elements) gets *moved* out of the table, often to just before it. This
  relocation has produced real mXSS bugs, because a sanitizer parsed the markup in one tree
  shape and the browser produced another.
- **Optional end tags.** `<li>`, `<tr>`, `<td>`, `<option>` and others have optional end tags;
  the parser closes them based on what comes next.
- **Reconstruction of active formatting elements.** Misnested `<b>`, `<i>`, etc. get cloned and
  reopened (the famous "adoption agency algorithm"). The DOM you get is not the markup you
  wrote.

The takeaway: **the DOM is the source of truth, not the bytes.** When you test, look at the
*rendered DOM* (DevTools → Elements), not just "View Source," because the browser may have
transformed your input into something quite different — sometimes into something exploitable.

## 3.5 Mutation XSS (mXSS): when the parser and the sanitizer disagree

This deserves its own section because it's one of the most elegant bypass classes.

A server-side or client-side sanitizer takes attacker HTML, parses it, decides it's safe, and
emits a "clean" string. That string is then assigned to `innerHTML` (or returned in a page)
and **re-parsed by the browser.** If the browser's parsing of the "clean" string differs from
the sanitizer's understanding, the result can be markup the sanitizer never intended to allow.

Drivers of mXSS include:

- **Reserialization quirks.** A sanitizer reads a DOM, serializes it back to a string, and the
  serialization isn't perfectly round-trippable. On re-parse, boundaries shift.
- **Special elements** like `<template>`, `<noscript>`, `<svg>`, `<math>`, `<style>`, and
  tables, whose content models change how nested markup is interpreted. SVG/MathML in
  particular introduce *foreign content* with XML-like rules embedded in HTML — a notorious
  mXSS playground.
- **Entity and encoding edge cases** that decode differently across passes.

You don't need to invent mXSS from scratch as a tester, but you must know it exists, because it
explains why "we use a sanitizer" is not the same as "we're safe," and it tells you to test
sanitizers with *foreign content* and *re-parsing-sensitive* inputs. [Chapter 18](../part6/18-defense-remediation.md)
covers choosing sanitizers that are hardened against this.

## 3.6 Foreign content: SVG and MathML islands

Inside `<svg>` and `<math>`, the parser switches to *foreign content* rules that are closer to
XML. This changes several things at once:

- **Case sensitivity and self-closing** behave differently than in HTML.
- **New element and attribute names** become meaningful (`<svg><script>`, `<svg><a>` with
  `xlink:href`, animation elements, etc.).
- **CDATA-like and namespace constructs** appear.

Because the rules differ inside these islands, payloads that are impossible in plain HTML can
become possible, and sanitizers frequently mishandle the HTML↔foreign-content boundary. We dig
into concrete SVG vectors in [Chapter 14](../part4/14-uploads-svg-markup.md).

## 3.7 Inside `<script>`: it's not as raw as you think

`<script>` is a raw-text element, so `<` inside it is text — `if (a<b)` is fine. The naive
mental model is "a script block ends at `</script>`." Reality is subtler, and the subtleties
have been exploited:

- The parser has special **script-data states** that account for the historical interplay
  between `<script>` content and `<!--` / `-->` and the literal sequence `<script` /
  `</script` appearing inside script text.
- This means a value reflected inside a `<script>` block can sometimes be escaped not only with
  `</script>` but via these script-data edge cases.

The defensive lesson (and the offensive opportunity) is the same as Chapter 2's: **HTML-entity
encoding does nothing inside `<script>`.** The browser does not decode `&lt;` inside a script.
So if user data sits in a script context and the developer "encoded" it as HTML, you likely
have an injection — you just need the right script-context break-out, often the literal
`</script>` to close the block and then start fresh markup.

## 3.8 Character references and where they're decoded

From Chapter 2 you know entities like `&#x3C;`. The parser decides *where* to decode them based
on state:

- **Decoded** in the data state (text) and in **attribute value** states (with minor nuances),
  and in **RCDATA** (`<textarea>`, `<title>`).
- **Not decoded** in RAWTEXT (`<style>`) or script data (`<script>`).

This is why `<a href="javascript&#58;alert(1)">` can work: the `&#58;` (a colon) is decoded
*inside the attribute value*, reconstituting `javascript:` *after* a filter that searched for
the literal substring already passed it. Attribute-context entity decoding is one of the most
reliable filter-bypass primitives.

## 3.9 Charset and encoding attacks

Decoding happens before tokenizing, so controlling or confusing the charset can change
everything:

- **Missing/ambiguous charset.** If the server doesn't declare a charset (header or
  `<meta charset>`), the browser may sniff or default, and an attacker who controls early bytes
  may influence the choice. Historically this enabled **UTF-7 XSS**, where `+ADw-` decodes to
  `<` under UTF-7, sailing past filters that looked for `<`. Modern browsers dropped UTF-7
  auto-detection, but charset confusion remains relevant on legacy stacks.
- **Mismatched declared vs actual encoding.** If bytes are UTF-8 but the page claims another
  charset (or vice versa), multibyte sequences can be reinterpreted, occasionally smuggling
  control characters past filters.
- **BOM and overlong encodings.** Edge cases in how byte-order marks and non-canonical UTF-8
  are handled have produced bypasses on older systems.

As a tester: always note the declared charset, try to influence it where the app reflects a
`charset`-like parameter, and remember that filter bypasses sometimes live below the character
level, in the bytes.

## 3.10 `parseFromString`, `innerHTML`, and the template element

Not all HTML parsing happens during the initial page load. JavaScript parses HTML too:

- **`element.innerHTML = str`** parses `str` with the element as context (and *won't* run
  `<script>` inserted this way — but `<img onerror>` and many other vectors still fire).
- **`DOMParser().parseFromString(str, "text/html")`** parses into a detached document.
- **`<template>`** parses its contents into an inert document fragment.

These "fragment parsing" paths have their own context sensitivity (parsing `<td>` inside a
`<div>` context vs a `<table>` context yields different trees) and are central to DOM-based XSS
(Chapter 10) and to sanitizer behavior. Knowing that the same string parses *differently*
depending on the insertion context is, again, the source of many bypasses.

> **Important nuance, often misunderstood:** assigning a `<script>` tag via `innerHTML` does
> **not** execute it. But this is a thin comfort — `innerHTML` happily runs event-handler
> vectors (`<img src=x onerror=...>`, `<svg onload=...>`, `<iframe srcdoc=...>`), so
> `innerHTML` with untrusted data is still a serious sink.

## 3.11 Putting it together: predicting parser behavior

A practical loop for reasoning about any payload:

1. **Where does it land?** Determine the tokenizer state at the injection point (text?
   attribute value? script data? RAWTEXT? foreign content?).
2. **What ends the current state?** That's your break-out character/sequence (`<`, `"`, `'`,
   `</script>`, `</textarea>`, space/`/` for attributes, etc.).
3. **Are entities decoded here?** If yes, you can encode payload characters to dodge string
   filters.
4. **What will tree construction do?** Will the element be auto-closed, relocated (tables), or
   reinterpreted (foreign content)? Check the *DOM*, not the source.
5. **Does anything re-parse the output?** Sanitizer → `innerHTML` round trips invite mXSS.

If you can answer these five questions, you can usually explain why a payload worked, why one
was blocked, and what to try next — which is the entire game.

## 3.12 A short, safe demonstration to run in your lab

In a local lab page (see [Chapter 17](../part5/17-labs.md)), open DevTools and run in the
console:

```js
// Watch the browser repair markup. The DOM is NOT what you typed.
const d = document.createElement('div');
d.innerHTML = '<p>one<p>two<b>bold<i>both</b>italic';
console.log(d.innerHTML);
// Observe auto-closed <p> tags and re-opened <i> (adoption agency).

// Foreign content / fragment context differences:
const t = document.createElement('table');
t.innerHTML = '<div>hi</div>';   // where did the <div> go?
console.log(t.outerHTML);
```

Seeing the browser rewrite your markup, live, is the fastest way to internalize this chapter.
Do this only in your own lab.

---

## Key takeaways

- HTML parsing is intentionally forgiving; browsers repair broken markup into a DOM that may
  differ from what you wrote — sometimes into something exploitable.
- The tokenizer is a state machine; **knowing the current state tells you the break-out
  character.** Attributes accept flexible separators (spaces, tabs, newlines, `/`).
- Entities are decoded in text/attribute/RCDATA contexts but **not** in `<script>`/`<style>`.
  Attribute-context decoding is a reliable filter bypass.
- **Mutation XSS** arises when a sanitizer and the browser disagree about parsing; foreign
  content (SVG/MathML), templates, and tables are hotspots.
- Always inspect the **rendered DOM**, not just View Source. Re-parsing paths (`innerHTML`,
  `DOMParser`, `<template>`) have their own context rules.

---

[← Previous: Chapter 2 — HTML Refresher for Security Testers](02-html-refresher.md) | [Next: Chapter 4 — The DOM and the Rendering Model →](04-the-dom.md)
