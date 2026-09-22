## Lesson 1
Define can bind a value or a function
- To bind a value it takes a name and an expression and binds the name to the value of the expression (define x 3)
- To bind a function, it takes a header and a body (define (double x) (+ x x))

Equality on naturals: nat=?

Pluck has no print. The only way to see anything is (query name (Marginal expr)), which returns a distribution in  two columns, the value and its probability. A deterministic expression is one row at probability 1.0. A certain value is a distribution with all its mass on one point.

Pluck has no infix. All operators are prefix.

You need to make sure you have the correct number of arguments per function. Pluck will return a function rather than an error if you miss an arg. 

## Lesson 2
flip is the primitive source of randomness. 
- Uniform, discrete, and geom are all built on top of it. 
- (flip 0.9) returns true with probability 0.9.

## Lesson 3
Let is how you get persistent structure in Pluck:
(let (bindings) body)
e.g (let c (flip 0.9) and c c c)

- let takes a list of bindings. 
- Be super careful not to call a random function more than once when you meant to re-use the draw. 

## Lesson 4

Currying: multi-arg functions are actually chains of one-arg functions. (f x y z) is sugar for (((f x) y) z). That's why calling a function with fewer args than it takes returns a function. 

Partial application is a payoff of currying: you can write a function with a building block function without using all of its args. 

Functions are values, but that means that arity mistakes don't error. 

## Lesson 5 

Uniform has no fixed arity: (uniform e1... en) It picks one of the expressions with equal probability. 

Remember, flip is the only source of randomness in Pluck. 

uniform is a macro. It's syntactic sugar which the compiler expands into code (nested flips) before runtime. Each flip selects a half until the result. 

discrete (discrete (e1 p1) ... (en pn)) chooses from a discrete distribution with fixed probabilities that sum to 1. Discrete is also a macro so the probabilities are literal numbers in the source, not values computed at runtime. 

Both uniform and discrete work with any expressions including your constructor values, they're not confined to numbers. 

"Query whether the spinner landed on 2" - it's a boolean question. 
