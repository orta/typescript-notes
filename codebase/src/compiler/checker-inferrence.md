# Type Inference

Typescript has a number of related techniques which are usually
called type inference: places where a type is discovered from
inspecting values instead of reading a type annotation. This document
covers them all in one place, aimed at teaching contributors to the
TypeScript compiler.

None of these techniques are Hindly-Milner type inference. Instead,
Typescript adds a few ad-hoc techniques to its normal type-checking.
The result is a system that can infer from many useful locations, but
nowhere near all of them.

An important concept that's true of all three techniques: type
inference is a separate step that happens before checking. The checker
will infer a type for a location; then it will check the type in the
normal way, as if the type had been explicitly written. This results
in redundant checking when the type inference is simple.

## Initialiser inference

The simplest kind of inference is from initialisers. This inference is
so simple that I don't believe it has been given a separate name until
now.

You can see this anywhere a variable, parameter or property has an
initialiser:

```ts
let x = 123
function f(x = 123) {
}
class C {
  x = 123
}
```

Remember, inference precedes checking, so checking `let x = 123`
looks like this:

1. Look for the type of `x`.
2. There is no annotation, so use the (widened) type of the initialiser: `number`.
3. Check that the initialiser's type `123` is assignable to `number`.

## Contextual typing

Contextual typing reverses the downward direction of the checker's
tree walk: it looks upward in the tree for a type. Contextual typing
only applies to two kinds of things: parameters and literals; but it
may walk up a significant chunk of an expression.

Here are 3 typical places that contextual typing finds a type:

1. A type annotation on a declaration:

``` ts
type Config = { before(data: string): void }
const cfg: Config = {
  before(data) {
    console.log(data.length)
  }
}
```

2. Assignment:

``` ts
let steps: ('up' | 'down' | 'left' | 'right')[] = ['up', 'up', 'down', 'down']
steps = ['down']
```

Contextual typing is important here because it gives 'down' the
*non-widening* type `'down'`, not `string`. That means `['down']` will
have the type `'down'[]`, which is assignable to `steps`.

3. An argument in a function call:

``` ts
declare function setup(register: (name: string, age: number) => void): void
setup((name, age) => console.log(name, age))
```

As you can see, the basic mechanism of contextual typing is a search
for a type annotation, even if it's non-local. Once a type annotation
is found, contextual typing drills down through the type based on the
path that it just walked upward.

### Walkthrough

Let's walk through examples (1) and (3).

1. checkFunctionExpressionOrObjectLiteralMethod is called on `before`.
2. After a few calls, we reach getApparentTypeofContextualType, which
   recursively looks for the contextual type of `before`'s parent.
3. This is an object literal, so we recursively look for the
   contextual type of *its* parent.
4. This is a variable declaration with a type annotation `Config`.
   This is the contextual type of the object literal.
5. Next we request `before` from `Config` and make sure it's a
   signature. The signature is the contextual type of `before`.
6. Finally, `assignContextualParameterTypes` assigns types to
   `before`'s parameters.

Note that if you have type annotations on some parameters already,
the unannotated parameters still get a contextual type.

Contextually typing `(name, age) => ...` in (3) works substantially
that same. When the search reaches
`getContextualType`, instead of a variable declaration, the parent is
a call expression. So we have to find the argument index of `(name,
age) => ...`, 0, and look up the matching parameter of `setup`, which
is `register`. `register`'s type is now the contextual type, and
`assignmentContextualParameterTypes` works as in (1).

## Type Parameter Inference

Type parameter inference is quite different from the other two
techniques. It still infers **types** based on provided **values**,
but the inferred types don't apply immediately to values. Instead
they're provided as type arguments to a function, which results in
instantiating a generic function with some specific type. For example:

``` ts
declare function setup<T>(config: { initial(): T }): T
setup({ initial() { return "last" } })
```

First checks `{ initial() { return "last" } }` to get
`{ initial(): string }`. By looking for `T` in `{ initial(): T }`, it
infers that `T` is `string`, making the second line equivalent to

``` ts
setup<string>({ initial() { return "last" } })
```

Meaning that the compiler then checks that
`{ initial() { return "last" } }` is assignable to
`{ initial(): string }`.

### Walkthrough

Also unlike the previous two type inferences, the first step is to get
the type of the argument. The argument's type is called the source
type, and will be the **source** of any new inferences. The
parameter's type is the **target** type. Type parameter inference is a
type-walking search for type parameters in the target type, to be
paired with the matching type in the source type. The type is walked
structurally a lot like a tree is elsewhere in the compiler.

1. `inferTypeArguments` checks the type of each argument to the
   function call, then pairs it with the parameter type of the
   function whose type arguments are being inferred.
2. `inferTypes` gets called on each argument/parameter pair with
   argument=source/parameter=target. There's only one pair here:
   `{ initial(): string }` and `{ initial(): T }`.
3. Since both sides are object types, `inferFromProperties` looks
   through each property of the target and looks for a match in the
   source. In this case both have the property `initial`.
4. `initial`'s type is also an object, but this time inference recurs
   through `inferFromSignature`.
5. This recursively infers from the return type.
6. Finally, the source is the type parameter `T`. This adds `string`
   to the list of inferences for `T`.

Once all the parameters have had `inferTypes` called on them,
`getInferredTypes` condenses each candidate array to a single type.
The most important thing here is that inference to return types,
`keyof T` and mapped type constraints (which are usually `keyof` too)
produce a union, whereas all other places produce a common supertype. (Object
types are always unioned together first, regardless of position)

### Other considerations

#### Interference Between Contextual Typing and Type Parameter Inference

Type parameter inference operates in two passes. The first pass skips
contextually typed expressions so that if good inferences are found
from other arguments, then good contextual types can be found. Then
the second pass proceeds with (hopefully) better types.

#### Inference Priorities

Different positions have different inference priorities; when the type
walk finds a candidate at a higher priority position than existing
candidates, it throws away the existing candidates and starts over
with the higher-priority candidate. For example, a lone type variable
has the highest priority, but a type variable found in a return type
has one of the lowest priorities.

### Contravariant Candidates

Certain candidates are inferred contravariantly, such as parameters of
callbacks. This is a separate system from inference priorities;
contravariant candidates are even higher priority.

#### Reverse Mapped Types

A reverse mapped type is a mapped type that is constructed during
inference, and it requires information obtained from inference, but is
not a central part of inference. A reverse mapped type is constructed when
the target is a mapped type and the source is an object type. It
allows a inference to apply to every member of an object type:

``` ts
type Box<T> = { ref: T }
type Boxed<T> = { [K in keyof T]: Box<T[K]> }
declare function unbox<T>(boxed: Boxed<T>): T;
unbox({ a: { ref: 1 }, m: { ref: "1" } }) // returns { a: number, m: string }
```

  <!-- prettier-ignore-start -->

[0]: <src/compiler/checker.ts - function inferTypes(>
[1]: https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-6.html#strict-function-types

  <!-- prettier-ignore-end -->
