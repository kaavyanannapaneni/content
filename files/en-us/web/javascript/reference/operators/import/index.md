---
title: import
slug: Web/JavaScript/Reference/Operators/import
tags:
  - ECMAScript 2015
  - JavaScript
  - Language feature
  - Modules
  - Reference
  - dynamic import
  - import
browser-compat: javascript.operators.import
---

{{jsSidebar("Operators")}}

The `import()` call, commonly called _dynamic import_, is a function-like expression that allows loading an ECMAScript module asynchronously and dynamically into a potentially non-module environment.

Unlike the [declaration-style counterpart](/en-US/docs/Web/JavaScript/Reference/Statements/import), dynamic imports are only evaluated when needed, and permits greater syntactic flexibility.

## Syntax

```js
import(moduleName)
```

The `import()` call is a syntax that closely resembles a function call, but `import` itself is a keyword, not a function. You cannot alias it like `const myImport = import`, which will throw a {{jsxref("SyntaxError")}}.

### Parameters

- `moduleName`
  - : The module to import from. The evaluation of the specifier is host-specified, but always follows the same algorithm as static [import declarations](/en-US/docs/Web/JavaScript/Reference/Statements/import).

### Return value

It returns a promise which fulfills to an object containing all exports from `moduleName`, with the same shape as a namespace import (`import * as name from moduleName`): an object with `null` prototype, and the default export available as a key named `default`.

## Description

The import declaration syntax (`import something from "somewhere"`) is static and will always result in the imported module being evaluated at load time. Dynamic imports allows one to circumvent the syntactic rigidity of import declarations and load a module conditionally or on demand. The following are some reasons why you might need to use dynamic import:

- When importing statically significantly slows the loading of your code and there is a low likelihood that you will need the code you are importing, or you will not need it until a later time.
- When importing statically significantly increases your program's memory usage and there is a low likelihood that you will need the code you are importing.
- When the module you are importing does not exist at load time.
- When the import specifier string needs to be constructed dynamically. (Static import only supports static specifiers.)
- When the module being imported has side effects, and you do not want those side effects unless some condition is true. (It is recommended not to have any side effects in a module, but you sometimes cannot control this in your module dependencies.)
- When you are in a non-module environment (for example, `eval` or a script file).

Use dynamic import only when necessary. The static form is preferable for loading initial dependencies, and can benefit more readily from static analysis tools and [tree shaking](/en-US/docs/Glossary/Tree_shaking).

If your file is not run as a module (if it's referenced in an HTML file, the script tag must have `type="module"`), you will not be able to use static import declarations, but the asynchronous dynamic import syntax will always be available, allowing you to import modules into non-module environments.

## Examples

### Import a module for its side effects only

```js
(async () => {
  if (somethingIsTrue) {
    // import module for side effects
    await import("/modules/my-module.js");
  }
})();
```

If your project uses packages that export ESM, you can also import them for side
effects only. This will run the code in the package entry point file (and any files it
imports) only.

### Importing defaults

You need to destructure and rename the "default" key from the returned object.

```js
(async () => {
  if (somethingIsTrue) {
    const {
      default: myDefault,
      foo,
      bar,
    } = await import("/modules/my-module.js");
  }
})();
```

### Importing on-demand in response to user action

This example shows how to load functionality on to a page based on a user action, in this case a button click, and then call a function within that module. This is not the only way to implement this functionality. The `import()` function also supports `await`.

```js
const main = document.querySelector("main");
for (const link of document.querySelectorAll("nav > a")) {
  link.addEventListener("click", (e) => {
    e.preventDefault();

    import("/modules/my-module.js")
      .then((module) => {
        module.loadPageInto(main);
      })
      .catch((err) => {
        main.textContent = err.message;
      });
  });
}
```

### Importing different modules based on environment

In processes such as server-side rendering, you may need to load different logic on server or in browser because they interact with different globals or modules (for example, browser code has access to web APIs like `document` and `navigator`, while server code has access to the server file system). You can do so through a conditional dynamic import.

```js
let myModule;

if (typeof window === "undefined") {
  myModule = await import("module-used-on-server");
} else {
  myModule = await import("module-used-in-browser");
}
```

### Importing modules with a non-literal specifier

Dynamic imports allow any expression as the module specifier, not necessarily string literals.

Here, we load 10 modules, `/modules/module-0.js`, `/modules/module-1.js`, etc., in parallel, and call the `load` functions that each one exports.

```js
Promise.all(
  Array.from({ length: 10 }).map((_, index) =>
    import(`/modules/module-${index}.js`)
  )
).then((modules) => modules.forEach((module) => module.load()));
```

## Specifications

{{Specifications}}

## Browser compatibility

{{Compat}}

## See also

- [`import` declaration](/en-US/docs/Web/JavaScript/Reference/Statements/import)
