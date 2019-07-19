# Text Changes

The majority of this file is devoted to a class called the [`ChangeTracker`][0]. This class is nearly always
created via `ChangeTracker.with` where you would give it a context object.

Here is an example context object:

```ts
{
  cancellationToken:CancellationTokenObject {cancellationToken: TestCancellationToken}
  errorCode:2304
  formatContext:Object {options: Object, getRule: }
  host:NativeLanguageServiceHost {cancellationToken: TestCancellationToken, settings: Object, sys: System, …}
  preferences:Object {}
  program:Object {getRootFileNames: , getSourceFile: , getSourceFileByPath: , …}
  sourceFile:SourceFileObject {pos: 0, end: 7, flags: 65536, …}
  span:Object {start: 0, length: 6}
}
```

You only really see `ChangeTrack` in use within the codefixes and refactors given that the other case where
TypeScript emits files is a single operation of emission.

The change tracker keeps track of individual changes to be applied to a file. There are [currently][1] four main
APIs that it works with:
`type Change = ReplaceWithSingleNode | ReplaceWithMultipleNodes | RemoveNode | ChangeText;`

The `ChangeTrack` class is then used to provide high level API to describe the sort of changes you might want to
make, which eventually fall into one of the four categories above.

### Writing

[`newFileChanges`][3] hadnles

### TextChangesWriter

[`createWriter`][2] is the production version of an object which the emit the changes held by

<!-- prettier-ignore-start -->
[0]: <src/services/textChanges.ts - export class ChangeTracker>
[1]: <src/services/textChanges.ts - type Change =>
[2]: <src/services/textChanges.ts - function createWriter>
[2]: <src/services/textChanges.ts - function newFileChanges>
<!-- prettier-ignore-end -->
