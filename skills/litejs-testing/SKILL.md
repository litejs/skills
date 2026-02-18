---
name: litejs-testing
description: Use when writing, running, or debugging tests in LiteJS projects using test.js framework, lj test / lj t commands, or node -r ./test.js
license: MIT
---

# LiteJS Testing Framework

Zero-dependency test framework. ES5 compatible. Runs in Node.js and browsers.

## Running Tests

```sh
lj test                    # run test/*.js (default glob)
lj t                       # shorthand
lj t test/foo.js           # specific file
lj t 5                     # run only test case #5
lj t --tap                 # TAP output
lj t --brief               # failures only
lj t --watch               # re-run on file change
lj t --up                  # update snapshots
lj t --seed=123            # deterministic Math.random
lj t --timeout=5000        # default test timeout ms
lj t --no-color            # disable ANSI colors
lj t --tz=UTC              # set timezone
node -r ./test.js test.js  # direct invocation
```

## Test Structure

```js
describe("Suite", function() {
    test("case", function(assert) {
        assert.ok(true)
        assert.end()
    })
    it("returns promise", function(assert) {
        return Promise.resolve()  // auto-ends on resolve/reject
    })
    it("with mock", function(assert, mock) {
        // mock available when fn has 2+ params
        assert.plan(2)  // auto-ends after 2 assertions
        assert.ok(1).ok(1)
    })
    it("pending test")           // no fn = pending/skipped
    it("skipped", false, fn)     // data=false = skipped
})
```

**Variants:** `describe` (suite), `test`/`it` (case, "it" prefixes output), `should` (prefixes "it should").
**Chaining:** `describe(...).test(...).it(...)` — all return `describe`.
**Object suites:** `describe("Name", { "case": fn, "nested": { "sub": fn } })`

## Assert Methods

All chainable, return `assert`. The assert param is itself callable: `assert(value, msg, actual, expected)`.

| Method | Check |
|---|---|
| `ok(value)` | truthy |
| `notOk(value)` | falsy |
| `equal(actual, expected)` | deep equal (handles NaN, Date, RegExp, circular) |
| `notEqual(actual, expected)` | not deep equal |
| `strictEqual(actual, expected)` | `===` |
| `notStrictEqual(actual, expected)` | `!==` |
| `own(actual, expected)` | actual has all properties of expected (subset match) |
| `notOwn(actual, expected)` | inverse of own |
| `throws(fn)` | fn must throw |
| `type(value, name)` | `describe.type(value) === name` |
| `anyOf(value, array)` | value in array |

**Control:** `assert.end()`, `assert.plan(n)`, `assert.setTimeout(ms)`

## Data-Driven Tests

```js
it("adds {0} + {1}", [
    [1, 2, 3],
    [4, 5, 9]
], function(a, b, expected, assert) {
    assert.equal(a + b, expected).end()
})

// Suite-level tables
describe("env {0}", [["dev"], ["prod"]], function(env) {
    it("case {0}", [[1], [2]], function(val, assert) {
        assert.end()
    })
})
```

`{0}`, `{1}` in name are replaced from each row. Data args come before `assert, mock` params.

## Mock

Available as second parameter when test function has 2+ args. Auto-restored after `assert.end()`.

### mock.fn([behavior])

```js
var spy = mock.fn()                     // returns undefined
var wrap = mock.fn(originalFn)          // wraps, tracks calls
var cycle = mock.fn(["a", "b", "c"])   // cycles return values
var map = mock.fn({"1": "one", "*": "default"})  // maps args
var val = mock.fn(42)                   // always returns 42
var async = mock.fn(originalFn, true)   // returns Promise
var cb = mock.fn(originalFn, 0)         // calls args[0](err, result)

spy.called    // call count
spy.calls     // [{scope, args, error, result}, ...]
spy.errors    // error count
spy.results   // [returnVal, ...]
```

### mock.spy / mock.swap

```js
mock.spy(obj, "method")              // wrap with spy
mock.spy(obj, "method", stubFn)      // replace with spy
mock.swap(obj, "key", newValue)      // replace value
mock.swap(obj, {a: 1, b: 2})        // replace multiple
mock.restore()                       // manual restore (auto on end)
```

### mock.time / mock.tick

Replaces `Date`, `setTimeout`, `setInterval`, `setImmediate`, `clearTimeout`, `clearInterval`, `clearImmediate`, `process.nextTick`, `process.hrtime`.

```js
mock.time()                          // freeze at current time
mock.time("2024-01-15T10:30:00Z")   // freeze at specific time
mock.time(1705312200000)             // freeze at epoch ms
mock.tick(100)                       // advance 100ms, run timers
mock.tick()                          // advance to next timer
```

### mock.rand

```js
mock.rand()        // deterministic random using conf.seed
mock.rand(12345)   // specific seed
// Math.random() now returns reproducible sequence
```

## Snapshots

Requires `require("../snapshot.js")` in test file.

```js
assert.matchSnapshot("file.js", actual)       // compare to file.snap.js
assert.matchSnapshot("file.js", transformFn)  // transformFn(fileContents)
assert.cmdSnapshot("lj build f.js", "f.js")  // run cmd, compare stdout
assert.cmdSnapshot("cmd", "f.js", { expectFail: true })  // non-zero exit ok
```

Update all snapshots: `lj t --up`

## Utilities on `describe`

```js
describe.equal(a, b)           // standalone deep equal -> boolean
describe.type(value)           // -> "string","number","array","object","null","nan",...
describe.stringify(value, max) // custom stringifier
describe.diff(a, b, sep)      // diff two strings
describe.format(str, obj)     // "{key}" template substitution
describe.opts(argv, defaults)  // parse --flag=value args
describe.conf                  // live config object
describe.print(str)            // output a line
```
