# Couplers

Classes tied together too tightly, or the opposite: delegation taken too far. Entries: Feature Envy, Inappropriate Intimacy, Incomplete Library Class, Message Chains, Middle Man. Message Chains and Middle Man are opposite ends: curing the first too eagerly produces the second.

## Feature Envy
<https://refactoring.guru/smells/feature-envy>

**Signs:** A method reaches into another object's data far more than it uses its own.
**Why it hurts:** Often left behind when data moved into a data-holding class but the logic stayed; data and the code on it normally change together.
**Treatment:** Move Method when it clearly belongs elsewhere; Extract Method for the part that touches the other object, then move that part; for a method using several classes, place it where most of its data lives, or split it and distribute the pieces (`techniques/moving-features.md`, `techniques/composing-methods.md`).
**Ignore when:** Behaviour is kept apart from data on purpose so it can be swapped at runtime, as Strategy or Visitor do.

## Inappropriate Intimacy
<https://refactoring.guru/smells/inappropriate-intimacy>

**Signs:** One class digs into another's internal fields and methods instead of using its public interface.
**Why it hurts:** Classes that interact this much become tangled; they are harder to maintain and reuse.
**Treatment:** Move Method and Move Field to the class that actually uses them, if the original does not need them; Extract Class and Hide Delegate to put the contact behind a formal access point (`techniques/moving-features.md`); Change Bidirectional Association to Unidirectional when each depends on the other (`techniques/organizing-data.md`); Replace Delegation with Inheritance when the closeness is between subclass and superclass (`techniques/generalization.md`).
**Ignore when:** Nothing on the site.

## Incomplete Library Class
<https://refactoring.guru/smells/incomplete-library-class>

**Signs:** A third-party class lacks something the project needs, and you cannot edit it.
**Why it hurts:** The library authors never built the feature or chose not to, and the gap pushes you to rewrite what a working library nearly offers.
**Treatment:** Introduce Foreign Method for a few helper methods; Introduce Local Extension — your own wrapper or subclass — for larger additions (`techniques/moving-features.md`).
**Ignore when:** Extending the library would create extra work, because later library changes force changes in your extension.

## Message Chains
<https://refactoring.guru/smells/message-chains>

**Signs:** A long run of linked calls, each result used to reach the next object: `$a->b()->c()->d()`.
**Why it hurts:** The client depends on how objects are wired together, so a change to any link breaks it.
**Treatment:** Hide Delegate, so the client calls one direct method; or Extract Method for what the client needs from the last object, then Move Method to the start of the chain (`techniques/moving-features.md`, `techniques/composing-methods.md`).
**Ignore when:** Hiding too many delegates makes it unclear where the real work happens and turns into Middle Man. Own note: a fluent interface returning the same object (`$query->condition()->range()`) is not a chain through other objects.

## Middle Man
<https://refactoring.guru/smells/middle-man>

**Signs:** A class most of whose methods only pass the call to another class.
**Why it hurts:** Left over after cutting chains too far, or after a class lost its real duties; it is an empty shell. House rule 8 is the same thing at method level: no wrapper over an existing one-line call.
**Treatment:** Remove Middle Man where most of the calls simply forward (`techniques/moving-features.md`).
**Ignore when:** The intermediary keeps two classes from depending on each other, or the delegation is the point, as in Proxy and Decorator.
