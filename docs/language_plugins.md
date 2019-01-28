# Language plugins

Iodide supports programming languages other than Javascript through the use of
language plugins.

Languages that are mature and well supported automatically load when you create
and run a JSMD chunk for that language. These are called "built-in" because
Iodide knows about them, even if support is loaded dynamically only when needed.
Other languages require the use of a language plugin chunk, described below.

Not all language plugins have the same level of interoperability between
Javascript and the plugin language. We've defined the following approximate
levels of support to make it easier to know what to expect.

- Level 1: The plugin just provides string output, so is useful as a basic
  console REPL (read-eval-print-loop).
  
- Level 2: The plugin converts basic data types (numbers, strings, arrays and
  objects) to and from Javascript.
  
- Level 3: The plugin supports sharing of class instances (objects with methods)
  between the plugin language and Javascript.
  
- Level 4: The plugin supports sharing of *n*-dimensional arrays and data frames
  between the plugin language and Javascript.
  
| Project                                                                         | Language       | Level 1 | Level 2 | Level 3 | Level 4 | Built-in |
|---------------------------------------------------------------------------------|----------------|---------|---------|---------|---------|----------|
| [Pyodide](http://github.com/iodide-project/pyodide)                             | Python         | ✓       | ✓       | ✓       | partial | ✓        |
| [AssemblyScript](https://codepen.io/ds604/pen/432d293df115432859f0af62e69d2e64) | AssemblyScript | ✓       | ✓       |         |         |          |
| [Lua](https://codepen.io/ds604/pen/d9791eed4e1ce19e11fb0f3c71000d72)            | Lua            | ✓       | ✓       |         |         |          |
| [Opal](https://codepen.io/ds604/pen/6cef7f732750cb2bde469b5ff387a320)           | Ruby           | ✓       | ✓       |         |         |          |
| [Julide](https://github.com/keno/julia-wasm)                                                                        | Julia           | ✓        |         |         |         |          |
| [Domical](https://github.com/louisabraham/domical)                              | OCaml          | ✓       |         |         |         |          |

(We try to keep the above table up-to-date, but things fall through the cracks.
Let us know on [Github](http://github.com/iodide-project/iodide/) if you see anything
needs updating).

## Using a custom language plugin

The language plugin is specified by a JSMD chunk containing a JSON string with
the following format:

```
%% plugin

{
  "languageId": "jsx",
  "displayName": "React JSX",
  "codeMirrorMode": "jsx",
  "keybinding": "x",
  "url": "https://raw.githubusercontent.com/hamilton/iodide-jsx/master/docs/evaluate-jsx.js",
  "module": "jsx",
  "evaluator": "evaluateJSX",
  "pluginType": "language"
}
```

The individual fields are described below:

- `languageId`: A short identifier for the language.  This is used to specify the language at the beginning of a [JSMD](jsmd.md) chunk, for example, `%% jsx`.  By convention, this should be the filename extension that is most commonly used for the language.

- `displayName`: A longer name used to identify the language in menus and other UX elements.

- `codeMirrorMode`: The name of the CodeMirror plugin used to provide syntax highlighting for the language.  A list of the available plugin names is [here](https://github.com/codemirror/CodeMirror/tree/master/mode).

- `keybinding`: The key used to select the language.  (TODO: Is this used anymore following the JSMD editing refactor?)

- `url`: The URL to a Javascript source file that defines the language support.  It is evaluated directly in the scope that runs Iodide user code, therefore it should should be "modularized" such that it only adds a single object to the global namespace.

- `module`: The name of the module provided by the Javascript file given by `url`.

- `evaluator`: The name of the function in the module that evaluates code for the custom language.  For example, if `module` is `jsx` and `evaluator` is `evaluateJSX`, Iodide will call `window.jsx.evaluateJSX()` to run code for the custom language.  This function must take a single string argument containing source code, and return an arbitrary Javascript value for the result.  For the best user experience when displaying values in the UI, this object should be a Javascript representation that mirrors as closely as possibly the representation in the plugin language.  To return a custom HTML representation of the object, return an object with a `iodideRender` method, returning a string of raw HTML.

- `asyncEvaluator`: (optional) If evaluating code requires making asynchronous calls, for example, to load additional code from a remote location, an `asyncEvaluator` method should be provided.  It will take precendence over `evaluator` if provided.  It takes a string of source code, but returns a `Promise` that resolves to result value rather than returning the result immediately.  Otherwise, it follows the same conventions as `evaluator`.

- `pluginType`: Must always be `language` for language plugins.  Other values are resolved for other plugin types to be defined in the future.