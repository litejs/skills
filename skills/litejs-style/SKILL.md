---
name: litejs-style
description: Use when writing or modifying JavaScript, CSS, or commit messages in LiteJS projects — enforces non-standard conventions like no-semicolons, comma-first declarations, and same-level switch/case
---

# LiteJS Style Guide

Coding conventions for LiteJS projects. These diverge from standard JS/CSS defaults — follow them exactly.

## JavaScript

### No Semicolons

Do NOT add semicolons at end of lines. Use a semicolon ONLY before `[`, `(` or `/` at the beginning of a line:

```javascript
var arr = [1, 2]
;[3, 4].forEach(fn)
;(function() {})()
```

### Comma-First Declarations

Uninitialized variables first on one line, then comma-first for initialized:

```javascript
var foo, bar
, MAX_AGE = 120
, baz = "baz"
, quux = 123
```

### Leading Dot for Chains

```javascript
person
.be("confused")
.do("makeup")
```

### Switch/Case Same Level

```javascript
switch (sex) {
case "male":
	person.wear("bow tie")
	break
default:
	person.be("confused")
}
```

### Functions

- Prefer declarations over expressions: `function a() {}` not `var a = function() {}`
- Don't use `that` for outer `this` — use lowercased constructor name: `var person = this`
- Pass scope instead of binding: `arr.map(fn, scope)` not `arr.map(fn.bind(scope))`
- Accept optional scope argument in functions with callbacks

### Naming

| Convention | Use |
|---|---|
| `lowerCamelCase` | methods, variables |
| `UpperCamelCase` | constructors |
| `CAPITAL_SNAKE_CASE` | symbolic constants |
| `_prefix` | private methods/properties |

### Spacing

- One space before `{`, after `;` in for loops, around binary operators `1 + 2`
- No space between unary operator and variable: `i++`
- Braces on same line as statement

## General

- Tabs for indentation, spaces for alignment within a line
- Max 80 character lines (don't break greppable strings like log messages)
- Double quotes; single only to avoid escaping
- UNIX newlines, newline at end of file
- No trailing whitespace, no indented empty lines
- Hyphens instead of spaces in file/directory names

## CSS

- One selector per line, one rule per line
- SUIT CSS naming: `[<namespace>-]<ComponentName>[-descendentName][--modifierName]`
- `.is-` modifiers only combined with a component (`.Btn.is-error`, never `.is-error` alone)
- No ID selectors, no preprocessor, no media queries
- Align vendor-prefixed properties with spaces
- Shorthand properties where possible

```css
.Icon {
	width:  16px;
	height: 16px;

	-webkit-transition: all 4s ease;
	   -moz-transition: all 4s ease;
	        transition: all 4s ease;
}

.Icon--person   { background-position: -16px   0  ; }
.Icon--files    { background-position:   0   -16px; }
```

## Commits

Format: `scope: Capitalized summary up to 50 chars`

- Imperative present tense: Add, Fix, Update, Remove, Refactor, Document, Test
- Optional scope narrows area: build, ci, docs, style, test, ui/form
- `! ` prefix for breaking changes: `! Rewrite public api`
- No period at end of subject
- One commit = one thing
- Fix bug = write regression test

## Common AI Mistakes

| AI default | Correct LiteJS style |
|---|---|
| `var x = 1;` | `var x = 1` (no semicolon) |
| `var x = 1, y = 2` | Comma-first (see above) |
| `  case "a":` indented | `case "a":` same level as switch |
| `.bind(this)` | Pass scope argument |
| `var that = this` | `var person = this` (constructor name) |
| `const fn = () => {}` | `function fn() {}` (declaration) |
| `'single quotes'` | `"double quotes"` |
| Semicolons everywhere | Only before `[`, `(`, `/` at line start |
