---
name: refactoring-code
description: MUST be loaded BEFORE writing any answer when the user asks to review, clean up, tidy, refactor, simplify, rewrite or improve code, says code is hard to read, unreadable, messy, confusing, complex, over-nested or magic, or asks for clean code, even for one short snippet; also load it to self-review your own diff before reporting. Holds house rules for code a human reads in one pass (guard clauses, loops instead of dense array_*/collection-function chains, no nested ternaries, named variables, flat nesting, honest names), which differ from default habits, plus all 23 code smells and 66 techniques of refactoring.guru and React patterns. Any language.
---

# Refactoring for human readers

## Goal

Code a person reads in one pass beats fewer lines or cleverness. A line that needs a second read loses to three that do not. A comment that says *what* a line does is a sign to rewrite the line.

## House rules

They win over refactoring.guru where the two conflict. Formatting follows the project's own style files; these rules are about readability.

1. Guard clauses and early return. No `else` after `return`, `continue`, `throw` or `break`.
2. Nesting depth: at most two blocks in a function (loop + `if` is fine). A loop in a loop with a condition: a `continue` guard, or move the inner part into a named method.
3. One collection function per expression. Two or more, or a callback inside a nested call (`array_unique(array_filter(array_map(fn…)))`): a loop with `if`, or named intermediate variables. In that loop `array_filter` without a callback becomes `if ($value)` (same truthiness; `!== ''` lets `NULL` and `'0'` through) and `array_unique` becomes an array keyed by the value, never kept after the loop. TS: one `.filter`/`.map` is fine; two calls are rewritten unless both callbacks are one property read or one comparison; three calls or any `.reduce`: a loop or named steps. **Exception:** `array_keys(array_filter(...))` is always a loop with `if`, even split over two variables: `array_filter` without a callback is a silent truthiness trick and naming the variable does not remove it.
4. Compute conditional values into named variables before a `return`, an array or object literal, or JSX.
5. A ternary only picks between two simple values. Never nest one.
6. Names come from the domain: no abbreviations or placeholders; a variable is called what the UI text or error message calls it.
7. A comment explains *why*, never *what*.
8. No wrapper method around an existing one-line call. Extract a method only when its name says what the code does not.
9. No defensive code for cases that cannot happen; prove a case is reachable first: find every caller, check what the UI already prevents (a modal, a disabled control) and what access rules allow.
10. No magic literals in calls (`0`/`1`, boolean flags): named constants, factory methods or separate methods.
11. No silent tricks: `+` on arrays that quietly keeps the left keys; comparing an array with its own filtered copy.
12. Behaviour stays identical. Compare outputs before and after, edge inputs included. A rewrite that changes an edge case is a bug fix, not readability: leave it out or report it.
13. Anything outside the task is listed separately, never changed unasked. Always scan the signature for it: a boolean parameter that picks a behaviour, a long parameter list, a name that lies.

## Review workflow

Follow this checklist every time; copy it into the answer only for a review the user asked for on its own:

```
- [ ] 1. Read the code to the end before judging
- [ ] 2. House rules: grep the diff (see Quick search), check each hit
- [ ] 3. Name the smell for each hit and for each signature flaw; open its smells/ file only if unsure
- [ ] 4. Pick the technique; open its file (techniques/, expressions.md or react.md) for the steps, edge cases and when-not
- [ ] 5. Rewrite; behaviour unchanged (compare outputs where possible)
- [ ] 6. Re-read every new line as a human without context; nothing needs a second pass
```

## Router

Every row is one file; the last column is an exact `## ` heading in it.

| Symptom | File | Entry |
|---|---|---|
| Nested if/else, `else` after `return` | techniques/simplifying-conditionals.md | Replace Nested Conditional with Guard Clauses |
| Two or more collection functions in one expression | expressions.md | Dense collection-function composition |
| Loop in a loop with a condition | expressions.md | Nested loops with conditions |
| Long function, a comment above each block | techniques/composing-methods.md | Extract Method |
| Comment says what a line does | smells/dispensables.md | Comments |
| Boolean parameter picks a behaviour | techniques/simplifying-method-calls.md | Replace Parameter with Explicit Methods |
| Long parameter list | techniques/simplifying-method-calls.md | Introduce Parameter Object |
| Magic number or string | techniques/organizing-data.md | Replace Magic Number with Symbolic Constant |
| Same code in two places | smells/dispensables.md | Duplicate Code |
| Nested or long ternary | expressions.md | Nested and long ternaries |
| Long boolean condition with `and`/`or` | expressions.md | Long boolean expressions |
| Condition inside a `return` or a literal | expressions.md | Conditional values inside return and literals |
| `+` between arrays | expressions.md | Silent array union |
| One loop does two jobs | expressions.md | Split Loop |
| Pipeline reads worse than a loop | expressions.md | Replace Loop with Pipeline |
| `useEffect` that only sets state from props | react.md | Derived state in useEffect |
| Ternary inside JSX | react.md | Nested ternaries in JSX |
| Long condition in JSX | react.md | Long JSX conditions |
| Handler with logic written in JSX | react.md | Inline handlers with logic |
| Component does three things | react.md | Extract Component |
| Props passed through layers that ignore them | react.md | Props drilling |
| `switch` or `if` chain on a type | smells/oo-abusers.md | Switch Statements |
| `$a->b()->c()->d()` | smells/couplers.md | Message Chains |
| Method mostly uses another object's data | smells/couplers.md | Feature Envy |
| Array used as a record | techniques/organizing-data.md | Replace Array with Object |
| Primitive carries a domain idea | smells/bloaters.md | Primitive Obsession |
| Error code or `null` as failure | techniques/simplifying-method-calls.md | Replace Error Code with Exception |
| One variable reused for different things | techniques/composing-methods.md | Split Temporary Variable |
| Class does too much | techniques/moving-features.md | Extract Class |

Smell entries name their techniques and files. All 13 files:

- smells/bloaters.md: Long Method, Large Class, Primitive Obsession, Long Parameter List, Data Clumps
- smells/oo-abusers.md: Alternative Classes with Different Interfaces, Refused Bequest, Switch Statements, Temporary Field
- smells/change-preventers.md: Divergent Change, Parallel Inheritance Hierarchies, Shotgun Surgery
- smells/dispensables.md: Comments, Duplicate Code, Data Class, Dead Code, Lazy Class, Speculative Generality
- smells/couplers.md: Feature Envy, Inappropriate Intimacy, Incomplete Library Class, Message Chains, Middle Man
- techniques/composing-methods.md: Extract Method, Extract Variable, Inline Temp, Substitute Algorithm and five more
- techniques/moving-features.md: Move Method, Extract Class, Hide Delegate and five more
- techniques/organizing-data.md: Replace Magic Number, Replace Array with Object, Encapsulate Collection and 12 more
- techniques/simplifying-conditionals.md: Decompose Conditional, Guard Clauses, Polymorphism, Null Object and four more
- techniques/simplifying-method-calls.md: Rename Method, Parameter Object, Explicit Methods and 11 more
- techniques/generalization.md: Pull Up, Push Down, Extract Superclass, Template Method and eight more
- expressions.md: collection functions, nested loops, ternaries, boolean expressions, literals, array union, Split Loop
- react.md: derived state, JSX conditions, handlers, Extract Component, props drilling

## Quick search

Run from this skill's directory; the diff checks run on the changed files.

```bash
grep -nE '^## ' techniques/*.md smells/*.md expressions.md react.md        # find an entry by name
grep -nE 'array_(map|filter|keys|values|intersect|diff|flip|column|merge|unique|fill_keys)\(.*array_' FILE   # dense PHP
grep -nE '\] \+ \$' FILE                                                    # silent array union
grep -Pzo '(return|continue|throw|break)\b[^;]*;\s*\}\s*else' FILE          # else after return
grep -nE ' \? [^;]* \? ' FILE                                               # nested ternary
grep -nE '\)\.(filter|map|reduce|flatMap)\(.*\)\.(filter|map|reduce|flatMap)\(' FILE   # TS chain
```

## When NOT to refactor

- The code is outside the task: report it, leave it.
- There is no way to compare behaviour before and after.
- The rewrite is longer and reads no better.
- The code already reads in one pass. "Already readable, no change" is a valid review: do not add wrappers, collection functions or rewrites to look useful.

## Reporting

One finding per block: `file:line — smell — technique`, then the code after in 2–4 lines. Findings outside the task go in a separate list at the end, unchanged.
