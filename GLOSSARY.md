### Terminology from inside the codebase

- `Parser` - Takes a string and tries to convert it into an in-memory tree representation which you can work with
  in the compiler
- `Scanner` - A scanner takes a string an chops into tokens in a linear fashion, then it's up to a parser to
  tree-ify them
- `Token` - A set of characters with some kind of semantic meaning, a parser generates a set of tokens
- `AST` - An abstract syntax tree. Basically the in-memory representation of all the identifiers as a tree of
  tokens.
- `Node` - An object that lives inside the tree
- `Location`
- `Freshness` - When a literal type is first created and not expanded by hitting a mutable location, see [Widening
  and Narrowing in TypeScript][wnn].

### Type stuff which can be see outside the compilers

- `Structural Type System` - A school of types system where the way types are compared is via the structure of
  their properties.

  For example:

  ```ts
  interface Duck {
    hasBeak: boolean;
    flap: () => void;
  }

  interface Bird {
    hasBeak: boolean;
    flap: () => void;
  }
  ```

  These two are the exact same inside TypeScript. The basic rule for TypeScript’s structural type system is that
  `x` is compatible with `y` if `y` has at least the same members as `x`.

* `Literal` - A literal type is a type that only has a single value, e.g. `true`, `1`, `"abc"`, `undefined`.

  For immutable objects, TypeScript creates a literal type which is is the value. For mutable objects TypeScript
  uses the general type that the literal matches. See [#10676](https://github.com/Microsoft/TypeScript/pull/10676)
  for a longer explanation.

  ```ts
  // The types are the literal:
  const c1 = 1; // Type 1
  const c2 = c1; // Type 1
  const c3 = "abc"; // Type "abc"
  const c4 = true; // Type true
  const c5 = c4 ? 1 : "abc"; // Type 1 | "abc"

  // The types are the class of the literal
  let v1 = 1; // Type number
  let v2 = c2; // Type number
  let v3 = c3; // Type string
  let v4 = c4; // Type boolean
  let v5 = c5; // Type number | string
  ```

- `Generics` - A way to have variables inside a type system.

  ```ts
  function first(array: any[]): any {
    return array[0];
  }
  ```

  You want to be able to pass a variable type into this function, so you annotate the function with angle brackets and a
  _type parameter_:

  ```ts
  function first<T>(array: T[]): T {
    return array[0];
  }
  ```

  This means the return type of `first` is the same as the type of the array elements passed in. (These can start
  looking very complicated over time, but the principle is the same; it just looks more complicated because of the
  single letter.) Generic functions should always use their type parameters in more than one position (e.g. above,
  `T` is used both in the type of the `array` parameter and in the function’s return type). This is the heart of what
  makes generics useful—they can specify a _relationship_ between two types (e.g., a function’s output is the same as
  input, or a function’s two inputs are the same type). If a generic only uses its type parameter once, it doesn’t
  actually need to be generic at all, and indeed some linters will warn that it’s a _useless generic_.
  
  Type parameters can usually be inferred from function arguments when calling generics:
  
  ```ts
  first([1, 2, 3]); // 'T' is inferred as 'number'
  ```
  
  It’s also possible to specify them explicitly, but it’s preferable to let inference work when possible:
  
  ```ts
  first<string>(['a', 'b', 'c']);
  ```
  
* `Outer type parameter` - A type parameter declared in a parent generic construct:

  ```ts
  class Parent<T> {
    method<U>(x: T, y: U): U {
      // 'T' is an *outer* type parameter of 'method'
      // 'U' is a *local* type parameter of 'method'
    }
  }
  ```

* `Narrowing` - Taking a union of types and reducing it.

  A great case is when using `--strictNullCheck`

  ```ts
  // I have a dog here, or I don't
  declare const myDog: Dog | undefined;

  // Outside the if, myDog = Dog | undefined
  if (dog) {
    // Inside the if, myDog = Dog
    // because the type union was narrowed via the if statement
    dog.bark();
  }
  ```

- `Expanding` - The opposite of narrowing, taking a type and converting it to have more potential values.

```ts
const helloWorld = "Hello World"; // Type; "Hello World"

let onboardingMessage = helloWorld; // Type: string
```

When the `helloWorld` constant was re-used in a mutable variable `onboardingMessage` the type which was set is an
expanded version of `"Hello World"` which went from one value ever, to any known string.

- `Transient` - unsure

- `Partial Type` -
- `Synthetic`
- `Union Types`
- `Enum`
- `Discriminant`
- `Intersection`
- `Indexed Type` - A way to access subsets of your existing types.

```ts
interface User {
  profile: {
    name: string;
    email: string;
    bio: string;
  };
  account: {
    id: string;
    signedUpForMailingList: boolean;
  };
}

type UserProfile = User["profile"]; // { name: string, email: string, bio: string }
type UserAccount = User["account"]; // { id: string, signedUpForMailingList: string }
```

This makes it easier to keep a single source of truth in your types.

- `Index Signatures` - A way to tell TypeScript that you might not know the keys, but you know the type of values
  of an object.

  ```ts
  interface MySettings {
    [index: string]: boolean;
  }

  declare function getSettings(): MySettings;
  const settings = getSettings();
  const shouldAutoRotate = settings.allowRotation; // boolean
  ```

- `IndexedAccess` - ( https://github.com/Microsoft/TypeScript/pull/30769 )
- `Conditional`
- `Substitution`
- `NonPrimitive`
- `Instantiable`
- `Tuple` - A mathematical term for a finite ordered list. Like an array but with a known length.

TypeScript lets you use these as convenient containers with known types.

```ts
// Any item has to say what it is, and whether it is done
type TodoListItem = [string, boolean];

const chores: TodoListItem[] = [["read a book", true], ["done dishes", true], ["take the dog out", false]];
```

Yes, you could use an object for each item in this example, but tuples are there when it fits your needs.

- `Mapped Type` - A type which works by taking an existing type and creating a new version with modifications.

```ts
type Readonly<T> = { readonly [P in keyof T]: T[P] };

// Map for every key in T to be a readonly version of it
// e.g.
interface Dog {
  furColor: string;
  hasCollar: boolean;
}

// Using this
type ReadOnlyDog = Readonly<Dog>;

// Would be
interface ReadonlyDog {
  readonly furColor: string;
  readonly hasCollar: boolean;
}
```

This can work where you

- `Type Assertion` - override its inferred and analyzed view of a type

```ts
interface Foo {
    bar: number;
    bas: string;
}
var foo = {} as Foo;
foo.bar = 123;
foo.bas = 'hello';
```

### Rarely heard

- `Deferred`
- `Homomorphic`

### JS Internals Specifics

[`Statement`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements) - "JavaScript
applications consist of statements with an appropriate syntax. A single statement may span multiple lines.
Multiple statements may occur on a single line if each statement is separated by a semicolon. This isn't a
keyword, but a group of keywords."

[wnn]: https://github.com/sandersn/manual/blob/master/Widening-and-Narrowing-in-Typescript.md
