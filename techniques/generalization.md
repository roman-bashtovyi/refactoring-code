# Generalization

Techniques for moving members up and down a class hierarchy, and for choosing between inheriting and holding another object. Direction in this skill: prefer holding an object over inheriting from it unless the class really is a kind of its parent and uses all of its interface.

## Contents
- Pull Up Field
- Pull Up Method
- Pull Up Constructor Body
- Push Down Field
- Push Down Method
- Extract Subclass
- Extract Superclass
- Extract Interface
- Collapse Hierarchy
- Form Template Method
- Replace Inheritance with Delegation
- Replace Delegation with Inheritance

## Pull Up Field
<https://refactoring.guru/pull-up-field>

**Signs:** Sibling subclasses each declare the same field (maybe under different names), and each carries its own copy of the code that handles it.
**Do:** Check that the field means the same thing in every subclass, and give them one name. Declare it once in the parent (`protected` if it was private), delete it from the subclasses. Wrap it in accessors afterwards if needed: Self Encapsulate Field (organizing-data.md).
**When not:** The same name hides different meanings. Only some subclasses use the field — it belongs to those (Push Down Field).
**Inverse:** Push Down Field.

## Pull Up Method
<https://refactoring.guru/pull-up-method>

**Signs:** Sibling subclasses with methods that do the same job, copied or differing only in names; an override that repeats what the parent already does (Duplicate Code, smells/dispensables.md).
**Do:** Make the copies identical first: same name, same parameters, same body apart from parts that truly vary. If the body leans on members only the subclasses have, pull those up too or declare them abstract in the parent (which then becomes abstract). Move one copy up, delete the rest, then switch callers to the parent type where they can.
**When not:** The copies differ in a step or in the order of steps — Form Template Method. Only one subclass has it — leave it there.
**Inverse:** Push Down Method.

## Pull Up Constructor Body
<https://refactoring.guru/pull-up-constructor-body>

**Signs:** Subclass constructors open with the same assignments.
**Do:** Reorder each subclass constructor so the shared lines come first. Write a parent constructor that takes only the parameters the shared setup needs; every subclass calls it as its first statement and keeps its own setup after. Unlike Pull Up Method, a constructor is not just moved: each subclass must call the parent one first, and it may take other parameters than the subclass does.
**When not:** The shared lines sit in the middle or depend on values only a subclass has. Only one subclass exists.

## Push Down Field
<https://refactoring.guru/push-down-field>

**Signs:** A field in the parent that one or two subclasses use; the rest carry data they never read (a planned feature that never came, or a part of the hierarchy moved away).
**Do:** Add the field to every subclass that uses it, then remove it from the parent — in that order, so nothing breaks between the steps. Move the methods that use it down as well (Push Down Method).
**When not:** Many subclasses will use it the same way — the copies would only duplicate it; keep it in the parent or an intermediate class.
**Inverse:** Pull Up Field.

## Push Down Method
<https://refactoring.guru/push-down-method>

**Signs:** A method in the parent that one subclass, or a few, use; the others inherit it and must never call it; callers check the type before calling.
**Do:** Copy the method into the subclasses that need it, delete it from the parent, then check that each call site goes through a variable of the subclass type — the step that is easy to skip. If several but not all subclasses use it, give them a new intermediate class instead of copying.
**When not:** Callers hold the parent type and the method is part of its contract.
**Inverse:** Pull Up Method.

## Extract Subclass
<https://refactoring.guru/extract-subclass>

**Signs:** A class with fields or methods that matter only in a special case — an optional feature behind a flag, fields empty most of the time (Temporary Field, smells/oo-abusers.md), `if` on a kind inside its methods.
**Do:** A subclass for the special case, with a constructor that takes the extra data and passes the rest up. Switch the creation sites to it. Push methods down first, then fields. Finally drop the flag that chose between behaviours and replace its conditionals with overrides: Replace Conditional with Polymorphism (simplifying-conditionals.md).
**When not:** The class varies along two independent axes — an object cannot join two sibling subclasses; hold the second axis as an object (Replace Inheritance with Delegation, this file). The behaviour must change while the object lives — Replace Type Code with State/Strategy (organizing-data.md).

## Extract Superclass
<https://refactoring.guru/extract-superclass>

**Signs:** Two classes with the same fields and methods, or the same ones in slightly different form, and no common parent (Duplicate Code, smells/dispensables.md).
**Do:** An abstract parent. Move fields up first, then methods — the methods use the fields. Then, in client code, type variables and parameters as the parent where only the shared part is used.
**When not:** One of the classes already has a parent. The classes share a few lines, not a role — Extract Class (moving-features.md) and hold it.

## Extract Interface
<https://refactoring.guru/extract-interface>

**Signs:** Several clients use the same few methods of a class; two classes offer the same methods and callers treat them alike; a client must be run against a fake or another implementation.
**Do:** Declare an interface with exactly the shared methods, make the classes implement it, and type parameters and variables as the interface wherever only those methods are used.
**When not:** One implementation and no reason to swap it (Speculative Generality, smells/dispensables.md). An interface shares signatures, not code: duplicated bodies stay duplicated — Extract Superclass or Extract Class (moving-features.md).

## Collapse Hierarchy
<https://refactoring.guru/collapse-hierarchy>

**Signs:** A subclass that adds almost nothing to its parent after features were moved or removed; an abstract parent with one child (Lazy Class and Speculative Generality, smells/dispensables.md).
**Do:** Pick the class that is easier to lose. Keeping the parent: pull the subclass's members up. Keeping the subclass: push the parent's down into it. Replace every mention of the removed class (creation, type declarations, comments), then delete it. With several subclasses, re-parent the rest to the survivor only where they still fit. It is Inline Class (moving-features.md) for a parent and child.
**When not:** The subclass is a marker that other code tests for (`instanceof`) or that a framework looks up by class.

## Form Template Method
<https://refactoring.guru/form-template-method>

**Signs:** Sibling subclasses run the same algorithm — the same steps in the same order — and differ in some steps; a fix lands in one sibling and not in the others.
**Do:** Split each subclass's algorithm into one method per step. Move the identical steps up; give the differing ones the same name and signature in every subclass and declare them abstract in the parent. Move the algorithm method up last: it calls concrete and abstract steps. Make it `final` when subclasses must not reorder the steps.
**When not:** The subclasses differ in the number or order of steps. Only one step varies and there is no hierarchy yet — pass the step in as a callback or an object, which is lighter.

## Replace Inheritance with Delegation
<https://refactoring.guru/replace-inheritance-with-delegation>

**Signs:** A subclass that uses little of what it inherits, overrides methods to disable them, or exposes parent methods no one should call (Refused Bequest, smells/oo-abusers.md); inheritance chosen for reuse, not because the class is a kind of its parent.
**Do:** Add a field typed as the former parent and at first assign `$this` to it, so nothing changes. Route the class's own calls through the field. Write forwarding methods only for the inherited methods outside code really calls, remove `extends`, and finally put a new instance of the former parent into the field.
**When not:** The class is a true specialisation and uses the whole parent interface — forwarding all of it by hand is worse (Replace Delegation with Inheritance, this file).
**Inverse:** Replace Delegation with Inheritance.

## Replace Delegation with Inheritance
<https://refactoring.guru/replace-delegation-with-inheritance>

**Signs:** A class whose methods mostly forward to one other object, a trivial forwarder for each of its public methods (Middle Man, smells/couplers.md).
**Do:** Make the class extend the target. First hold `$this` in the delegate field so users of the field still work, then remove the forwarders one by one (rename those whose names differ, so they match), point the field's uses at `$this`, delete the field.
**When not:** It forwards only some of the target's public methods — inheriting the rest breaks substitutability. It already has a parent. The delegate can change at run time or is shared with others.
**Inverse:** Replace Inheritance with Delegation.
