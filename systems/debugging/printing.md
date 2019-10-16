## For `console.log` debugging

A lot of the time to see what the code you're representing you want to look in either `declaration` or
`declarations`. These are Node subclasses which means they'll have a copy of `__debugGetText()` which should get
you to the syntax representation.

### `Type`

Useful: `console.log(type.__debugKind)`.

Chances are through you want to go through symbol

### `Symbol`

Using `symbolToString(symbol)` could get you somewhere, but so far that's not been

### `Node`

### `Signature`

`signature.declaration.__debugGetText()`
