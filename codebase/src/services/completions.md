## Completions in TS

Completions come in through the language service, and most of the work starts in
[`getCompletionEntriesFromSymbols`](https://github.com/Microsoft/TypeScript/blob/340f81035ff1d753e6a1f0fedc2323d169c86cc6/src/services/completions.ts#L305).

Step one is to find a `contextToken`. This is generally the preceding token to your cursor, as that tends to be
the most important thing when deciding what to show next. A token

Next it goes through the following checks for a set of completions.

#### String Literal Completions

E.g. are you inside a string and asking for completions? TS differentiates between reference comments
([triple slash](https://www.typescriptlang.org/docs/handbook/triple-slash-directives.html)):

![./screenshots/threeslash-refs.png](./screenshots/threeslash-refs.png)

And strings as a part of the AST. These have a
[few](https://github.com/Microsoft/TypeScript/blob/340f81035ff1d753e6a1f0fedc2323d169c86cc6/src/services/stringCompletions.ts#L103)
different uses:

- They could be path references
- They could be module references
- They could be indexed keys from an object
- They could be parts of a union object
- etc

####
