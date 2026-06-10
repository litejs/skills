---
name: litejs-ui
description: Use when building UI with LiteJS framework — writing .ui templates, creating views and routes, using bindings, handling events, i18n, El API, or editing @litejs/ui source code
---

# LiteJS UI Engine

Dependency-free ES5 UI engine (~25kB). Templates, routing, data binding, i18n, touch gestures — no transpiling/bundling.

Although written in ES5, LiteJS works seamlessly in ESM projects — just include the scripts and start building.

## Template Syntax (.ui files)

Indentation-based hierarchy using CSS selectors. Indent = child, dedent = sibling.

```
h1 My list
ul.green.star
    li
        a[href="#a"] Item A
    li > a[href="#b"] Item B
footer
    button:disabled My button
```

Becomes `<h1>My list</h1><ul class="green star"><li><a href="#a">Item A</a></li>...`.

**Selectors:** `tag#id.class1.class2[attr=value]`. Omitting tag defaults to `div`.
**Child combinator:** `li > a[href="#b"] Item B` — inline child.
**Text content:** bare text after selector becomes element text via i18n: `_(text)`.
**Raw text:** `= text content` — direct text node insertion, no i18n.
**Comments:** `/ comment text` — lines starting with `/`.

## Plugins (% prefix)

| Plugin | Usage | Purpose |
|--------|-------|---------|
| `%view name [parent]` | `%view home #public` | Define routed view |
| `%el name` | `%el Dialog` | Define reusable custom element |
| `%slot [name]` | `%slot footer` | Content placeholder in custom elements |
| `%def routes files` | `%def users/{id} users.js,%.css` | Define route with file dependencies |
| `%css` | `%css` + indented CSS block | Inject inline CSS |
| `%js` | `%js` + indented JS block | Inline JavaScript handlers |
| `%each items` | `%each ["a","b"]` | Replicate template block |
| `%svg name` | `%svg icon` | Define SVG custom element (alias for `%el`) |
| `%start` | `%start` | Trigger app initialization (start routing) |

**View names:** A leading `#` (e.g. `#public`, `#login`) makes a view a **non-routable structural container** — used as a parent / `%slot` host, or shown programmatically (`app.show("#login")` / `$ui("#login").show()`), but **not reachable via an `href`**. Routable views have **no** leading `#`; `href="#home"` navigates to the route `home` (the `#` is the URL fragment marker, not part of the view name — so `href="#login"` does *not* reach a view named `#login`).
**Custom elements:** After `%el Dialog`, use `Dialog` as a selector in templates.

### Loading Templates

Templates are loaded from `<script type="ui">` tags in HTML:

```html
<script type="ui">
%view home #
  h1 Hello
%start
</script>
<script src="https://litejs.com/litejs.full.min.js"></script>
```

External .ui files: `<script type="ui" src="views.ui"></script>`

## Bindings (; prefix)

Bindings connect data to DOM. Suffix `!` = execute once, don't update.

| Binding | Syntax | Effect |
|---------|--------|--------|
| `;txt` | `;txt expression` | Set text content |
| `;cls` | `;cls "active", condition` | Add/remove CSS class |
| `;css` | `;css "color", value` | Set inline style |
| `;set` | `;set "data-id", id` | Set attribute |
| `;val` | `;val formField` | Two-way form value binding |
| `;if` | `;if condition` | Conditional render (removes/restores element) |
| `;each` | `;each! "item", array` | List iteration, creates subscope per item |
| `;el` | `;el tagName` | Dynamic element type |
| `;ref` | `;ref myRef` | Store element reference in scope |
| `;name` | `;name fieldName` | Set form element name |
| `;view` | `;view url` | Set href for view navigation |
| `;on` | `;on "event", handler` | Attach event listener |
| `;one` | `;one "event", handler` | One-time event listener |
| `;is` | `;is value, "a,10=b,20=c"` | Threshold-based class switching |
| `;d` | `;d text` | Render block-level document markup |
| `;t` | `;t text` | Render inline markup |
| `;xlink` | `;xlink "#route"` | SVG namespace href (`xlink:href`) |
| `$s` | `;$s` | Initialize scope with element attributes |

**Once modifier:** `;txt! value` — bind at render, never re-evaluate.
**Default binding:** Bare text `h1 Hello` becomes `;txt _("Hello",$s)` — auto i18n lookup.
**Unknown bindings** fall through to `;set` (attribute setter). E.g., `;href! url` sets the `href` attribute.
**Separator:** `:` works the same as space: `;txt:value` equals `;txt value`.
**View-level bindings:** Inside `%view`, bindings set view properties. `;f "file.js"` sets file dependencies loaded on navigation.

## Event Shorthand (@ prefix)

```
@click handler              →  ;on! "click", handler
@click! handler             →  ;one! "click", handler
@keyup "navigate", param    →  ;on! "keyup", "navigate", param
```

Event handlers can be strings (emitted on View) or function references.

**Important:** Outside of `%view` blocks, `@click handler` resolves `handler` from the scope chain. Since the scope is `$d` (global scope), the handler function must be on `$d`:

```javascript
%js
    // WRONG — function declaration not accessible from template scope
    function doSomething() { ... }

    // RIGHT — attach to $d so template bindings can find it
    $d.doSomething = function() { ... }
```

## `;val` Binding Details

`;val` is designed for **form-level** two-way binding. Put `;val data` on a `form` element and use `name` attributes on inputs:

```
form
    ;val data
    input[name=email][type=email]
    textarea[name=notes][rows=4]
```

This syncs form values into `$d.data = {email: "...", notes: "..."}`.

**Gotcha:** Using `;val field` on a standalone input/textarea (outside a `form` with `;val`) does **not** reliably sync user input back to the scope variable. For standalone elements, either wrap in a form or read the DOM value directly:

```javascript
var el = document.getElementById("myInput")
var value = el.value
```

## Views and Routes

### Initialization

```javascript
var app = LiteJS()
```

Options only needed when changing defaults:

| Option | Default | Effect |
|--------|---------|--------|
| `home` | `"home"` | Default view name |
| `root` | `document.body` | Root element for views |
| `breakpoints` | — | Responsive breakpoints, e.g. `"sm,601=md,1025=lg"` |
| `locales` | — | Locale definitions, e.g. `{en: "en"}` |
| `globals` | — | Default translations |

Returns View constructor. Available as `$ui` in templates, use chosen variable name (`app`) in plain JavaScript.

### Defining Views

In .ui templates:
```
%view #public #
.app
    nav
        a[href="#home"] Home
    %slot

%view home #public
p Welcome

%view user/{userId} #public
p User {params.userId}
```

`#public` is a structural container (no route, hosts `%slot`); `home` and
`user/{userId}` are routable. `a[href="#home"]` navigates to the route `home`.
A `#`-prefixed view has no href route — reach it with `app.show("#name")` /
`$ui("#name").show()`, or nest routable children under it.

In JavaScript:
```javascript
app.def("route file.js,file.css\nuser/{id} user.js")
app.show("home")
app.get(url, params)
app.param(["user"], function(value, name, view, params) { /* resolve */ })
```

### View Lifecycle

Navigation: `app.show(url)` → route match → `ping` (each view, async-friendly with `this.wait()`) → render → `open` → `show`.

| Event | When |
|-------|------|
| `ping` | Before render, fetch data here. Call `this.wait()` for async. |
| `pong` | After render |
| `open` | View becomes active |
| `close` | View deactivated |
| `nav` | Navigation started |
| `show` | Navigation complete |
| `resize` | Viewport resized |

### Scope Variables

| Variable | Contains |
|----------|----------|
| `$s` | Current scope |
| `$el` | Current element |
| `$ui` | View router (in templates; use chosen var name in plain JS) |
| `$d` | Global scope |
| `$up` | Parent scope |
| `$i` | Loop index (inside `;each`) |
| `$len` | Loop length (inside `;each`) |
| `_()` | i18n format function |
| `params` | URL parameters |

## El API

`El(selector)` — Create DOM element from CSS selector string.

```javascript
El("div#id.class1.class2[data-x=1]")  // Create element
```

### Static Methods

| Method | Purpose |
|--------|---------|
| `El.append(parent, child)` | Append, handles slots |
| `El.render(el)` | Process bindings on element tree |
| `El.scope(el, parent)` | Get/create scope for element |
| `El.cls(el, name, add, sel, delay)` | Add/remove/toggle class (transfer mode: `add` as DOM element removes class from that element and adds to `el`; auto-revert after `delay` ms) |
| `El.flip(el, sel, fn, opts)` | FLIP animation (snapshot, mutate, animate) |
| `El.get(el, attr)` | Get attribute |
| `El.val(el, val)` | Get/set form value (handles nested forms, selects, checkboxes) |
| `El.kill(el, transition)` | Remove element (with optional CSS transition) |
| `El.empty(el)` | Remove all children |
| `El.replace(old, new)` | Replace element |
| `El.closest(el, sel)` | Find closest ancestor matching selector |
| `El.matches(el, sel)` | Test if element matches selector |
| `El.nearest(el, sel)` | Find nearest element |
| `El.rate(fn, ms)` | Throttle function calls |
| `El.stop(event)` | Stop event propagation |
| `El.addKb(map, killEl)` | Push keyboard shortcut map |
| `El.rmKb(map)` | Remove keyboard shortcut map |
| `El.asEmitter(obj)` | Add `.on`/`.off`/`.one`/`.emit` to object |
| `El.$b` | Bindings object |
| `El.$d` | Global scope |
| `El.T` | Text attribute name (`textContent` or `innerText`) |

## Emitter Pattern

```javascript
El.asEmitter(MyProto.prototype)
var obj = new MyProto()
obj.on("event", handler, scope)     // Listen
obj.one("event", handler, scope)    // Listen once
obj.off("event", handler, scope)    // Remove
obj.emit("event", data)             // Returns listener count
```

## i18n Format Functions

`_("key")` — translate. `_{format, data}` — format with substitution.

| Format | Symbol | Example |
|--------|--------|---------|
| `date` | `@` | `_{@D, timestamp}` |
| `num` | `#` | `_{#.01, 3.14159}` |
| `plural` | `*` | `_{*item, count}` |
| `pick` | `?` | `_{?gender, value}` |
| `pattern` | `~` | `_{~phone, "+1234567"}` |
| `map` | `map` | `_{map items, template}` |
| `lo` | `lo` | lowercase |
| `up` | `up` | UPPERCASE |

## Keyboard Shortcuts

```javascript
El.addKb({
    h: goHome,
    "shift+s": goSettings,
    "mod+h": goHome,     // cmd on Mac, ctrl on PC
    bubble: true,         // bubble to previous map
    input: true           // capture in inputs
})
El.rmKb(map)
```

In templates (preferred — automatically added/removed when the view opens/closes).
String values navigate to the view, function values are called directly:

```
%view home #public
    @kb {h: "home", u: "users", "ctrl+s": save}
```

`El.addKb`/`El.rmKb` can be used directly in JavaScript but requires manual cleanup. The template version is preferred as it binds to the view lifecycle.

## Touch/Pointer Events

Built-in gesture recognition via event system:

| Event | Fires when |
|-------|-----------|
| `pan` | Single-pointer drag |
| `pinch` | Two-pointer distance change |
| `rotate` | Two-pointer angle change |
| `tap` | Quick touch |
| `hold` | Long press |

**Event properties on pan/hold events:**

| Property | Contains |
|----------|----------|
| `e.el` | The element that initiated the gesture |
| `e.mode` | Gesture mode (`"pan"` or `"hold"`) — set on `panstart`/`holdstart` event object |
| `e.x0`, `e.y0` | Pointer start position (clientX/Y at pointerdown) |
| `e.dx`, `e.dy` | Delta from start |
| `e.ex`, `e.ey` | Current position offset (dx + element's CSS left/top) |

## El.flip — FLIP Animations

`El.flip(el, sel, fn, opts)` — Animate DOM mutations using the FLIP technique.

```javascript
// Snapshot positions, mutate DOM, animate the difference
El.flip(container, ".item", function() {
    container.appendChild(someItem)
})

// Snapshot only (no animation) — returns position array
var positions = El.flip(container, ".item")
```

**Options:**

| Option | Default | Effect |
|--------|---------|--------|
| `anim` | `"anim"` | CSS class for transition |
| `enter` | `"op0"` | CSS class for entering elements |
| `leave` | `"op0"` | CSS class/style for leaving elements |
| `skip` | — | Element to exclude from move animation |

**Returns:** Flat array of `[el, rect, el, rect, ...]` pairs.

## CSS Framework

Modular CSS in `css/` directory. Key files:

| File | Utilities |
|------|-----------|
| `grid.css` | `.grid`, `.row`, `.col` — CSS Grid |
| `grid-1of.css` | `.w1`-`.w12` — fraction widths |
| `spacing-4.css` | `.p1`-`.p4`, `.m1`-`.m4` — padding/margin |
| `form.css` | `.btn`, `.field`, `.group`, `.input` |
| `anim.css` | `.anim` — transition classes |
| `accessible.css` | Focus, skip links |
| `prefers.css` | `@media prefers-color-scheme` |

**Responsive prefixes:** `sm-`, `md-`, `lg-` (e.g., `.md-w6`).
**Orientation:** `.port`, `.land`.
**Inject dynamically:** `xhr.css(text)` or `%css` in templates.

## Custom Bindings

Register custom bindings on `El.$b`:

```javascript
El.$b.sticky = function(el, opts) {
    // `this` is the scope, `el` is the DOM element
    new IntersectionObserver(function(entries) {
        entries.forEach(function(entry) {
            El.cls(entry.target, "is-stuck", entry.intersectionRatio < 1)
        })
    }, { threshold: 1 }).observe(el)
}
```

Use in templates: `;sticky!` or `;sticky! options`.

## Asset Loading

```javascript
xhr(method, url, callback)          // Make HTTP request
xhr.load(["file.js", "file.css"], callback)  // Load files once (cached, won't reload)
xhr.css(cssText)                    // Inject stylesheet
xhr.ui(templateSource)              // Register .ui template source
```

**JSON handler pattern** — define `xhr.json` to handle loaded .json files:

```javascript
xhr.json = function(str, url) {
    $d[url.split(".")[0]] = JSON.parse(str)
}
```

## Built-in Elements (el/ directory)

| Element | File | Purpose |
|---------|------|---------|
| `Dialog` | `el/dialog.ui` | Modal dialog with actions, numpad, keyboard |
| `Modal` | `el/dialog.ui` | Overlay container with blur effect |
| `Form-*` | `el/form.ui` | Form field elements (text, boolean, enum, list, array) |
| `Slider` | `el/Slider.ui` | Range slider |
| `Sortable` | `el/sortable.ui` | Drag-to-reorder list using El.flip |
| `Pie` | `el/Pie.tpl` | SVG pie chart |
| `Segment7` | `el/Segment7.tpl` | 7-segment display |

**Modal usage:** `$ui.emit("modal", title, opts, callback)`
- `opts.actions`: Array of `{action, title, key}` or strings
- `opts.code`: Numpad PIN length
- `opts.vibrate`, `opts.sound`: Notification feedback

## Project Structure

```
ui.js           Core engine (templates, bindings, views, events, i18n)
shim.js         ES5 polyfills for IE5.5+ (~8kB)
load.js         XHR asset loader, theme detection
require.js      CommonJS-like module loader
extract-lang.js Language extraction utility
el/             Reusable UI elements (.ui, .tpl)
binding/        Extra bindings (persist.js, svg.js)
css/            Modular stylesheets
bin/            CLI tools (lj-extract-lang)
```

## Markup Syntax (`;t` and `;d` bindings)

Inline markup (`_;t`) and block markup (`;d`) use bracket-based syntax:

**Inline formatting:** `[op content op]` — e.g., `[*bold*]`, `[/italic/]`, `[-deleted-]`, `[_underline_]`, `[`code`]`, `[^super^]`, `[,sub,]`, `[:mark:]`, `[;span;]`, `[~strike~]`, `[+inserted+]`

**Links:** `[text <url>]` or `[<url>]` — e.g., `[Click here <https://example.com>]`

**Images:** `[alt !src!]` or `[!src!]` — e.g., `[sunset !photo.jpg!.right]`

**Embeds:** `[!protocol:id!]` — e.g., `[!youtube:dQw4w9!]`. Register handlers via `El.$b.d.embed`:
```javascript
El.$b.d.embed.youtube = function(id, text, extra) {
    return "<iframe src=\"https://youtube.com/embed/" + id + "\"" + extra + "></iframe>"
}
```

**Abbreviations:** `[Full Name ?ABBR?]` defines, `[?ABBR?]` reuses — e.g., `[HyperText Markup Language ?HTML?]`

**Time:** `[@2024-07-12@]` or `[label @2024-07-12@]`

**CSS classes:** Append `.class` — e.g., `[*bold*.red]`, `[<url>.tip]`

**Block markup (`;d` only):** Headings `= text` / `== text`, lists ` - item`, ordered lists ` 1. item`, blockquotes `> text`, `---` for `<hr>`.

## Common Patterns

**Conditional rendering:**
```
.panel
    ;if user.loggedIn
    p Welcome {user.name}
```

**List rendering:**
```
ul
    li
        ;each! "item", items
        span {item.name}
```

**Form with two-way binding:**
```
form
    ;val data
    input[name=email][type=email]
    input[name=password][type=password]
    button[type=submit] Submit
        @click handleSubmit
```

**Custom element with slots:**
```
%el card
.card
    h3 > %slot title
    .card-body > %slot

%view home #public
card
    span[slot=title] My Card
    p Card body content
```

**View with async data loading (from test/html/routed.html):**
```
%view blog/{blogFile} #pub
main
    article ;d!blogFile

%js
    var blogCache = {}
    $ui.param("blogFile", function(value, name, view, params) {
        if (blogCache[value]) return $d[name] = blogCache[value]
        var cb = view.wait()
        xhr("GET", value + ".text", function(err, body) {
            $d[name] = blogCache[value] = body || "Error"
            cb()
        }).send()
    })
```

**View with file dependencies (from test/html/routed.html):**
```
%view about #pub
;f "contacts.json"
main
    h2 About
    ul
        li ;each!"row", contacts
            a ;txt row.title;href!row.link
```

**Inline markup (`;t` binding):**
```
dd ;t 'Contribute on [[*GitHub*] <https://github.com/litejs/>].'
```

**Full SVG SPA (from test/html/svg-spa.html):**
```
%js
    El.cache["a"] = document.createElementNS("http://www.w3.org/2000/svg", "a")

%view main #
    svg[viewBox="0 0 200 100"]
        a.Menu-item ;xlink: "#home"
            text[x=10][y=40] Home
        svg[x=60][y=30]
            %slot

%view home main
    text[x=5][y=15] Main page

%start
```
