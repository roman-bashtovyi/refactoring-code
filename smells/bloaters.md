# Bloaters

Code that grew too big to take in at once. Entries: Long Method, Large Class, Primitive Obsession, Long Parameter List, Data Clumps.

## Long Method
<https://refactoring.guru/smells/long-method>

**Signs:** A method past roughly ten lines; blank-line-separated blocks, each introduced by a comment; many locals alive at once; a loop whose body is a second algorithm.
**Why it hurts:** Lines get added to a method far more often than removed, so it becomes the place where logic and duplicates hide.
**Treatment:** Extract Method for a block that needs a comment to be understood, loops included; Replace Temp with Query or Replace Method with Method Object when locals block the extraction (`techniques/composing-methods.md`); Decompose Conditional for a heavy condition (`techniques/simplifying-conditionals.md`); Introduce Parameter Object or Preserve Whole Object when parameters block it (`techniques/simplifying-method-calls.md`). House rule 8 narrows the site: extract only when the name says what the code does not — never a method that wraps an existing one-line call.
**Ignore when:** Nothing on the site.

## Large Class
<https://refactoring.guru/smells/large-class>

**Signs:** Many fields, many methods, many lines; a name that needs "And" or "Manager"; groups of fields that only some methods touch.
**Why it hurts:** Adding to an existing class is easier than creating a new one, so classes only grow, and the reader must hold too many attributes at once.
**Treatment:** Extract Class to move one responsibility out (`techniques/moving-features.md`); Extract Subclass for behaviour needed rarely or in variants, Extract Interface to name what clients may use (`techniques/generalization.md`); Duplicate Observed Data when a UI class holds domain data (`techniques/organizing-data.md`).
**Ignore when:** Nothing on the site.

## Primitive Obsession
<https://refactoring.guru/smells/primitive-obsession>

**Signs:** Strings or numbers standing in for money, ranges, phone numbers; constants standing in for categories; text keys used to pull values out of an array that acts as an object; bare `0`/`1` or booleans in calls (house rule 10).
**Why it hurts:** A new field is quicker than a new class, so the host class swells, and the set of allowed values is scattered across the code.
**Treatment:** Replace Data Value with Object, Replace Type Code with Class, Replace Type Code with Subclasses, Replace Type Code with State/Strategy, Replace Array with Object, Replace Magic Number with Symbolic Constant (`techniques/organizing-data.md`); Introduce Parameter Object or Preserve Whole Object for parameters (`techniques/simplifying-method-calls.md`).
**Ignore when:** Nothing on the site.

## Long Parameter List
<https://refactoring.guru/smells/long-parameter-list>

**Signs:** A signature past three or four parameters; a flag parameter that picks which branch runs (house rule 10).
**Why it hurts:** Hard to read, easy to call in the wrong order, and it keeps growing as each new need adds a slot.
**Treatment:** Replace Parameter with Method Call when an argument is only the result of asking another object the method can already reach; Preserve Whole Object when several values come out of one object; Introduce Parameter Object when they come from different sources; Replace Parameter with Explicit Methods for a flag (all in `techniques/simplifying-method-calls.md`).
**Ignore when:** Removing the parameter would make the class depend on something it should not know about.

## Data Clumps
<https://refactoring.guru/smells/data-clumps>

**Signs:** The same group of variables appears together in several signatures or classes, like the settings for one connection. Test: delete one value from the group — if the rest stop making sense, they belong together.
**Why it hurts:** Born of copy-paste and weak structure; every operation on the group is scattered across the code.
**Treatment:** Extract Class when the clump is fields (`techniques/moving-features.md`); Introduce Parameter Object when it is parameters, Preserve Whole Object when only part of it is passed on (`techniques/simplifying-method-calls.md`); then move the code that works on the clump into the new class.
**Ignore when:** Passing the whole object would create a dependency between classes that you do not want.
