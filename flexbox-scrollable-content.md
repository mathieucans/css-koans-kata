# Flexbox: why you need `min-height: 0` / `min-width: 0` to make an area scroll

## The symptom

You want a full-screen layout: a fixed header and a main area that scrolls.
You put `overflow: auto` on the main area… and **nothing scrolls**. Instead, the whole page overflows.

```html
<body>
  <div class="header">head</div>
  <div class="wrapper">
    <div class="left">left</div>
    <div class="main">
      <!-- lots of content -->
    </div>
  </div>
</body>
```

```css
html {
  height: 100%;
  display: flex;
}

body {
  flex: 1;
  display: flex;
  flex-direction: column;   /* header on top, wrapper below */
}

.wrapper {
  flex: 1;                  /* should take the remaining height... in theory */
  display: flex;            /* left | main side by side */
}

.main {
  flex: 1;
  overflow: auto;           /* does not work! */
}
```

## The cause: the automatic minimum size of flex items

In regular CSS, `min-height` and `min-width` default to `auto`, which in practice means `0`.

**But for a flex item (a direct child of a `display: flex` container), `auto` means something different:**

> By default, the minimum size of a flex item is **the size of its content**.
> A flex item **refuses to shrink below its content**, even with `flex-shrink: 1`.

The specification calls this the *automatic minimum size* (or *content-based minimum size*).

The rule exists to stop content from being squashed or overlapping. It is a sensible default, but it gets in the way as soon as you want a scrollable area.

## What happens, step by step

1. `body` has a fixed height (100% of the viewport) and lays out its children in a **column**.
2. `.wrapper` is a **flex item** of `body` with `flex: 1`, so it should fill the remaining height.
3. But `.wrapper` has `min-height: auto`, so it cannot be **smaller than its content**.
4. Its content is `.main` with all its paragraphs, which is very tall.
5. So `.wrapper` grows to that height and **overflows `body`**.
6. `.main` is stretched to the height of `.wrapper`, so it is **never too small** for its content.
7. Its `overflow: auto` therefore has **no reason to show a scrollbar**.

**The problem is not on the element that should scroll. It is on its flex ancestor, which refuses to shrink.**

## The fix

Allow the intermediate flex item to be smaller than its content:

```css
.wrapper {
  flex: 1;
  display: flex;
  min-height: 0;      /* lets .wrapper shrink below its content size */
}

.main {
  flex: 1;
  overflow-y: auto;   /* the only scrolling area */
}
```

Now:

- `.wrapper` takes exactly the remaining height inside `body`;
- `.main` is limited to that height;
- its content overflows, so `overflow-y: auto` shows a scrollbar.

## `min-height` or `min-width`?

It depends on the **main axis of the parent flex container**:

| Parent container | Main axis | Property to set on the child |
|---|---|---|
| `flex-direction: column` | vertical | `min-height: 0` |
| `flex-direction: row` (default) | horizontal | `min-width: 0` |

### Horizontal example

A flex item in a row contains a wide table, a `<pre>` block, or a long URL with no spaces:

```css
.row {
  display: flex;
}

.content {          /* flex item of .row */
  flex: 1;
  min-width: 0;     /* otherwise .content grows to its content width and pushes the layout */
}

.content pre {
  overflow-x: auto; /* horizontal scrolling can now kick in */
}
```

Without `min-width: 0`, `.content` grows to the width of the `<pre>` and the whole row overflows horizontally.

## Nested flex containers

The rule applies at **every level**. If several nested flex items sit between the fixed-size container and the scrolling element, **each one** must be allowed to shrink along its parent's main axis:

```css
body      { display: flex; flex-direction: column; height: 100vh; }
.level1   { flex: 1; display: flex; flex-direction: column; min-height: 0; }
.level2   { flex: 1; display: flex; flex-direction: column; min-height: 0; }
.scroller { flex: 1; overflow-y: auto; }
```

Forget a single `min-height: 0` in the chain and scrolling stops working.

## Key takeaways

> **When `overflow: auto` does not scroll inside a flex layout, find the flex ancestor that refuses to shrink and give it `min-height: 0` (vertical axis) or `min-width: 0` (horizontal axis).**

- By default, a flex item cannot be smaller than its content (`min-*: auto`).
- An element that is never too small never overflows, so it never scrolls.
- `min-*: 0` removes that constraint explicitly.

## References

### Specification (W3C)

- **CSS Flexible Box Layout Module Level 1, §4.5 Automatic Minimum Size of Flex Items**
  <https://www.w3.org/TR/css-flexbox-1/#min-size-auto>
  Defines the content-based automatic minimum size of flex items.

### MDN documentation

- `min-height`: <https://developer.mozilla.org/en-US/docs/Web/CSS/min-height>
- `min-width`: <https://developer.mozilla.org/en-US/docs/Web/CSS/min-width>
- `overflow`: <https://developer.mozilla.org/en-US/docs/Web/CSS/overflow>

### Articles

- **Josh W. Comeau, *An Interactive Guide to Flexbox*, section "The minimum size gotcha"**
  <https://www.joshwcomeau.com/css/interactive-guide-to-flexbox/#the-minimum-size-gotcha>
  > "The Flexbox algorithm refuses to shrink a child below its minimum size. The content will overflow rather than shrink further, no matter how high we crank flex-shrink!"
- **Ahmad Shadeed, *Defensive CSS*, "Minimum content size in CSS flexbox"**
  <https://defensivecss.dev/tip/flexbox-min-content-size/>

### Stack Overflow

- *Why don't flex items shrink past content size?*
  <https://stackoverflow.com/questions/36247140>
- *Scrolling a flexbox with overflowing content*
  <https://stackoverflow.com/questions/21515042>