# Binder

The binder walks the tree visiting each declaration in the tree.
For each declaration that it finds, it adds a `Symbol` entry in a map called a `SymbolTable`.
The map is associated with a container that creates a scope, like a function, a block, or a module file.
`Symbol`s track the declaration, so that the checker can look up names and then check their types.
It also contains a small summary of what kind of declaration it is -- mainly whether it is a value, a type, or a namespace.
(Namespaces are the least common kind of declaration.)

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

## Walkthrough

```ts
function f(m: number) {
    type n = string
    const n = m + 1
    return m + n
}
```

The binder's basic tree walk starts in `bind`.
There, it first encounters `f` and calls `bindFunctionDeclaration` and then `bindBlockScopeDeclaration` with `SymbolFlags.Function`.
This function has special cases for files and modules, but the default case calls `declareSymbol` to add a symbol in the current container.
There is a lot of special-case code in `declareSymbol`, but the important path is to check whether the symbol table already contains a symbol with the name of the declaration -- `f` in this case.
If not, a new symbol is created.
If so, the old symbol's exclude flags are checked against the new symbol's flags.
If they conflict, the binder issues an error.

Finally, the new symbol's `flags` are added to the old symbol's `flags` (if any), and the new declaration is added to the symbol's `declarations` array.
In addition, if the new declaration is for a value, it is set as the symbol's `valueDeclaration`.

#### Containers

After `declareSymbol` is done, the `bind` visits the children of `f`; `f` is a container, so it calls `bindContainer` before `bindChildren`.
The binder is recursive, so it pushes `f` as the new container by copying it to a local variable before walking its children.
It pops `f` by copying the stored local back into `container`.

The binder tracks the current lexical container as a pair of variables `container` and `blockScopedContainer` (and `thisParentContainer` if you OOP by mistake).
It's implemented as a global variable managed by the binder walk, which pushes and pops containers as needed.
The container's symbol table is initialised lazily, by `bindBlockScopedDeclaration`, for example.

## Flags

The rules for symbol merging are complicated, but they're implemented in a surprisingly small space using bitflags.
The downside is that the bitflag system is very confusing.

The basic rule is that a declaration's flags may not conflict with the existing excludes flags.
And each kind of declaration maintains a list of declaration kinds that it may not merge with.

In the example above, `type n` is a type alias, so the binder uses the flag `SymbolFlags.TypeAlias` and excludeFlags `SymbolFlags.TypeAliasExcludes`.
The latter is an alias `SymbolFlags.Type`, which is a list of things not allowed to merge with type aliases:

```ts
Type = Class | Interface | Enum | EnumMember | TypeLiteral | TypeParameter | TypeAlias
```

Notice that these are all types, and includes `TypeAlias` itself.

Next, when the binder reaches `const n`, it uses the flag `BlockScopedVariable` and excludeFlags `BlockScopedVariableExcludes`.
`BlockScopedVariableExcludes = Value`, which is a list of every kind of value declaration.

```ts
Value = Variable | Property | EnumMember | ObjectLiteral | Function | Class | Enum | ValueModule | Method | GetAccessor | SetAccessor
```

`declareSymbol` looks up the existing excludeFlags for `n` and makes sure that `Value` doesn't conflict; `Value & Type === 0` so it doesn't.
Then it *or*s the new and old flags and the new and old excludeFlags.
In this example, that will prevent more value declarations because `Value & (Value | Type) !== 0`.

## Cross-file global merges

Because the binder only binds one file at a time, the above system for merges only works with single files.
For global (aka script) files, declarations can merge across files.
This happens in the checker in `initializeTypeChecker`, using `mergeSymbolTable`.

## Special names

`getDeclarationName` translates certain nodes into internal names.
`export=`, for example, gets translated to `InternalSymbolName.ExportEquals`

TODO: Finish this

## Control Flow

TODO: Missing completely

## Emit flags

TODO: Missing completely

## Exports

TODO: Missing completely

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
