# Pluck — Lessons 1 to 8

## Running things

```
cd ~/Pluck.jl
julia
```
```julia
using Pkg; Pkg.activate("."); using Pluck
load_pluck_file("path/to/lessons.pluck");
```

The trailing `;` hides Julia's echo. Pluck code goes in the `.pluck` file, never at the `julia>` prompt. Edit, save, re-run the load line — no need to restart Julia.

---

## Everything is prefix

No infix operators. `(+ 2 3)`, never `2 + 3`.

Arity matters more than usual, because Pluck curries: too few arguments hands back a function instead of raising an error.

| | Arity | |
|---|---|---|
| `+` `-` `*` | 2 | `(+ x (+ x x))` for three things |
| `nat=?` | 2 | equality on naturals |
| `constructor=?` | 2 | same tag? works on any ADT |
| `not` | 1 | |
| `and` | 2 | |
| `if` | 3 | condition, then, else |
| `flip` | 1 | |

---

## define

```scheme
(define x 3)                    ;; value — no parens round the name
(define (double x) (+ x x))     ;; function — parens round name + parameters
(define (make-weather) ...)     ;; zero-arg function — still parens
```

Parens go round **the thing being defined**, not the body.

The zero-argument case matters: `(define (coin) (flip 0.5))` flips when called. `(define coin (flip 0.5))` flips once at load time and binds the result. Randomness belongs in queries, not defines.

`(define (f x y) body)` is sugar for `(define f (lambda x y -> body))`. `lambda` makes an unnamed function inline — needed when a function has to *return* a function.

---

## Seeing things

No `print`. `(query name (Marginal expr))` is the only way.

Output is two columns: **value** on the left, **probability** on the right.

```
false   1.0
```

Only outcomes with non-zero probability get a row, so a deterministic expression gives one row at 1.0. Row order carries no meaning.

`Marginal` = the distribution of this expression, everything else summed out.

Keep `Marginal` out of function bodies. Functions build the model; queries ask things of it.

---

## Randomness

`flip` is the only primitive. `uniform`, `discrete`, `geom` are all built on it.

```scheme
(uniform e1 ... en)                 ;; equal weights, any number of options
(discrete (e1 p1) ... (en pn))      ;; given weights, must sum to 1
```

Both are **macros** — the compiler expands them into nested flips before runtime. So `discrete`'s probabilities must be literal numbers in the source. `flip` is a real function, so its argument can be computed:

```scheme
(flip (if b 0.2 0.7))                          ;; ✓
(discrete (x (if b 0.2 0.7)) (y ...))          ;; ✗
```

Options can be any expression, including your own constructor values.

---

## One draw or several — the Lesson 3 point

**Each written occurrence of a random expression is its own draw.**

```scheme
(and (flip 0.5) (flip 0.5))        ;; two coins → true 0.25
(let ((c (flip 0.5))) (and c c))   ;; one coin  → true 0.5
```

Both run clean. Neither errors. If you meant one coin and wrote two, you get a plausible wrong answer.

`let` is how you write a draw once and use it in several places:

```scheme
(let ((name expr))     ;; list of bindings — note the two layers of parens
  body)                ;; body goes INSIDE the let
```

The commonest bug is closing the `let` before the body.

This matters because real models have something latent generating several observations — one biased coin flipped repeatedly, one urn drawn from three times. `let` is how "one" gets said.

Separate queries are not this bug. The bug needs one answer built from two draws that should have been one.

---

## Types and constructors

```scheme
(define-type weather (Sunny) (Rainy) (Cloudy))
(define-type list (Nil) (Cons any list))
(define-type nat (O) (S nat))
```

A declaration, not a function. It lists the **alternatives** a type offers.

- `Cons` is a **constructor** — the tag
- `(Cons 1 (Nil))` is a **value** — tag plus contents

A constructor isn't a function. A function computes and the call disappears; a constructor wraps its arguments and keeps the tag. The tag is what makes the value inspectable later.

`constructor=?` compares tags only, ignoring contents — so it's real equality only for constructors that carry no data, like booleans.

Booleans are an ADT: `(define-type bool (True) (False))`. You write `(True)`, output prints `true`.

**Recursive types have two constructors: one that stops, one that continues.** `Nil`/`Cons`, `O`/`S`. That's what makes unbounded size possible.

---

## match

Takes a value apart by asking which constructor built it.

```scheme
(match l
  Nil => 0
  Cons x rest => (+ 1 (my-length rest)))
```

`x` and `rest` are your names, bound to what the constructor held. The constructor names are fixed by the type.

Two hard constraints:

- **No `else`.** Every constructor needs its own branch.
- **Patterns are flat.** `Cons x (Nil)` is illegal — you can't look two levels deep. Use a nested `match` instead.

`match` looks **once**. It's a fork, not a loop.

---

## Recursion over lists

A list is either empty or an element plus a smaller list. So a function over it has two branches, and one of them recurses.

```scheme
(define (my-length l)
  (match l
    Nil => 0
    Cons x rest => (+ 1 (my-length rest))))
```

`rest` isn't shortened — it *is* the list that was sitting in the second slot, already one element smaller. Passing it is what makes the recursion terminate: you reach `Nil` eventually, and `Nil` doesn't recurse.

**Consuming vs producing.** `match` takes a list apart; `Cons` puts one together.

```scheme
(define (random-list)
  (if (flip 0.5)
      (Nil)
      (Cons (flip 0.5) (random-list))))
```

Nothing is passed in, so there's nothing to match on — the coin decides whether to stop. Zero arguments.

Checking emptiness: match on the tag, don't count.

```scheme
(define (empty? l)
  (match l
    Nil => (True)
    Cons x rest => (False)))
```

`(nat=? (length l) 0)` walks the whole list when the first constructor already told you — and laziness makes that worse than merely wasteful.

---

## Errors you have actually hit

**"Expected closing paren" often means wrong arity.** `if` with five arguments reads as a bracket error. Count arguments before brackets.

**Body outside the form.** `(define (f x))` on its own line, body underneath — the `define` closed early. Same shape as the `let` bug. Cursor next to the opening paren; VS Code shows you where it closes.

**`+1` is one token.** Whitespace separates names. `(+ 1 ...)`.

**Passing a function where a value belongs.** `(empty? random-list)` hands over the function; `(empty? (random-list))` calls it first.

**Case-sensitive.** `query`, not `Query`.

**Floating-point noise is normal.** `0.0999999...` is `1 - 0.9` in binary. Pluck's exactness is about method — no sampling error — not infinite precision.
