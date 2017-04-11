# Pattern Matching

Pattern matching deconstructs a value to bring zero or more expressions into scope. Pattern matches are introduced with the `case` keyword.

Pattern matches have the following general form:

```purescript
case value of
  pattern -> result
  ...
  pattern -> result
```

Pattern matching can also be used in the declaration of functions, as we have already seen:

```purescript
fn pattern_1 ... pattern_n = result
```

Patterns can also be used when introducing functions. For example:

```purescript
example x y z = x * y + z
```

The following pattern types are supported:

- Wildcard pattern
- Literal patterns
- Variable pattern
- Array patterns
- Constructor patterns
- Record patterns
- Named patterns
- Guards

The exhaustivity checker will introduce a `Partial` constraint for any pattern which is not exhaustive.
By default, patterns must be exhaustive, since this `Partial` constraint will not be satisfied. The error can be silenced, however, by adding a local `Partial` constraint to your function.

Wildcard Patterns
-----------------

The wildcard `_` matches any input and brings nothing into scope:

```purescript
f _ = 0
```

Literal Patterns
----------------

Literal patterns are provided to match on primitives:

```purescript
f true = 0
f false = 1

g "Foo" = 0
g _ = 1

h 0 = 0
h _ = 1
```

Variable Patterns
-----------------

A variable pattern matches any input and binds that input to its name:

```purescript
double x = x * 2
```

Array Patterns
--------------

Array patterns match an input which is an array, and bring its elements into scope. For example:

```purescript
f [x] = x
f [x, y] = x * y
f _ = 0
```

Here, the first pattern only matches arrays of length one, and brings the first element of the array into scope.

The second pattern matches arrays with two elements, and brings the first and second elements into scope.

Constructor patterns
--------------------

Constructor patterns match a data constructor and its arguments:

```purescript
data Foo = Foo String | Bar Number Boolean

foo (Foo s) = true
foo (Bar _ b) = b
```

Record Patterns
---------------

Record patterns match an input which is a record, and bring its properties into scope:

```purescript
f { foo: "Foo", bar: n } = n
f _ = 0
```

Nested Patterns
---------------

The patterns above can be combined to create larger patterns. For example:

```purescript
f { arr: [x, _], take: "firstOfTwo" } = x
f { arr: [_, x, _], take: "secondOfThree" } = x
f _ = 0
```

Named Patterns
--------------

Named patterns bring additional names into scope when using nested patterns. Any pattern can be named by using the ``@`` symbol:

```purescript
f a@[_, _] = true
f _ = false
```

Here, in the first pattern, any array with exactly two elements will be matched and bound to the variable `a`.

Guards
------

Guards are used to impose additional constraints inside a pattern using boolean-valued expressions, and are introduced with a pipe after the pattern::

```purescript
evens :: List Int -> Int
evens Nil = 0
evens (Cons x xs) | x `mod` 2 == 0 = 1 + evens xs
evens (Cons _ xs) = evens xs
```

When using patterns to define a function at the top level, guards appear after all patterns:

```purescript
greater x y | x > y = true
greater _ _ = false
```
