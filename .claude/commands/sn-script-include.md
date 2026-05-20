# ServiceNow Script Include Scaffold

Create a new ServiceNow Script Include for the NeedIt app. The argument is: `<ClassName> <short description>`.

Arguments: $ARGUMENTS

## App Context
- Scope: `x_58872_needit`
- App sys_id: `6ead8e780f603200cd674f8ce1050ed1`
- JS level: `helsinki_es5` — **strict ES5 only**

## ES5 Rules (never break these)
- `var` only — no `let`, no `const`
- No arrow functions `() =>`
- No template literals (backticks)
- No `class` keyword — use `Class.create()` + prototype
- No `Promise`, `async`, `await`
- No destructuring, no spread operator
- Logging: `gs.info()`, `gs.warn()`, `gs.error()`
- HTTP calls: `RESTMessageV2` (server-side)
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

Generate **two files**:

### 1. `update/sys_script_include_<snake_case_name>.xml`
Use this exact XML structure:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<record_update table="sys_script_include">
    <sys_script_include action="INSERT_OR_UPDATE">
        <access>public</access>
        <active>true</active>
        <api_name>x_58872_needit.ClassName</api_name>
        <client_callable>false</client_callable>
        <description>Short description here</description>
        <name>ClassName</name>
        <script><![CDATA[
// JavaScript goes here (ES5 only)
        ]]></script>
        <sys_class_name>sys_script_include</sys_class_name>
        <sys_id>GENERATE_A_UNIQUE_32_CHAR_HEX_ID</sys_id>
        <sys_package display_value="NeedIt" source="x_58872_needit">6ead8e780f603200cd674f8ce1050ed1</sys_package>
        <sys_scope display_value="NeedIt">6ead8e780f603200cd674f8ce1050ed1</sys_scope>
        <sys_update_name>sys_script_include_SAME_32_CHAR_HEX_ID</sys_update_name>
    </sys_script_include>
</record_update>
```
- Generate a random-looking but valid 32-character hex sys_id (lowercase)
- `sys_update_name` = `sys_script_include_` + that same sys_id

### 2. `scripts/<kebab-case-name>.js`
Same JavaScript, plus this header and stubs at the top:
```javascript
/**
 * ClassName
 * ServiceNow Script Include — x_58872_needit scope
 * Authoritative version: update/sys_script_include_<name>.xml
 */

/* ServiceNow global stubs for local testing */
if (typeof gs === 'undefined') {
    var gs = {
        info:  function(m) { console.log('[INFO]', m); },
        warn:  function(m) { console.warn('[WARN]', m); },
        error: function(m) { console.error('[ERROR]', m); },
        sleep: function(ms) {}
    };
}
if (typeof GlideRecord === 'undefined') { var GlideRecord = function(t) { this.table = t; }; }
if (typeof Class === 'undefined') { var Class = { create: function() { return function(){}; } }; }
/* End stubs */
```

After creating both files, remind the user to import the XML via:
**ServiceNow → System Update Sets → Retrieved Update Sets → Import Update Set from XML**
