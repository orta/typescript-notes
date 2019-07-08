# Checker

Ok, yeah, so it's a 30k LOC file. So better to get started somewhere. 

### An entry-point

[0]: <src/compiler/program.ts - function getDiagnosticsProducingTypeChecker() {>
[1]: <src/compiler/checker.ts - function getDiagnosticsWorker(sourceFile: SourceFile): Diagnostic[] {>
[2]: <src/compiler/checker.ts - function checkSourceFileWorker(node: SourceFile) {>
[3]: </src/compiler/types.ts - export interface NodeLinks>

The likely entry point is via a Program. The program has a memoized typechecker created 
in [`getDiagnosticsProducingTypeChecker`][0] which creates a type checker.

The initial start of type checking starts with [`getDiagnosticsWorker`][1], worker in this case isn't a threading term
I believe ( at least I can't find anything like that in the code ) - it is set up to listen for diagnostic results 
(e.g. warns/fails) and then triggers [`checkSourceFileWorker`][2]. 

This function starts at the root `Node` of any TS/JS file  node tree: `SourceFile`.

# Checking a Node



### Caching Data

Note there are two ways in which TypeScript is used, as a server and as a one-off compiler. In a server, we want to
re-use as much as possible between API requests, and so the Node tree is treated as immutable data. This gets tricky
inside the Type Checker, which for speed reasons needs to cache data somewhere. The solution to this \
is the [`NodeLinks`][3] property on a Node. 

The Type Checker fills this up during the run and re-uses it, then it is discarded.
