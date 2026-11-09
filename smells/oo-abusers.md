# Object-Orientation Abusers

Object-oriented tools used wrongly, or not used where they fit. Entries: Alternative Classes with Different Interfaces, Refused Bequest, Switch Statements, Temporary Field.

## Alternative Classes with Different Interfaces
<https://refactoring.guru/smells/alternative-classes-with-different-interfaces>

**Signs:** Two classes do the same job under different method names.
**Why it hurts:** Usually one author did not know the other class existed; the reader must work out why there are two.
**Treatment:** Rename Method, Add Parameter and Parameterize Method until the signatures match (`techniques/simplifying-method-calls.md`); Move Method to line up the behaviour (`techniques/moving-features.md`); Extract Superclass when only part of the job overlaps (`techniques/generalization.md`). Afterwards one of the classes may be deletable.
**Ignore when:** Merging is impractical or pointless, for example when the two classes come from different libraries, each with its own version.

## Refused Bequest
<https://refactoring.guru/smells/refused-bequest>

**Signs:** A subclass uses only a fraction of what it inherits; the rest sits unused or is overridden to throw or do nothing.
**Why it hurts:** Inheritance was picked to reuse code, not because the classes are related, so the hierarchy reads as nonsense (a report extending a logger).
**Treatment:** Replace Inheritance with Delegation when nothing real is shared; if the hierarchy is right, Extract Superclass — move the shared members into a new parent and let both classes inherit from it (`techniques/generalization.md`).
**Ignore when:** Nothing on the site.

## Switch Statements
<https://refactoring.guru/smells/switch-statements>

**Signs:** A long `switch`, `match` or `if`/`elseif` chain on a type or mode; the same chain repeated in several places, so a new case means finding every copy.
**Why it hurts:** The branching sits away from the data it depends on and multiplies with each new case.
**Treatment:** Extract Method, then Move Method to put the branching where it belongs (`techniques/composing-methods.md`, `techniques/moving-features.md`); a type code that changes at runtime — Replace Type Code with Subclasses or Replace Type Code with State/Strategy (`techniques/organizing-data.md`); then Replace Conditional with Polymorphism; a null branch — Introduce Null Object (`techniques/simplifying-conditionals.md`); a few branches calling one method with different arguments — Replace Parameter with Explicit Methods (`techniques/simplifying-method-calls.md`).
**Ignore when:** The switch only does simple actions, or it lives in factory code choosing which class to create. Own note: a `match` that just maps one value to another is a lookup table, not this smell.

## Temporary Field
<https://refactoring.guru/smells/temporary-field>

**Signs:** Fields that hold a meaningful value only in some situations and are empty otherwise; code that checks whether a field is set before using it.
**Why it hurts:** The inputs of one algorithm were stored on the object instead of being passed around, so the reader meets empty fields and cannot tell why they exist.
**Treatment:** Extract Class — move the fields and the code that uses them into their own class, which is the same result as Replace Method with Method Object (`techniques/moving-features.md`, `techniques/composing-methods.md`); Introduce Null Object to replace the "is it set" checks (`techniques/simplifying-conditionals.md`).
**Ignore when:** Nothing on the site.
