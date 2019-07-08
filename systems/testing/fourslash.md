# Fourslash

The fourslash tests are an integration level testing suite. By this point they have a very large API surface, and
tend to cover a lot of the "user-facing" aspects of TypeScript. E.g. things which an IDE might have an interest
in knowing.

### Formatting

[]

### How to run one

`gulp runtests` will run all the fourslash tests eventually. Or `gulp runtests -i --tests=[filename]` should speed 
things up if you only want to see those specific changes.

