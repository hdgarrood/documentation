<!--markdownlint-disable MD046 -->
# Getting Started with PureScript

Let's walk through the basics of getting set up to use the PureScript compiler `purs`, and its interactive mode `purs repl`.

We'll start with the installation of the compiler and Spago build tool, and then go through the basic usage of `purs repl`, working towards a solution of problem 1 from [Project Euler](http://projecteuler.net/problem=1).

## Installing the Compiler

There are many [installation options](https://github.com/purescript/purescript/blob/master/INSTALL.md), but we recommend using `npm`.

[Install Node.js and npm via a node version manager](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm#using-a-node-version-manager-to-install-nodejs-and-npm). Using a node version manager helps avoid [`EACCES` errors](https://docs.npmjs.com/getting-started/fixing-npm-permissions) when installing packages globally (as we'll do in the next step).

Install the Purescript compiler (`purs`) with npm:

    npm install -g purescript

Try running the PureScript compiler on the command line to verify that the PureScript compiler executables are available on your `$PATH`:

    purs

If you encounter problems during installation, see the compiler's [Install Guide](https://github.com/purescript/purescript/blob/master/INSTALL.md) for troubleshooting suggestions.

## Setting up the Development Environment

[Spago](https://github.com/spacchetti/spago) is the recommended package manager and build tool for PureScript.

If you don't have Spago installed, install it now:

    npm install -g spago

Create a new project in an empty directory using `spago init`:

    mkdir my-project
    cd my-project
    spago init

Your directory should now contain the following files:

- `spago.yaml` - contains Spago configuration and library dependency information
- `src/Main.purs` - Entry point module for your project
- `test/Main.purs` - An empty test suite

At this point, you should be able to build the project and run the tests:

    spago build
    spago test

You should see output similar to the following:

    [...]
    Building...
               Src   Lib   All
    Warnings     0     0     0
    Errors       0     0     0

    ✅ Build succeeded.

    Running tests for package: purescript-pasta
    🍕
    You should add some tests.

    ✅ Test succeeded for package "purescript-pasta".

If everything was built successfully, and the tests ran without problems, then the last line should state "Tests succeeded."

## Installing Dependencies

Dependencies can be installed using Spago. We will be using the `lists` and `foldable-traversable` libraries shortly, so install those now:

    spago install lists foldable-traversable

The `lists` and `foldable-traversable` library sources should now be available in the `.spago/p/lists-{version}/` and `.spago/p/foldable-traversable-{version}/` subdirectories respectively, and will be included when you compile your project.

## Working in PSCI

PSCi is the interactive mode of PureScript. It is useful for working with pure computations, and for testing ideas.

Open PSCi by typing `spago repl` at the command line. Optionally, you can create a file in your directory called `.purs-repl`, which contains instructions to PSCi to load your modules and dependencies. If you invoke the PSCi executable directly, you would need to load these files by hand.

    PSCi, version 0.12.0
    Type :? for help

    import Prelude

    >

As the introduction indicates, you can type `:?` to see a list of commands:

    The following commands are available:

    :?                        Show this help menu
    :quit                     Quit PSCi
    :reload                   Reload all imported modules while discarding bindings
    :clear                    Discard all imported modules and declared bindings
    :browse      <module>     See all functions in <module>
    :type        <expr>       Show the type of <expr>
    :kind        <type>       Show the kind of <type>
    :show        import       Show all imported modules
    :show        loaded       Show all loaded modules
    :show        print        Show the repl's current printing function
    :paste       paste        Enter multiple lines, terminated by ^D
    :complete    <prefix>     Show completions for <prefix> as if pressing tab
    :print       <fn>         Set the repl's printing function to <fn> (which must be fully qualified)

    Further information is available on the PureScript documentation repository:
    --> https://github.com/purescript/documentation/blob/master/guides/PSCi.md

We will use a selection of these commands during this tutorial.

Start by pressing the Tab key to use the autocompletion feature. You will see a collection of names of functions from the Prelude which are available to use.

To see the type of one of these values, first import the appropriate module using the `import` command.

Next, use the `:type` command, followed by a space, followed by the name of the value:

    > import Prelude
    > :type map
    forall (@f :: Type -> Type) (a :: Type) (b :: Type). Functor f => (a -> b) -> f a -> f b

    > import Data.List
    > :type zip
    forall (a :: Type) (b :: Type). List a -> List b -> List (Tuple a b)

We will be using some of the functions from the `Prelude` and `Data.List` modules, so make sure you have imported those by using the `import` keyword:

    import Prelude
    import Data.List

Note that using `Tab` to autocomplete names can be a useful time-saving device in `psci`.

## Solving Project Euler #1

The following problem is taken from [Project Euler](http://projecteuler.net/problem=1):

> If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23.
>
> Find the sum of all the multiples of 3 or 5 below 1000.

We can solve this problem neatly using functions and function composition, directly in the REPL.

Let's start by listing all of the natural numbers below 1000 as a list. We can do this using the `range` function from `Data.List`:

    > range 0 999

You should see a list with 1000 elements printed to the command line.

This value can be given a name:

    > ns = range 0 999

Now let's filter out all of those elements which do not meet the criterion. We can use the `filter` function from `Data.List`, by providing a predicate function as its first argument:

    > multiples = filter (\n -> mod n 3 == 0 || mod n 5 == 0) ns

You can see the result by evaluating `multiples` if you like, or even check its type:

    > multiples
    (0 : 3 : 5 : 6 : ...
    > :type multiples
    List Int

Now we need to find the sum of the `multiples` list, to complete the solution. We can use the `sum` function from the `Data.Foldable` module.

    > import Data.Foldable
    > sum multiples
    233168

When you have finished using PSCi, type `:quit` to quit:

    > :quit
    See ya!

## Compiling a Solution

Now that we've seen how to use the REPL to reach the answer, let's move our solution into a source file and compile it.

Create a new text file `src/Euler.purs` and copy the following code:

```purescript
module Euler where

import Prelude

import Data.List (range, filter)
import Data.Foldable (sum)

ns = range 0 999

multiples = filter (\n -> mod n 3 == 0 || mod n 5 == 0) ns

answer = sum multiples
```

This sample illustrates a few key ideas regarding modules:

- Every file begins with a module header. A module name consists of one or more capitalized words separated by dots. In this case, only a single word is used, but `My.First.Module` would be an equally valid module name.
- Modules are imported using their full names, including dots to separate the parts of the module name. Here, we import the `Prelude` module, which provides `mod`, `==`, and many other common functions. We also import `Data.List` which provides the explicitly-listed `range` and `filter` functions. We can either import all functions in a module implicitly, as is done with `Prelude`, or list them explicitly. Guidelines are to only have one module with implicit imports.

It is possible to load this file directly into the REPL and to continue working:

    spago repl
    > import Euler
    > answer
    233168
    > :quit
    See ya!

Alternatively, we can use Spago to compile our new module to JavaScript:

    spago build

This will compile each module present in `src/` into a separate file in the `output/` directory.

The compiler will display several warnings about missing type declarations. In general it is considered good practice to provide explicit type signatures. In this guide, they are left out for brevity. In the absence of type signatures, the PureScript compiler infers types automatically but will remind us to consider adding them.

## Writing a Test Suite

To test our code, we'll use the `assert` library:

    spago install assert

Replace the contents of the `test/Main.purs` file with the following code:

```purescript
module Test.Main where

import Prelude

import Euler (answer)
import Test.Assert (assert)

main = do
  assert (answer == 233168)
```

Our "test suite" is just a single assertion that the `answer` value equals the correct integer. In a real test suite, we might use the `Effect` monad to compose multiple tests in our `main` function.

Run the tests using `spago test`, and you should hopefully see "Tests succeeded." in the last line.

## Creating Executables

We can modify the `main` function in the `src/Main.purs` module to print our result to the console:

```purescript
module Main where

import Prelude

import Euler (answer)
import Effect.Console (log)

main = do
  log ("The answer is " <> show answer)
```

The `spago run` command can be used to compile and run the `Main` module:

    $ spago run
    [info] Build succeeded.
    The answer is 233168

## Compiling for the Browser

Spago can be used to turn our PureScript code into JavaScript suitable for use in the web browser by using the `spago bundle` command:

    $ spago bundle
    [...]

      index.js  11.8kb

    ⚡ Done in 14ms
    ✅ Bundle succeeded.

All the code in the `src` directory and any project dependencies have been compiled to JavaScript. The resulting code is bundled as `index.js` and has also had any unused code removed, a process known as dead code elimination. This `index.js` file can now be included in an HTML document.

Try creating an `index.html` file with the following contents:

```html
<!DOCTYPE html>
<html>

  <head>
    <meta charset="UTF-8">
    <title>Euler Exercise</title>
  </head>

  <body>
    <script src="./index.js"></script>
  </body>

</html>
```

Open this `index.html` file in your web browser. The page will be blank, but if you open the browser's developer console, you should see a log message containing: "The answer is 233168".

If you open `index.js`, you should see a few compiled modules which look like this:

```javascript
(() => {
  // output/Data.Show/foreign.js
  var showIntImpl = function(n) {
    return n.toString();
  };

  [...]

  // output/Euler/index.js
  var ns = /* @__PURE__ */ function() {
    return range2(0)(999);
  }();
  var multiples = /* @__PURE__ */ function() {
    return filter(function(n) {
      return mod(euclideanRingInt)(n)(3) === 0 || mod(euclideanRingInt)(n)(5) === 0;
    })(ns);
  }();
  var answer = /* @__PURE__ */ function() {
    return sum(foldableList)(semiringInt)(multiples);
  }();

  // output/Main/index.js
  var main = /* @__PURE__ */ function() {
    return log("The answer is " + show(showInt)(answer));
  }();

  // <stdin>
  main();
})();
```

This illustrates a few points about the way the PureScript compiler generates JavaScript code:

- Every module gets turned into an object, created by a wrapper function, which contains the module's exported members.
- PureScript tries to preserve the names of variables wherever possible.
- Function applications in PureScript get turned into function applications in JavaScript.
- The main method is run after all modules have been defined and is generated as a simple method call with no arguments.
- PureScript code does not rely on any runtime libraries. All of the code that is generated by the compiler originated in a PureScript module somewhere which your code depended on.

These points are important since they mean that PureScript generates simple, understandable code. The code generation process, in general, is quite a shallow transformation. It takes relatively little understanding of the language to predict what JavaScript code will be generated for a particular input.

## Compiling ES Modules

Spago can also be used to generate ES modules from PureScript code. This can be useful when using NodeJS, or just when developing a larger project which uses ES modules to break code into smaller components.

To build ES modules, use the `spago build` command:

    $ spago build
    [...]
    ✅ Build succeeded.

The generated modules will be placed in the `output` directory by default. Each PureScript module will be compiled to its own ES module, in its own subdirectory.

### What Next?

If you're new to typed functional programming, check out the [PureScript by Example](https://book.purescript.org/) book. This book walks you through learning PureScript by solving practical problems.

If you are already familiar with an [ML-family language](https://en.wikipedia.org/wiki/ML_(programming_language)), the [PureScript by Example](https://book.purescript.org/) book should still be a good starting point.

If you are somewhat familiar with typed functional programming, You may alternatively browse the [language reference in the documentation repository](https://github.com/purescript/documentation/tree/master/language). It gives a brief, reference-style description of the language.

Haskell programmers will find the [Differences from Haskell](https://github.com/purescript/documentation/blob/master/language/Differences-from-Haskell.md) page very useful.

We encourage new PureScript programmers to spend some time browsing [Pursuit](https://pursuit.purescript.org). It is worth familiarising yourself with the [core libraries](https://github.com/purescript) (especially the [prelude](https://pursuit.purescript.org/packages/purescript-prelude)) as these provide many basic concepts, which are useful for writing programs.
