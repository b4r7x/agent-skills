# Code Slop Patterns

Reference for detecting AI-generated patterns in source code. Each pattern includes detection heuristics and fix guidance.

Language focus: TypeScript / JavaScript. Patterns marked `(universal)` apply to any language.

---

## Category 1 — Unnecessary Comments (universal)

Comments that add no information beyond what the code already says.

### 1a. Restating the code

```
❌ // increment counter
   counter++;

❌ // check if user is authenticated
   if (isAuthenticated) {

❌ // return the result
   return result;

❌ // set the name
   this.name = name;
```

**Detection:** Comment immediately before a statement where the comment is a plain-English restatement of the next line. Look for comments containing the same identifiers as the code below them.

**Fix:** Delete the comment.

### 1b. Obvious JSDoc / docstrings

```
❌ /**
    * Gets the user by ID.
    * @param id - The user ID
    * @returns The user
    */
   function getUserById(id: string): User {

❌ /** The name of the user */
   name: string;

❌ /** Constructor */
   constructor() {
```

**Detection:** JSDoc/docstring where every line is derivable from the function signature, parameter names, or property name. If removing the comment loses zero information, it's slop.

**Fix:** Delete the docstring. Keep only docstrings that explain *why*, *gotchas*, *side effects*, or *non-obvious behavior*.

### 1c. TODO placeholders (universal)

```
❌ // TODO: implement error handling
❌ // TODO: add validation
❌ // TODO: handle edge cases
❌ // FIXME: improve performance
```

**Detection:** `TODO` or `FIXME` comments that are vague and don't reference a ticket, issue, or specific scenario. Generic TODOs left by AI that will never be addressed.

**Fix:** Delete if the code already handles the case or if the TODO is too vague to act on. If it's a real concern, either implement it or reference a ticket: `// TODO(PROJ-123): rate limit this endpoint`.

### 1d. Section dividers and decoration (universal)

```
❌ // ============================================
   // Authentication Methods
   // ============================================

❌ // --- Helper Functions ---

❌ /* ******* EXPORTS ******* */
```

**Detection:** Comment lines that are primarily decoration characters (`=`, `-`, `*`, `#`) or section labels that mirror the file structure already visible from function/class names.

**Fix:** Delete. If the file is so long it needs section headers, consider splitting it instead.

### 1e. "Removed" markers (universal)

```
❌ // removed: old validation logic
❌ // previously: this used to call the legacy API
❌ // deprecated — keeping for reference
```

**Detection:** Comments referencing removed/old/previous code. This is what git history is for.

**Fix:** Delete the comment entirely.

---

## Category 2 — Over-Engineering

Code that's more complex than the problem requires.

### 2a. Premature abstractions (universal)

```
❌ // Used exactly once
   function formatUserName(user: User): string {
     return `${user.firstName} ${user.lastName}`;
   }

   // ...later:
   const displayName = formatUserName(user);

✅ const displayName = `${user.firstName} ${user.lastName}`;
```

**Detection:** Helper function/method that is called exactly once, has a trivial body (1-3 lines), and doesn't simplify a complex expression. Search for call sites — if there's only one, it's likely premature.

**Fix:** Inline the function body at the call site. Delete the function.

### 2b. Unnecessary factory / builder patterns

```
❌ function createConfig(options: ConfigOptions): Config {
     return {
       host: options.host ?? 'localhost',
       port: options.port ?? 3000,
     };
   }
   const config = createConfig({ host: 'example.com' });

✅ const config = { host: 'example.com', port: 3000 };
```

**Detection:** Factory function that just spreads defaults into an object literal. Builder patterns where the built object has ≤5 properties.

**Fix:** Inline the object. Use spread or `??` directly where needed.

### 2c. Config for hardcoded values

```
❌ const MAX_RETRIES = 3;
   const RETRY_DELAY_MS = 1000;
   const BASE_URL = 'https://api.example.com';
   // ...used in exactly one place each

❌ // config.ts — exported but consumed by a single file
   export const config = {
     maxRetries: 3,
     retryDelay: 1000,
     baseUrl: 'https://api.example.com',
   };
```

**Detection:** Constants or config objects used in exactly one place. Extracting a value to a constant is useful when it's shared or when the name adds meaning — not when `3` is already obvious in context.

**Fix:** Inline the value if it's used once and self-evident. Keep it if the name genuinely clarifies intent (e.g., `BCRYPT_ROUNDS = 12`).

### 2d. Wrapper functions that just forward (universal)

```
❌ function logError(message: string) {
     console.error(message);
   }

❌ async function fetchData(url: string) {
     return fetch(url).then(res => res.json());
   }
```

**Detection:** Function whose entire body is a single call to another function with the same (or subset of) arguments. No added logic, no transformation, no error handling.

**Fix:** Call the underlying function directly. Delete the wrapper.

### 2e. Unnecessary interface/type for one-off shapes

```
❌ interface ProcessUserDataOptions {
     userId: string;
     includeProfile: boolean;
   }
   function processUserData(options: ProcessUserDataOptions) { ... }
   // called exactly once

✅ function processUserData(userId: string, includeProfile: boolean) { ... }
```

**Detection:** Interface or type alias used by exactly one function, with ≤3 properties, where named parameters would be clearer.

**Fix:** Use direct parameters or an inline type. Delete the interface.

---

## Category 3 — Defensive Over-Coding

Error handling and validation for scenarios that can't happen.

### 3a. Null checks on typed non-nullable values

```
❌ function greet(name: string) {
     if (!name) {
       throw new Error('Name is required');
     }
     return `Hello, ${name}`;
   }
```

**Detection:** Guard clauses checking for null/undefined/falsy on parameters that TypeScript already guarantees are non-nullable. Check the type signature — if the param isn't `string | null | undefined`, the check is slop.

**Fix:** Delete the guard clause. Trust the type system.

### 3b. Try-catch wrapping infallible operations (universal)

```
❌ try {
     const sum = a + b;
   } catch (error) {
     console.error('Failed to calculate sum:', error);
   }

❌ try {
     const items = array.filter(x => x.active);
   } catch (error) {
     return [];
   }
```

**Detection:** try-catch around pure operations: arithmetic, string manipulation, array methods on known arrays, object property access on typed objects. These can't throw.

**Fix:** Remove the try-catch. Keep the operation.

### 3c. Redundant else-after-return (universal)

```
❌ if (condition) {
     return early;
   } else {
     // ...rest of function
   }

✅ if (condition) {
     return early;
   }
   // ...rest of function
```

**Detection:** `else` block after a `return`, `throw`, or `continue` statement.

**Fix:** Remove the `else`, dedent the body.

### 3d. Exhaustive error messages for internal errors (universal)

```
❌ if (!user) {
     throw new Error(
       `User with ID "${userId}" was not found in the database. ` +
       `Please verify the user ID is correct and the user exists. ` +
       `This may indicate a data integrity issue.`
     );
   }

✅ if (!user) {
     throw new Error(`User not found: ${userId}`);
   }
```

**Detection:** Error messages longer than ~80 characters that include troubleshooting advice, multiple sentences, or explanations of what "may" have gone wrong.

**Fix:** Shorten to the essential fact. Error messages are for developers reading logs, not end users.

### 3e. Fallback values that hide bugs (universal)

```
❌ const port = config.port ?? 3000;  // config.port is always set
❌ const name = user?.name ?? 'Unknown';  // user is never null here
```

**Detection:** Nullish coalescing (`??`) or optional chaining (`?.`) on values that the surrounding code guarantees are present. Check if the value could actually be null/undefined in this context.

**Fix:** Remove the fallback. Access the value directly. If it *can* be null, that's a real bug to fix upstream, not mask.

---

## Category 4 — AI Voice in Code

Naming and comments that reveal AI authorship through characteristic phrasing.

### 4a. "Enhanced" / "Optimized" / "Improved" naming (universal)

```
❌ const enhancedUser = { ...user, fullName };
❌ function optimizedSearch(items: Item[]) { ... }
❌ const improvedConfig = mergeConfigs(base, override);
```

**Detection:** Variable or function names containing: `enhanced`, `optimized`, `improved`, `robust`, `elegant`, `sophisticated`, `streamlined`, `comprehensive`. These are AI confidence markers, not descriptions.

**Fix:** Name it for what it *is*, not how good you think it is: `userWithFullName`, `searchItems`, `mergedConfig`.

### 4b. "Ensure" / "robust" / "graceful" in comments (universal)

```
❌ // Ensure proper error handling
❌ // Handle edge case gracefully
❌ // Robust validation of input
❌ // Properly clean up resources
❌ // Safely parse the response
```

**Detection:** Comments containing: `ensure`, `robust`, `graceful`, `properly`, `safely`, `elegant`, `comprehensive`, `appropriate`. These are AI filler words that add no specificity.

**Fix:** Either delete the comment (the code should speak for itself) or rewrite with the specific concern: `// close the DB connection even if the query fails`.

### 4c. Over-narrating control flow (universal)

```
❌ // First, we validate the input
   validate(input);
   // Then, we process the data
   const result = process(input);
   // Finally, we return the result
   return result;
```

**Detection:** Sequential comments that narrate the code like a tutorial: "First...", "Next...", "Then...", "Finally...", "Now we...". The code already reads top to bottom.

**Fix:** Delete all narration comments.

### 4d. Apologetic / hedging comments (universal)

```
❌ // This might not be the most efficient approach, but...
❌ // Note: This is a simplified implementation
❌ // TODO: Consider a more elegant solution
❌ // This works for now but could be improved
```

**Detection:** Comments containing hedging language: "might not be", "simplified", "for now", "could be improved", "not ideal", "workaround". AI produces these when it's unsure but doesn't want to commit.

**Fix:** Delete. If the code works, ship it. If it doesn't, fix it — don't apologize.

---

## Category 5 — Hallucinated / Dead Code

Code that doesn't do anything or references things that don't exist.

### 5a. Unused imports (universal)

```
❌ import { useState, useEffect, useCallback, useMemo } from 'react';
   // ...only useState is used in the file
```

**Detection:** Imports where some or all symbols aren't referenced elsewhere in the file.

**Fix:** Remove unused imports. Most linters catch this — if present, it means linting isn't running.

### 5b. Assigned-but-never-read variables (universal)

```
❌ const startTime = Date.now();
   doWork();
   // startTime never used again
```

**Detection:** Variable assigned a value but never referenced after assignment.

**Fix:** Delete the assignment. If it was meant for timing/logging, it was never finished — remove it.

### 5c. Unreachable code (universal)

```
❌ function getStatus() {
     return 'active';
     console.log('done');  // unreachable
   }
```

**Detection:** Code after `return`, `throw`, `break`, or `continue` in the same block.

**Fix:** Delete the unreachable code.

### 5d. Re-exports for "backwards compatibility"

```
❌ // Re-export for backwards compatibility
   export { OldName as NewName } from './module';

❌ // Keep old import path working
   export * from './new-location';
```

**Detection:** Exports with comments mentioning "backwards compatibility", "legacy", "old import path", "keep working". In a new codebase or when there are no external consumers, these are premature.

**Fix:** Delete the re-export. Update import paths at the actual call sites.

---

## Category 6 — Type Workarounds

Bypassing the type system instead of fixing the actual types.

### 6a. `as any` casts

```
❌ const data = response.body as any;
❌ (element as any).customMethod();
❌ return result as any as ExpectedType;
```

**Detection:** `as any` in TypeScript. Almost always a sign the real type is wrong or missing.

**Fix:** Fix the type upstream. Use `unknown` with type narrowing if the type genuinely isn't known. Double-cast (`as any as X`) is always wrong — fix the intermediate type.

### 6b. `@ts-ignore` / `@ts-expect-error` without context

```
❌ // @ts-ignore
   const value = thing.property;

❌ // @ts-expect-error
   someFunction(wrongArg);
```

**Detection:** TypeScript suppression comments without an explanation of *why* the error is expected/acceptable.

**Fix:** Fix the type error. If suppression is truly needed (e.g., library type bug), add the reason: `// @ts-expect-error — library types don't include the v3 overload`.

### 6c. Unnecessary type assertions

```
❌ const name = user.name as string;  // it's already string
❌ const items = getItems() as Item[];  // getItems already returns Item[]
```

**Detection:** `as X` where the expression is already typed as `X` by inference or declaration.

**Fix:** Remove the assertion.

---

## Category 7 — Verbose Patterns

Code that takes more space than needed for the same result.

### 7a. Ternary replaceable by `??` or `||`

```
❌ const name = user.name ? user.name : 'Anonymous';
✅ const name = user.name ?? 'Anonymous';

❌ const value = input !== undefined ? input : defaultValue;
✅ const value = input ?? defaultValue;
```

**Detection:** Ternary where the condition checks the same value returned in the truthy branch.

**Fix:** Replace with `??` (for null/undefined) or `||` (for all falsy).

### 7b. Boolean comparison verbosity

```
❌ if (isActive === true) {
❌ if (isValid === false) {
❌ if (!!value) {
❌ return condition ? true : false;

✅ if (isActive) {
✅ if (!isValid) {
✅ if (value) {
✅ return condition;
```

**Detection:** Explicit comparison against boolean literals, double-bang coercion, or ternary returning `true`/`false`.

**Fix:** Use the boolean directly.

### 7c. Unnecessary async/await

```
❌ async function getUser(id: string) {
     return await fetchUser(id);
   }

✅ function getUser(id: string) {
     return fetchUser(id);
   }
```

**Detection:** `async` function whose only `await` is on the return statement. The function can just return the promise directly.

**Fix:** Remove `async` and `await`. Return the promise. (Exception: if try-catch wraps the await, keep it.)

### 7d. Spreading just to copy

```
❌ const copy = { ...original };
   // copy is never mutated, or original is never used again

✅ // just use original directly
```

**Detection:** Object/array spread where the copy is never mutated separately from the original, or the original is immediately discarded.

**Fix:** Use the original directly. Only copy when mutation isolation is actually needed.

---

## Severity Guide

| Severity | Meaning | Examples |
|----------|---------|---------|
| **high** | Actively harmful — hides bugs, breaks types, or misleads | `as any`, fallbacks hiding bugs, hallucinated imports |
| **medium** | Degrades readability and maintainability | Unnecessary comments, over-engineering, verbose patterns |
| **low** | Minor style issue, cosmetic | Section dividers, boolean verbosity, redundant else |

## Language-Specific Notes

### Python

- Patterns 1a-1e apply (docstrings instead of JSDoc)
- Pattern 3a: check type hints — `def greet(name: str)` doesn't need `if name is None`
- Pattern 6: `# type: ignore` without explanation is the Python equivalent of `@ts-ignore`
- Pattern 7c: same issue with `async def` that just `return await`

### Go

- Pattern 1b: exported function comments are required by convention — but they should say something useful, not restate the signature
- Pattern 2a: Go community accepts small functions for readability — be more conservative flagging these
- Pattern 3c: Go uses early-return heavily — flag `else` after `return` confidently

### Rust

- Pattern 6: `unsafe` blocks without `// SAFETY:` comment explaining why it's sound
- Pattern 7: `.unwrap()` without context — should use `expect("reason")` or proper error handling
