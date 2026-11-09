# Dispensables

Code whose removal makes the codebase better. Entries: Comments, Duplicate Code, Data Class, Dead Code, Lazy Class, Speculative Generality.

## Comments
<https://refactoring.guru/smells/comments>

**Signs:** A method full of explanatory notes; a comment above a block saying what it does; a comment decoding a dense expression.
**Why it hurts:** The comment patches unclear code instead of fixing it; code that only makes sense with commentary needs restructuring. House rule 7: a comment says *why*, never *what*.
**Treatment:** Comment explains an expression — Extract Variable; comment describes a block — Extract Method, seeding the name from the comment (`techniques/composing-methods.md`); a method exists but needs a note — Rename Method (`techniques/simplifying-method-calls.md`); comment states a required condition — Introduce Assertion (`techniques/simplifying-conditionals.md`).
**Ignore when:** The comment explains why an approach was chosen, or documents a complex algorithm after simpler restructuring is exhausted.

## Duplicate Code
<https://refactoring.guru/smells/duplicate-code>

**Signs:** Two fragments that look almost the same — or look different and do the same job.
**Why it hurts:** Born of developers working in parallel unaware of each other, or of a deadline copy of something "close enough"; every fix must then be made twice.
**Treatment:** By where the copies sit (house rule 12: merge only after confirming identical behaviour):
- several methods of one class — Extract Method (`techniques/composing-methods.md`)
- sibling subclasses — Extract Method, then Pull Up Field or Pull Up Constructor Body; similar but not identical — Form Template Method; same result by different means — Substitute Algorithm (`techniques/generalization.md`, `techniques/composing-methods.md`)
- unrelated classes — Extract Superclass (`techniques/generalization.md`), else Extract Class and use it from the other (`techniques/moving-features.md`)
- many conditions with the same body — Consolidate Conditional Expression, then Extract Method; the same code in every branch — Consolidate Duplicate Conditional Fragments (`techniques/simplifying-conditionals.md`)

**Ignore when:** Rarely, merging two fragments makes the code less clear.

## Data Class
<https://refactoring.guru/smells/data-class>

**Signs:** A class of fields and plain getters/setters; other classes do all the work with its data.
**Why it hurts:** It began as exposed fields and never gained behaviour, so code about one set of data is scattered.
**Treatment:** Encapsulate Field and Encapsulate Collection (`techniques/organizing-data.md`); Move Method and Extract Method to pull the client logic that belongs to the data into the class (`techniques/moving-features.md`, `techniques/composing-methods.md`); then Remove Setting Method and Hide Method for access it no longer needs (`techniques/simplifying-method-calls.md`).
**Ignore when:** Nothing on the site. Own note: a readonly DTO that only carries values across a boundary is fine.

## Dead Code
<https://refactoring.guru/smells/dead-code>

**Signs:** Something declared and never referenced — a local, an argument, a property, a method, a whole class; a branch that can never be reached.
**Why it hurts:** Requirements change and fixes land, but nobody removes the leftovers; the codebase carries them forever.
**Treatment:** Delete it (an IDE or static analysis finds it fastest); Inline Class or Collapse Hierarchy for a class made useless by its parent or child (`techniques/moving-features.md`, `techniques/generalization.md`); Remove Parameter for a parameter that serves nothing (`techniques/simplifying-method-calls.md`). House rule 9: never add a guard for an unreachable case; outside the task, list dead code rather than delete it (rule 13).
**Ignore when:** Nothing on the site.

## Lazy Class
<https://refactoring.guru/smells/lazy-class>

**Signs:** A class (or subclass) that does too little to justify the effort of understanding and maintaining it.
**Why it hurts:** It may have shrunk after other changes, or have been built for features never written; every class costs reading time.
**Treatment:** Inline Class (`techniques/moving-features.md`); Collapse Hierarchy for a subclass with a few functions (`techniques/generalization.md`).
**Ignore when:** The near-empty class is a deliberate placeholder for planned work; weigh clarity against simplicity.

## Speculative Generality
<https://refactoring.guru/smells/speculative-generality>

**Signs:** An abstract class, a delegating class, a method, a parameter or a field that nothing uses, added for a future need.
**Why it hurts:** The anticipated feature never came, and the extra code makes the codebase harder to read and support.
**Treatment:** Unused abstract class — Collapse Hierarchy (`techniques/generalization.md`); pointless delegation — Inline Class; unused method — Inline Method (`techniques/moving-features.md`, `techniques/composing-methods.md`); unused parameter — Remove Parameter (`techniques/simplifying-method-calls.md`); unused field — delete it.
**Ignore when:** A framework offers a feature it does not use itself because its users need it. Before deleting, check whether tests rely on it — they sometimes reach into internals.
