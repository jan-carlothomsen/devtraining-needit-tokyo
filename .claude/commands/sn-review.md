# ServiceNow Code Review

Review ServiceNow JavaScript code in this repository for correctness, ES5 compliance, security, and ServiceNow best practices.

Arguments: $ARGUMENTS
(Optional: file path or class name to focus on. If blank, review all files in `update/` and `scripts/`.)

## Review Checklist

### 1. ES5 Compliance (hard failures — must fix)
- [ ] No `let` or `const` — only `var`
- [ ] No arrow functions `() =>`
- [ ] No template literals (backticks)
- [ ] No `class` keyword
- [ ] No `async` / `await` / `Promise`
- [ ] No destructuring assignment `const { a, b } = obj`
- [ ] No spread operator `...`
- [ ] No `for...of` loops (use `for` with index or `while (gr.next())`)
- [ ] No `Object.assign` with spread (use manual property copy)
- [ ] No optional chaining `?.` or nullish coalescing `??`

### 2. ServiceNow Script Include Structure
- [ ] Uses `Class.create()` and `prototype` pattern
- [ ] Has `type: 'ClassName'` property on prototype
- [ ] `initialize` function is the constructor
- [ ] `api_name` in XML matches `x_58872_needit.<ClassName>`
- [ ] `sys_package` and `sys_scope` both reference app sys_id `6ead8e780f603200cd674f8ce1050ed1`
- [ ] `sys_update_name` = `sys_script_include_` + `sys_id`

### 3. GlideRecord Safety
- [ ] Every `gr.query()` is followed by `gr.next()` before accessing fields
- [ ] Never access `gr.fieldName` after query without checking `gr.next()` returned true
- [ ] `gr.setValue()` is used for setting values (not direct assignment for reference fields)
- [ ] `gr.get(sysId)` is used for single-record lookup (returns boolean)
- [ ] Large queries use `gr.setLimit()` to prevent runaway queries
- [ ] Encoded queries use `addEncodedQuery()` or `addQuery()` — not string manipulation on SQL

### 4. REST / HTTP Safety (RESTMessageV2)
- [ ] Credentials are not hardcoded — use `gs.getProperty()` or `GlideEncrypter` or passed via config
- [ ] `response.getStatusCode()` is checked before using `response.getBody()`
- [ ] `JSON.parse()` is wrapped in try/catch (body may not be valid JSON)
- [ ] Timeouts are set (`rm.setTimeout()`)
- [ ] No unbounded retry loops (always limit retries)

### 5. Error Handling
- [ ] Methods return result objects `{ success, data, error }` rather than throwing
- [ ] `try/catch` around external calls (REST, GlideRecord in some contexts)
- [ ] Errors logged with `gs.error()`, not silently swallowed
- [ ] No `gs.print()` (deprecated) — use `gs.info()` / `gs.warn()` / `gs.error()`

### 6. Security
- [ ] No hardcoded credentials, tokens, or passwords in any file
- [ ] No `eval()` usage
- [ ] User-supplied input is not concatenated into GlideRecord encoded queries without escaping
- [ ] Reference fields set via `setValue()` or `setDisplayValue()`, not raw string assignment
- [ ] No `gs.executeQuery()` with raw SQL strings

### 7. XML Update Set Record
- [ ] `action="INSERT_OR_UPDATE"` on the table element
- [ ] `sys_id` is a valid 32-char lowercase hex string
- [ ] JavaScript inside `<![CDATA[...]]>` (not XML-escaped)
- [ ] `<client_callable>false</client_callable>` for server-side utilities
- [ ] No tracked-only fields (`sys_created_by`, `sys_updated_on`, etc.) in manually created records

## Output Format

For each issue found, report:
```
FILE: path/to/file.xml (line N)
SEVERITY: error | warning | info
RULE: <rule name from checklist>
ISSUE: <what is wrong>
FIX: <what to change it to>
```

Then provide a summary:
```
Errors (must fix before import): N
Warnings (should fix): N
Info (nice to fix): N
```

If no issues are found in a file, say so explicitly.

Finally, if there are errors: produce the corrected code directly.
