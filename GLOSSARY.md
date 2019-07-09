### Terminology from inside the codebase

`Parser` - Takes a string and tries to convert it into an in-memory representation which you can work with in the compiler
`Token` - A set of characters with some kind of semantic meaning, a parser generates a set of tokens 
`AST` - An abstract syntax tree. Basically the in-memory representation of all the identifiers as a tree of tokens.
`Node` - An object that lives inside the tree

### JS Internals Specifics

[`Statement`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements) - "JavaScript applications 
consist of statements with an appropriate syntax. A single statement may span multiple lines. Multiple statements 
may occur on a single line if each statement is separated by a semicolon. This isn't a keyword, but a group of keywords."


