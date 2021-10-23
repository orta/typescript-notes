# Binder

The binder walks the tree visiting each declaration in the tree.
For each declaration that it finds, it adds a `Symbol` entry in a map called a `SymbolTable`.
The map is associated with a container that creates a scope, like a function, a block, or a module file.
`Symbol`s track the declaration, so that the checker can look up names and then check their types.
It also contains a small summary of what kind of declaration it is -- mainly whether it is a value, a type, or a namespace.
(Namespaces are the least common kind of declaration.)

## Details

For example,

```ts
function f(n: number) {
    const m = n + 1
    return m + n
}
```

Creates a symbol table for `f` that contains two entries: `n` and `m`.
The binder finds `n` while walking the function's parameter list, and it finds `m` while walking the block that makes up `f`'s body.

Both `n` and `m` are marked as values.
However, there's no problem with adding another declaration for `n`:

```ts
function f(n: number) {
    type n = string
    const m = n + 1
    return m + n
}
```

Now `n` has two declarations, one type and one value.
The binder disallows more than one declaration of a kind of symbols with *block-scoped* declaration.
Examples are `type`, `function`, `class`, `let`, `const` and parameters; *function-scoped* declarations include `var` and `interface`.
But as long as the declarations are of different kinds, they're fine.
The binder tracks this using the same bitflag that tracks type and value status.

TODO: This is unfinished.

## Special names

`export=`, `export default`, etc.

## Intra-file merges

Just save all the declarations and let the checker aggregate all the members later.

## Cross-file global merges

They happen in the checker.

## Javascript and CommonJS

Javascript has additional types of declarations that it recognises, which fall into 3 main categories:

1. Constructor functions and pre-class-field classes: assignments to `this.x` properties.
2. CommonJS: assignments to `module.exports`.
3. Global browser code: assignments to namespace-like object literals.
4. JSDoc declarations: tags like `@type` and `@callback`.

Four! Four main categories!

A lot of these declarations have close Typescript equivalents.
However, they are function-scoped instead of block-scoped &mdash; they allow multiple declarations, which can conflict.
And, since they're mostly assignments, there are a few more ways that they can be used.

TODO: This is unfinished.

### Conflicting object literal export assignments

One particuarly complex case of CommonJS binding occurs when there is an object literal export assignment in the same module as `module.exports` assignments:

```js
module.exports = {
    foo: function() { return 1 },
    bar: function() { return 'bar' },
    baz: 12,
}
if (windows) {
    module.exports.foo = function () { return 11 }
}
```

In this case, the desired exports of the file are `foo, bar, baz`.
Even though `foo` is declared twice, it should have one export with two declarations.
The type should be `() => number`, though that's the responsibility of the checker.

In fact, this structure is too complicated to build in the binder, so the checker produces it through merges, using the same merge infrastructure it uses for cross-file global merges.
However, the code is in a different place: `XXX`.

TODO: Describe looking up a JS export= and merging its contents back into the module.
Then subsequent code has to ignore the JS export=.