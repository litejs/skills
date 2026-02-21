---
name: litejs-build
description: Use when working with LiteJS build system — HTML build attributes (cat, min, drop, inline, exclude, banner), lj build / lj b commands, asset handling, or editing @litejs/cli/lib/build.js
---

# LiteJS Build System

HTML-driven build tool. Declare minification rules as attributes on `<link>` and `<script>` elements — no extra config files.

## Commands

```sh
lj build --out=index.html dev.html     # full form
lj b --out=ui/index.html ui/dev.html   # shorthand
lj b                                   # uses package.json#litejs/build
```

## CLI Options

| Option | Default | Effect |
|---|---|---|
| `--assets` | `"{h}.{ext}"` | URL template for assets copied to output dir. Placeholders: `{h}` git hash, `{ext}` extension, `{name}` original name |
| `--banner` | `""` | Add commented banner to output |
| `--cat` | `true` | Build src files |
| `--fetch` | `true` | Fetch remote resources |
| `--jsmin` | `""` | JS minification command (default: uglifyjs). Custom commands read stdin without `--` |
| `--min` | `""` | Minified output file |
| `--out` | `""` | Output file. For JS/UI output: only indent/remove empty lines. `--min` fully minimizes |
| `--readme` | `""` | Replace readme tags in file |
| `--ver` | `""` | Override version string |
| `--worker` | `""` | Update worker file |

Options can be set in `package.json#litejs` or `.github/litejs.json`.

## HTML Attributes

Attributes on `<link>` and `<script>` elements control the build:

| Attribute | Effect |
|---|---|
| `cat="file.js"` | Concatenate listed source files into this element's src. Comma/space-delimited |
| `cat-drop="flags"` | Configuration flags applied during source build (cat phase only) |
| `min="target.js"` | Minimize content to target file. `{h}` replaced with git hash |
| `min=""` | Empty min appends to previous min target of same type |
| `drop="flags"` | Configuration flags applied during minification (min phase only) |
| `inline` | Embed file content directly into the HTML output |
| `exclude` | Remove element from production output |
| `banner="text"` | Add banner comment to the minified file |
| `if="expr"` | Load condition for dynamic code loader |
| `integrity` | Recalculate SHA-256 integrity hash |
| `rewrite="glob:target"` | Rewrite file references with hashed names |

### Path Expansion

`+` and `%` in `cat` and `min` values expand relative to the previous path:

```
cat="@litejs/ui,+/pointer.js"  → @litejs/ui, @litejs/ui/pointer.js
min="app.js?{h}"               → app.js?<git-hash>
min="%.min.js"                 → app.min.js (replaces from last .)
```

## Build Phases

The HTML build runs in strict order. Understanding this prevents ordering bugs.

### Phase 1: Exclude
Remove elements with `exclude` attribute.

### Phase 2: Cat (source build)
Process elements with `cat` attribute and their siblings (`cat=""`).
- Reads source files, applies `cat-drop` flags
- Concatenates siblings into the cat element
- Writes result to inDir (source folder)
- Copies to outDir if different

### Phase 3: Min (minification setup)
Process elements with `min` attribute and their siblings (`min=""`).
- Groups elements by min target
- Reads source files, applies `drop` flags
- `.ui` view files split: `%css` → CSS element, `%js` → JS element, template → UI element

### Phase 4: Asset copy
Copy `[src]` and `[href]` elements to outDir (when building to different directory).

### Phase 5: Pending writes
Minimize content and queue writes (not immediate — allows parseView appends).

### Phase 6: Inline
Embed `[inline]` and remaining `[min]` elements into HTML.
- Handles `{loadFiles}` and `{loadRewrite}` placeholders
- Flushes pending writes after inline processing

### Phase 7: Output
Serialize HTML. CSS is minified with `@import` resolution and URL rewriting.

## Configuration Flags (drop / cat-drop)

Toggleable comment blocks that can be activated per build target:

```javascript
/*** ie8 ***/
function legacyCode() {}  // included in source
/*/
function legacyCode() {}  // minimal stub for production
/**/
```

`drop="ie8"` on a `min` element flips the toggle — the first block is commented out, the second becomes active.

- `drop` — applied during min phase only
- `cat-drop` — applied during cat phase only
- Multiple flags: `drop="ie8 debug"` (space-separated)

## Asset Handling

When building to a different output directory (`--out=build/index.html dev.html`):

1. **HTML assets** — `<img src>`, `<link href>` elements are copied via `cpOut`
2. **CSS assets** — `url()` references are copied and renamed using `--assets` template
3. **View assets** — `[src=...]` in `.ui` templates are copied and renamed (only `src`, not `href` — links are left untouched)

The `{h}` placeholder in `min` attribute and `--assets` uses abbreviated git object hashes from `git ls-files --abbrev=1`.

Skipped URLs:
- Scheme URLs (`data:`, `http:`, `mailto:`, etc.) — detected by `/^\w+:/`
- Fragment-only references are preserved: `icons.svg#star` → copies `icons.svg`, returns `hash.svg#star`

## Example: Full dev.html

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <base href=/ >
    <!-- Build CSS: concat base+theme, minimize to min.css -->
    <link rel="stylesheet" href="app.css" min="min.css?{h}" cat="@litejs/ui/css/base.css,%.grid.css">
    <link rel="stylesheet" href="theme.css" min>

    <!-- Inline the loader for faster first paint -->
    <script src="load.js" inline cat="@litejs/ui/load"></script>
</head>
<body>
    <!-- Main app bundle -->
    <script src="app.js" min="app.min.js?{h}" banner="MIT" cat="@litejs/ui,%/model">
    </script>
    <script src="polyfill.js" min="%.min.js" cat="@litejs/ui/polyfill/es5.js" if="!Function.prototype.bind"></script>

    <!-- View templates -->
    <script type="ui" src="views.ui" min="views.ui"></script>

    <!-- Dev-only tools -->
    <script src="debug.js" exclude></script>
</body>
</html>
```

Build: `lj b --out=build/index.html ui/dev.html`

## Internals (lib/build.js)

Key functions for contributors:

| Function | Purpose |
|---|---|
| `build(opts)` | Entry point, dispatches to `html()` or `jsMin()` |
| `html(opts, next)` | HTML build pipeline |
| `clean(str)` | Remove whitespace preserving strings and operators |
| `drop(el, content, attr)` | Flip toggleable comments matching flags |
| `defMap(str)` | Path expansion (`+` append, `%` suffix replace) |
| `cssMin(str, urlFn)` | Minify CSS via `@litejs/dom` `CSS.minify()` with optional URL callback |
| `parseView(content, extTo, lastMinEl, urlFn)` | Split `.ui` into `%css`/`%js`/template, rewrite `[src=]` URLs |
| `viewMin(str, attrs)` | Minify view template (join bindings, compact selectors) |
| `write(dir, name, content, el)` | Write file, handle `{h}` hash replacement |

### Exported for testing

```javascript
var build = require("./lib/build.js")
build.clean(str)
build.drop(el, content, attr)
build.defMap(str)
build.cssMin(str, urlFn)
build.parseView(content, extTo, lastMinEl, urlFn)
```

## JS/UI Build (non-HTML)

Build directly to `.js` or `.ui` output without an HTML file:

```sh
# Build JS bundle from mixed inputs (.js, .css, .ui/.view files)
lj b --min=app.min.js src/app.js src/style.css src/views.ui

# Build .ui output (unminified JS/CSS, indented sections)
lj b --out=app.ui src/app.js src/style.css src/views.ui

# Build .ui output (minified JS/CSS/UI)
lj b --min=app.min.ui src/app.js src/style.css src/views.ui
```

Input file transforms:
- `.ui`/`.view` → split into `%css`/`%js`/template sections (JS output: `xhr.css()`/`xhr.ui()` calls)
- `.css` → JS output: wrapped in `xhr.css()`, UI output: `%css` section
- `.js` → JS output: concatenated, UI output: `%js` section

Consecutive `xhr.css()` calls are merged in JS output.

### Custom JS minifier

```sh
# Use esbuild instead of uglifyjs
lj b --jsmin="esbuild --minify --loader=js" --min=app.min.js app.js

# Use terser
lj b --jsmin="terser -c -m" --min=app.min.js app.js
```

Can be set in `.github/litejs.json`:

```json
{
  "build": {
    "jsmin": "esbuild --minify --loader=js"
  }
}
```

## Common Patterns

**Cat without min** — build source file only, no minification:
```html
<script src="app.js" cat="src/a.js,src/b.js"></script>
```

**Min without cat** — minify existing file:
```html
<script src="app.js" min="app.min.js"></script>
```

**Inline view with separate min** — views inlined in HTML, also written to file:
```html
<script type="ui" src="views.ui" min="views.min.ui"></script>
```

**Hash-based cache busting**:
```html
<script src="app.js" min="app-{h}.js"></script>
<link rel="stylesheet" href="app.css" min="app-{h}.css">
```

**Different drop flags for cat vs min**:
```html
<script src="app.js" cat="src/*.js" cat-drop="dev" min="app.min.js" drop="ie8"></script>
```
