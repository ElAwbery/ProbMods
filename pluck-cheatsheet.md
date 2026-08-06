# Pluck Quick Reference

Running cheatsheet. New building blocks get appended as each lesson introduces them.

## Session startup

From the `Pluck.jl` folder:

```
julia
```
```julia
using Pluck
load_pluck_file("programs/simple_example.pluck")
```

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

`(Marginal (triple 3))` and `(Marginal (is-six (triple 3)))` are different questions — the number itself, versus the test applied to the number. Easy to conflate early on.

`Marginal` always means "the distribution of this expression's value," whether or not anything random happened. A deterministic value is a distribution with one point at probability 1.0.

```scheme
(define (double x) (+ x x))

(query lesson1-check (Marginal (double 3)))
;; => 6   1.0
```

### Reading the output

Two columns: **value** on the left, **probability** on the right.

```
false   1.0     ;; the value is `false`, and it's certain
```

Only outcomes with non-zero probability get a row. There's no `true 0.0` line, because `true` isn't a possible outcome of a deterministic expression that returns `false`. Probabilities always sum to 1 across whatever rows are present.

With one row, the two columns blur together — easy to misread `9 1.0` as a single fact. They separate visually as soon as Lesson 2 gives you two rows.

---

## Gotchas

*(running list)*

**Trailing `;` suppresses Julia's echo.** Without it, `load_pluck_file` prints your query results *and* a dump of every form it parsed. 

```julia
load_pluck_file("programs/lessons.pluck");
```

**Query names must be distinct.** Two queries sharing a name — or sharing a name with a function — makes output unreadable. They're just labels; pick different ones.

Pluck's exactness is about **method, not precision**. It computes probabilities algebraically by summing over the possibility space, rather than estimating them by running the program many times. So there's no sampling error, no variance. But the arithmetic still happens in ordinary 64-bit floats, so representation error remains.

---

## Workflow

Edit `.pluck` file in VS Code → save → re-run `load_pluck_file` in the open Julia session. No need to restart Julia between edits.

Definitions accumulate across loads, so if a redefinition behaves strangely, restart Julia for a clean slate.

VS Code doesn't know `.pluck` — set the language to **Clojure** (bottom-right status bar) for bracket matching. Scheme isn't built in; Clojure is, and `;;` is its comment character too. Use "Configure File Association for '.pluck'" to make it stick.

**Finding a function's arity:** 
- the primer's reference section near the front of the PDF
- Pluck's standard library source in the cloned repo (every function is defined there in plain Pluck)

---

