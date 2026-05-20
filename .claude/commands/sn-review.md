# ServiceNow Code Review

Review ServiceNow JavaScript code for ES5 compliance, correctness, security, and ServiceNow best practices.

Arguments: $ARGUMENTS
(Optional: paste the code or point to a file. If blank, review all JS files in `scripts/`.)

## Review Checklist

### 1. ES5 Compliance (hard failures — must fix)
- [ ] No `let` or `const` — only `var`
- [ ] No arrow functions `() =>`
- [ ] No template literals (backticks)
- [ ] No `class` keyword
- [ ] No `async` / `await` / `Promise`
- [ ] No destructuring `const { a, b } = obj`
- [ ] No spread operator `...`
- [ ] No `for...of` loops
- [ ] No optional chaining `?.` or nullish coalescing `??`

### 2. Script Include Structure
- [ ] Uses `Class.create()` and `prototype` pattern
- [ ] Has `type: 'ClassName'` property on prototype
- [ ] `initialize` is the constructor function

### 3. GlideRecord Safety
- [ ] Every `gr.query()` is followed by a `gr.next()` check before accessing fields
- [ ] `gr.setValue()` is used for setting values (not direct assignment for reference fields)
- [ ] `gr.get(sysId)` used for single-record lookup (returns boolean)
- [ ] Large queries use `gr.setLimit()` to prevent runaway queries
- [ ] No raw SQL string concatenation in queries

### 4. REST / HTTP Safety
- [ ] No hardcoded credentials, tokens, or passwords
- [ ] `response.getStatusCode()` checked before using `response.getBody()`
- [ ] `JSON.parse()` wrapped in try/catch
- [ ] Timeouts set via `rm.setTimeout()`
- [ ] Retries are bounded (no infinite loops)

### 5. Error Handling
- [ ] Methods return result objects `{ success, data, error }` rather than throwing
- [ ] Errors logged with `gs.error()`, not silently swallowed
- [ ] No `gs.print()` (deprecated) — use `gs.info()` / `gs.warn()` / `gs.error()`

### 6. Security
- [ ] No hardcoded credentials anywhere
- [ ] No `eval()` usage
- [ ] User input not concatenated into GlideRecord queries without escaping

## Output Format

For each issue found:
```
SEVERITY: error | warning | info
RULE: <rule name>
LINE: ~N
ISSUE: <what is wrong>
FIX: <what to change it to>
```

Summary:
```
Errors (must fix): N
Warnings (should fix): N
```

If there are errors, produce the corrected code in full so the user can paste it directly into ServiceNow.
