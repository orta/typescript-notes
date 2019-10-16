### Getting started

- Clone the repo: `git clone https://github.com/microsoft/TypeScript`
- `cd TypeScript`
- `npm i`
- `gulp`
- `code .` then briefly come back to terminal to run
- `gulp runtests-parallel`

OK, that should have a copy of the compiler up and running and then you have tests going in the background to
prove that everything is working fine. This could take maybe 10m? There'll be a progress indicator.

### Getting set up

To get your VS Code env working smoothly, set up the per-user config

- Set up your `.vscode/launch.json` by running: `cp ./vscode/launch.template.json ./vscode/launch.json`
- Set up your `.vscode/settings.json` by running: `cp ./vscode/settings.template.json ./vscode/settings.json`

In the `launch.json` I duplicate the configuration, and change `"${fileBasenameNoExtension}",` to be whatever test
file I am currently working on.

### Learn the debugger

You'll probably spend a good amount of time in it, if this is completely new to you. Here is a video from
[@alloy](https://github.com/alloy) covering all of the usage of the debugger inside VS Code and how it works.

To test it out, open up `src/compiler/checker.ts` find `function checkSourceFileWorker(node: SourceFile) {` and
add a debugger on the first line in the code. If you open up `tests/cases/fourslash/getDeclarationDiagnostics.ts`
and then run your new debugging launch task `Mocha Tests (currently opened test)`. It will switch into the
debugger and hit a breakpoint in your TypeScript.

You can learn about the different ways to [write a test here](./systems/testing) to try and re-produce a bug.
