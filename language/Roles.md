# Roles

Solving [Prim.Coerce.Coercible](https://pursuit.purescript.org/builtins/docs/Prim.Coerce#t:Coercible) constraints requires the compiler to know the _role_ played by type parameters in the runtime representation of their type. There's three roles, from most to least restrictive: _nominal_, _representational_ and _phantom_.

The primitives `->`, `Array` and `Record` types have _representational_ parameters and roles are otherwise inferred based on their appearance in the right hand side of a data type declaration.

## Role inference

* Nominal roles are inferred for parameters of foreign data types, since we do not have enough information to safely choose a less restrictive role.

Nominal parameters are only coercible to themselves:

```purescript
foreign import data Nominal :: Type -> Type
```

`Coercible (Nominal a) (Nominal b)` does not hold, even when `Coercible a b` does.

Albeit a safe default, this is often too constraining. More coercions can be allowed with a [role annotation](#role-annotations) when it is known to be safe.

Nominal roles are also inferred for constrained parameters:

```purescript
newtype Shown a = Shown ((Show a => a -> String) -> String)
```

`Coercible (Shown a) (Shown b)` does not hold, even when `Coercible a b` does.

Inferring a more permissive role would allow to coerce instances dictionaries, which would threaten _coherence_: we could exhibit multiple type class instances with different behaviour for the same type.

```purescript
shown :: forall a. Shown a -> String
shown (Shown f) = f show

newtype HTML = MkHTML String
instance showHTML :: Show HTML where
  show (MkHTML s) = "(HTML " <> show s <> ")"

shownString :: Shown String
shownString = Shown (\f -> f "Hello")

shownHTML :: Shown HTML
shownHTML = Shown (\f -> f (MkHTML "Hello"))

badShownHTML :: Shown HTML
badShownHTML = coerce shownString
```

```
> :type shownHTML
Shown HTML

> shown shownHTML
"(HTML \"Hello\")"

> :type badShownHTML
Shown HTML

> shown badShownHTML
"\"Hello\""
```

* Representational roles are inferred for parameters appearing under at least one of the constructors of their type.

Representational parameters are coercible when a Coercible constraint holds.

```purescript
data Maybe a = Nothing | Just a
```

`Coercible (Maybe a) (Maybe b)` holds only when `Coercible a b` does.

This rule must be amended for parameters appearing in the arguments of a type variable. Because the variable could be instantiated to anything we have to be conservative and infer _nominal_, the most restrictive role, instead of _representational_: `Coercible (a -> f b) (a -> f c)` does not hold, even when `Coercible b c` does.

* Phantom roles are inferred for parameters not appearing at all under any constructor of their type.

Phantom parameters are coercible to anything:

```purescript
data Proxy a = Proxy
```

`Coercible (Proxy a) (Proxy b)` holds for all `a` and `b`, regardless of `Coercible a b`.

* Roles of parameters appearing in the arguments of another type are inferred from the declaration of that type:

```purescript
newtype First a = First (Maybe a)
```

`Coercible (First a) (First b)` holds when `Coercible a b` does, even when the newtype constructor is out of scope, because the parameter of `Maybe` is _representational_.

We cannot infer the role of parameters in recursive position this way, so we default to _phantom_ but usually end up with something else because we keep the most restrictive role a parameter appears at:

```purescript
data List a = Nil | Cons a (List a)
```

Here the parameter appears at _representational_ (under the `Cons` constructor) and _phantom_ (in recursive position) roles so we infer _representational_: `Coercible (List a) (List b)` holds when `Coercible a b` does.

```purescript
newtype Mu f = In (f (Mu f))
```

Here the parameter appears at _representational_ (under the `In` constructor) and _nominal_ (as argument to itself) roles so we infer _nominal_: `Coercible (Mu f) (Mu g)` does not hold, unless `g` is actually `f`.

## Role annotations

Inferring _nominal_ roles for foreign data types is safe but can be too constraining sometimes. For example this prevents to coerce `Effect Age` to `Effect Int`, even though they actually have the same runtime representation.

The roles of foreign data types can thus be loosened with explicit role annotations, similar to the [RoleAnnotations](https://downloads.haskell.org/ghc/latest/docs/html/users_guide/exts/roles.html#extension-RoleAnnotations) GHC extension.

Conversely, we might want to strengthen the roles of parameters with invariants invisible to the type system. Maps are the canonical example of this: the shape of their underlying tree rely on the `Ord` instance of their keys, but the `Ord` instance of a newtype may behave differently than the one of the wrapped type so it would be unsafe to allow coercions between `Map k1 a` and `Map k2 a`, even when `Coercible k1 k2` holds.

A role annotation starts with `type role`, then the name of the annotated type and the role (`nominal`, `representational` or `phantom`) of each parameters of the type. Role annotations are only allowed for data and newtype declarations. They have to immediately follow the annotated type declaration.

For example this role annotation relaxes the role inferred for the parameter of `Effect` (which would be _nominal_ otherwise):

```purescript
type role Effect representational
```

and this one strengthens the role inferred for the first parameter of `Map` (which would be _representational_ otherwise), leaving its second parameter _representational_:

```purescript
type role Map nominal representational
```

Annotated roles are compared against the roles inferred by the compiler so it is not possible to compromise safety by ascribing too permissive roles, except for foreign types.
