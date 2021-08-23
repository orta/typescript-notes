# Binder

The binder walks the syntax tree looking for declarations. 
For each declaration that it finds, it creates a `Symbol` that records its location and kind of declaration.
Then it stores that symbol in the containing node for the current scope.
For example:

```ts
// @Filename: main.ts
var x = 1
console.log(x)
```

The only declaration in this program is `var x`, which is contained in the SourceFile node for `main.ts`.
Functions and classes introduce new scopes, so they are containers -- at the same time as being declarations themselves.

Since the binder is the first tree walk before checking, it also does some other tasks: setting up the control flow graph,
as well as annotating parts of the tree that will need to be downlevelled for old ES targets.

## Flags

The binder tracks the kind of declaration with `SymbolFlags`, a bitflag enum.

1. The first step is pretty simple: choose what kind of declaration you have: function, class, enum member, etc.
2. Second, find the matching *excludes* mask. This specifies the set of things that do not merge with the declaration.

For example, in the following program:

```ts
var f = 1
function f() { }
```

When the binder visits `function f`, it creates a symbol with `SymbolFlags.Function` and an excludes mask of `SymbolFlags.FunctionExcludes`.
`FunctionExcludes = Value & ~(Class | ValueModule | Function)`, which means that functions may not merge with values, except that they may merge with classes, namespaces and other functions.
Because `var f` is already bound in the file, and because `Variable` is part of the `Value` mask, the binder will log a duplicate identifier error on both declarations of `f`.

(Namespaces still often use the name 'module' internally, and namespaces containing values are distinct from namespaces that only contain types, which are not values.)