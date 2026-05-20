# ServiceNow Script Include Scaffold

Create a new ServiceNow Script Include for the NeedIt app.

Arguments: $ARGUMENTS
(Expected: `<ClassName> <short description>`)

## App Context
- Scope: `x_58872_needit`
- JS level: `helsinki_es5` — **strict ES5 only**

## ES5 Rules (never break these)
- `var` only — no `let`, no `const`
- No arrow functions `() =>`
- No template literals (backticks)
- No `class` keyword — use `Class.create()` + prototype
- No `Promise`, `async`, `await`
- No destructuring, no spread operator
- Logging: `gs.info()`, `gs.warn()`, `gs.error()`
- HTTP calls: `RESTMessageV2`
- DB access: `GlideRecord`, `GlideAggregate`

## Script Include Class Pattern
```javascript
var ClassName = Class.create();
ClassName.prototype = {
    initialize: function(config) {
        // constructor — config is a plain object
    },
    methodName: function(arg) {
        // implementation
    },
    type: 'ClassName'  // required — enables gs.instanceOf() checks
};
```

## What to produce

Output **only the JavaScript** — no XML, no file wrappers.

The user will paste this directly into the ServiceNow Script Include editor:
- **Name** field: `ClassName`
- **API Name** field: `x_58872_needit.ClassName`  
- **Script** field: the JavaScript you produce

After the code, show a short usage example the user can run in a Background Script to test it.
