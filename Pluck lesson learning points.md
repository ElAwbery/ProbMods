## Lesson 1
Define can bind a value or a function
- To bind a value it takes a name and an expression and binds the name to the value of the expression (define x 3)
- To bind a function, it takes a header and a body (define (double x) (+ x x))

Equality on naturals: nat=

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
