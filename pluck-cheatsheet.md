# Pluck Quick Reference

Running cheatsheet. New building blocks get appended as each lesson introduces them.

## Session startup

From the `Pluck.jl` folder:

```
julia
```
```julia
using Pkg; Pkg.activate("."); using Pluck
load_pluck_file("programs/simple_example.pluck");
```

All three parts of that first line are needed **every session**. Without `Pkg.activate(".")`, `using Pluck` won't find the package; without `using Pluck`, you get `UndefVarError: load_pluck_file not defined`.

`Pkg.instantiate()` was one-time — not needed again.

Check `pwd()` if `Pkg.activate(".")` misbehaves. VS Code's terminal opens wherever the editor is, which may not be `Pluck.jl`. Fix with `cd("/Users/Charlie/Pluck.jl")` from inside Julia.

Exit with `exit()` or Ctrl+D.

---

## Lesson 1 — Values, define, and arithmetic

| Form | Arity | What it does |
|---|---|---|
| `(define name expr)` | — | Bind a name to a value — `(define x 3)` |
| `(define (name arg) body)` | — | Define a function of one or more arguments |
| `+` `-` `*` | 2 | Arithmetic on naturals, prefix only: `(+ 2 3)`. Use nesting: `(+ x (+ x x))` |
| `nat=?` | 2 | Equality test for naturals: `(nat=? x 6)`. `==` also works, but `nat=?` is what Pluck's standard library uses |
| `(query name expr)` | 2 | The only way to see a value. Pluck has no `print`. Always ask a query for a distribution, even one with a single point in it. `expr` must be a `Marginal` / `Posterior` / `PosteriorSamples` |

**Check your arities** because Pluck curries everything: supplying too few arguments hands back a function waiting for the rest. So an arity mistake fails silently or confusingly rather than cleanly.

**Reading nested expressions:** Work inside-out. The thing you're asking about is whatever sits outermost inside `Marginal`:

```scheme
(query nine-check (Marginal (is-six (triple 3))))
;;                                  └─ 1. triple 3 → 9
;;                          └─ 2. is-six 9 → false
;;                 └─ 3. Marginal reports the distribution of that
;; => false   1.0
```

`Marginal` always means "the distribution of this expression's value," whether or not anything random happened. A deterministic value is a distribution with one point at probability 1.0.

### Reading the output

Two columns: **value** on the left, **probability** on the right.

```
false   1.0     ;; the value is `false`, and it's certain
```

Only outcomes with non-zero probability get a row. There's no `true 0.0` line, because `true` isn't a possible outcome of a deterministic expression that returns `false`. Probabilities always sum to 1 across whatever rows are present.

---

## Lesson 2 — Randomness

| Form | Arity | What it does |
|---|---|---|
| `flip` | 1 | `(flip 0.9)` returns `true` with probability 0.9, `false` otherwise. The **only** primitive source of randomness — `uniform`, `discrete` and `geom` are all built on it |
| `and` | 2 | Boolean and, prefix like everything else: `(and A B)`. Returns a boolean, so it needs no `if` wrapper |
| `if` | 3 | `(if condition then else)`. Exactly three arguments |

**Each occurrence of `flip` is a separate coin.** Two written flips means two independent coins:

```scheme
(define (two-coins)
  (and (flip 0.5) (flip 0.5)))

(query two-coin-flips (Marginal (two-coins)))
;; => true    0.25
;;    false   0.75
```

(That's a consequence of writing `flip` twice, not of what `and` does.) 

**Only wrap in `if` when the task asks for specific return values.** `lopsided-coin` needed `if` because the task wanted 1 and 0. `two-coins` doesn't, because `and` already returns a boolean.

Zero-argument functions are how you delay a flip until the function is called:

```scheme
(define (coin) (if (flip 0.5) 1 0))   ;; ✓ flip happens when called
(define coin (if (flip 0.5) 1 0))     ;; ✗ flip happens at definition
```

`(define (f) body)` is sugar for `(define (f _) body)`, and `(f)` is sugar for `(f (Unit))`. `_` is the conventional name for an ignored argument. Same mechanism behind `(lambda -> ...)` later.

---

## Gotchas

*(running list)*

**Every new terminal session needs the full setup line** before `load_pluck_file` exists:

```julia
using Pkg; Pkg.activate("."); using Pluck
```

`UndefVarError: load_pluck_file not defined` means this was skipped. `REPL[1]` in the stacktrace confirms a fresh session.

**Trailing `;` suppresses Julia's echo.** Without it, `load_pluck_file` prints your query results *and* a dump of every form it parsed.

```julia
load_pluck_file("programs/lessons.pluck");
```

**Query names must be distinct.** Two queries sharing a name, or sharing a name with a function, makes output unreadable. They're just labels; pick different ones.

**"Expected closing paren" often means wrong arity, not unbalanced brackets.** Check argument numbers before counting brackets.

**Pluck is case-sensitive.** `query`, not `Query`.

**Floating-point noise is normal.** `0.09999999999999998` is `1 - 0.9` in binary. Pluck's exactness is about **method, not precision** — it computes probabilities algebraically by summing over the possibility space, rather than estimating them by running the program many times. So there's no sampling error, no variance. But the arithmetic still happens in ordinary 64-bit floats, so representation error remains.

---

## Workflow

Edit `.pluck` file in VS Code → save → re-run `load_pluck_file` in the open Julia session. No need to restart Julia between edits.

Definitions accumulate across loads, so if a redefinition behaves strangely, restart Julia for a clean slate.

VS Code doesn't know `.pluck` — set the language to **Clojure** (bottom-right status bar) for bracket matching. Scheme isn't built in; Clojure is, and `;;` is its comment character too. Use "Configure File Association for '.pluck'" to make it stick.

**Finding a function's arity:**
- the primer's reference section near the front of the PDF
- Pluck's standard library source in the cloned repo (every function is defined there in plain Pluck)
