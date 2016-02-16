# Simplescript

Simplescript is a strict subset of Typescript, which itself is a superset of Javascript with optional types. The goal is to provide a surprise-free language by avoiding subtley tricky areas. Because Javascript has an explicit goal of being backwards-compatible, many of the language decisions cannot be undone with Javascript in the wild. However, with transpilers, it is possible to avoid them and output un-surprising Javascript.

## Prior Art
- ESLint
  - xo
- JSCS
- TSLint
- JS standard

## Value proposition
- Make it into a well-integrated VS Code plug-in that generates auto-suggestions that you can approve / choose selection / ignore
- Focus on providing actionable recommendations in a command-line interface

## Design Goals

- Removes confusing edges of the Javascript language without losing its expressiveness
- Embrace Javascript idioms but make them explicit if it clarifies the intent
- Autofix when possible, otherwise make an error

## Architecture
- Creates a file simple.json that defines the rule
- Create a clear upgrade path when adding additional rules
- Simple CLI
  - Watch mode
  - Check before fixing mode
  - Interactive fix mode for warnings
  - Clear error messages / suggestions / references
  - Upgrade path

## General Utility functions

- [A]: Auto-suggest creating a `.gitignore` file if there isn't one already
  - Use a standard github one (e.g. `node_modules`, `bower-components`)
- [A]: Auto-creates a shrinkwrap dependency
  - proxy `npm install` and `npm shrinkwrap` calls
  - creates a `npm-shrinkwrap.clean.json`
- [A]: Renaming `file.js` triggers a search of all potential places that `import any from "file.js"`
- [A]: Creates `.gitkeep` file in folders with no file, that are not gitignored

## Specific Areas of Improvement

**[A]: Autofix**
**[W]: Warnings (will always compile) **
**[F]: Feature**

- [W]: `const obj = {}`, if you do `obj.hi = "hello"` throw a warning
  - Surprising behavior for those new to ES6 `const`
- [A]: Automatically include `()` for 0 or 2+ parameters when using lambda function `=>`
- [A]: `func.apply(null, paramArgs)` transpiles to `func(...paramArgs)`
- [W]: emit warning if using args without using spread operator
  - `fn(){args = [].slice.call(args)}`
- [W]: Don't use `Object.create(null)` for a hash table, use `new Map()` or `new WeakMap()` instead
- [A]: Either use all `tabs` or all `spaces`
- [W]: indentation and spacing (use JSCS or clang-format)
- [W]: Character limit per line
- [W]: Un-closed `(` or `{` or `[`
- [W]: Extraenous `)` or `}` or `]`
- [W]: Don't commit `ddescribe`, `iit`, `describe.only`, `it.only`, `fdescribe`, `fit`
- [W]: Do not nest `it` blocks for testing (common testing problem)
- [W]: Do not use `it` without being nested in a `describe` block (common testing problem)
   - Consider other frameworks...
- [A]: automatically insert semicolons
  - Improves clarity for people from other languages
- [A]: `NaN === nanVar` transpiles into `isNaN(nanVar)`
- [A]: `let`: always use let (can disable); `var` functional scoping is confusing
- [A]: Default to named function expression to improve stack trace:
  - `let printHi = () => return "hi"` -> `let printHi = function printHi(){return "hi"};`
- [W]: Always require `done` and `next` to be invoked if they are function parameters because their idiomatic use is to signify the completion of async work
- [F]: "use strict" by default
  - TODO: research the benefits of "use strict"
- [W]: No implicit global variable
  - `hello = "hi"` results in `Error: implicit global variable`
  - Must use `window.hello = "hi"`
- [W]: Private variables must be prefixed with _
  - `person._secretId`
- [W]: All uppercase variable names must be constants based on idiomatic JS usage
  - `let TOKEN_ID = "123"` results in `Error:uppercase variable name should be constant`
- [W]: Do not allow re-writing an object / class property in the same class
  - Rationale: confusing usage, usually signifies a mistake
- [A]: Transpile `!!isFlagEnabled` into `Boolean(isFlagEnabled)`
  - Make a common JS idiom more explicit for those less familiar with JS truthiness rules
- [A]: Transpile `if (x ==|!= undefined|null)` into `if (x === undefined || x === null)`
  - Make a common JS idiom more explicit for those less familiar with JS truthiness rules
- [A]: Use idiomatic JS way of checking if something exists
  - Transpiles `if (x == true)` into `if (x)`
- [W]: Cannot redefine undefined (look up Coffeescript)
  - Very confusing when you do comparison checks against undefined in the future
- [W]: Do not name parameters `e` as shorthand for `error` and `event`; instead be explicit with the parameter name `err`, `error`, `event`
- [A]: Explicitly transpile object property names into strings
  - `{hi: "hello"}` into `{"hi": "hello"}`
- [W]: Use `snake_case` file convention instead of capitalizing (while OS X is not case sensitive, other OS are)
  - `bubble_module.js` instead of `AbstractClass.js`
  - Make the file-ending convention pluggable: (e.g. applicable to `.js`, `.ts`, `.coffee`)
- [W]: For folders with only `.js` files follow the above convention
- [W]: Do not have more than two line breaks consecutively
   - Comments between line breaks resets the count of contiguous line breaks
- [A]: Include an extra line at bottom of file
  - Reduces merge conflicts / diffing
  - Old systems will have issues with concating if there's no new line
- [W]: Capitalized (pascal case (?)) variable names are only for Classes
  - If you modify the `prototype` of the function name, then treat as a class
  - Warning for: `function GenerateData` results in "Capitalized case"
  - Warning for: `class cat`
- [A]: Alway include parenthesis in `new` call to make it more explicit that Class is a function being invoked
  - `new Cat` transpiles to `new Cat()`
- [A]: Transpile `var class` into `var className`
  - Idiomatic way of referencing class names (typically CSS) b/c `class` is a reserved keyword
- [W]: Do not allow fall-through in switch statements
  - `switch` must have a `break` in each case
- [W]: Don't allow instantiations of primitives (very confusing behaviors)
  - `new String("abc")`, `new Boolean(true)`, `new Number(1)`
- [W]: Discourage use of `console.log` - typically only used for development. If used in production (e.g. displaying errors), then use an injectable logger (see Angular) `$log.log`
- [W]: No use of `eval()` - too dangerous, allows user-injected scripts to run
- [A]: `parseInt("108")` transpiles to `parseInt("108", 10)` to avoid accidentally using another radix (e.g. base 8)
- [W]: No `with` very un-common to use
- [W]: No assignments in a comparison check `if (x = 10)`, usually a typo for `if (x === 10)`
- [W]: No prefix increment or decrement operator (e.g. `--x` or `++x`)
   - Confusing order of operations; postfix increment (`x++`) is common for `for` loops
- [W]: Do not allow comparison checks with an object literal or array literal (95% likely to be a mistake)
  - `if (x === {})`
  - `if (x === ["a", "b"])`
- [W]: Suggest using template literals if there is `<tag></tag>` inside string literals
  - `"<body></body>"` into ``<body></body>``
- [W]: Suggests ways of fixing mixed use of single-quote and double-quote - frequently happens when working with HTML templates in JS
- [A]: Suggest fix for spy syntax. It's always `spyOn(obj, "propName")`. Frequently it gets confused for `spyOn("obj", "propName")` or `spyOn(obj.propName)` or `spyOn(obj, propName)`


- [A]: Autoformats long lists of parameter names (common in DI) by identing to when the first parameter begins

```
function (firstDep, secondDep,
          thirdDep)
```
- [A]: Allow an extra comma for last list item / property name (better diffs)
  - Automatically inserts the extra comma in the source file, but automatically removes it from transpiled file

```
list = [
    a,
    b,
]
```

## Typescript-specific
- [A]: Suggest a return type when its discernible
- [W]: Writing a two word parameter name - is it missing a colon for type annotation?

## Potential Areas of Improvement
- [A]: Make sure Array checks are valid
  - Transpile `typeof arr === "array"` into `Array.isArray(arr)`
- [F]: Immutability by using Icepick.js for small Array sizes

## Notes

- Include extensive docs - be able to expand each rule in the same page
  - Use Vue.js / Hashicorp / Eslint docs as inspiration
  - Use Semantic UI to create the collapsible elements OR use the PUI
- Call it:
  - Simple.js
  - Idiom.js
  - Idiomatic.js