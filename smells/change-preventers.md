# Change Preventers

Code where one change drags many edits behind it. Entries: Divergent Change, Parallel Inheritance Hierarchies, Shotgun Surgery. Divergent Change and Shotgun Surgery are opposites: one class changed for many reasons, one reason changing many classes. Curing the first by over-splitting causes the second.

## Divergent Change
<https://refactoring.guru/smells/divergent-change>

**Signs:** One class is edited in several unrelated methods whenever it changes — a new kind of product touches the code that finds, shows and orders products.
**Why it hurts:** The class has several reasons to change, a result of weak structure or copy-paste.
**Treatment:** Extract Class, one per responsibility (`techniques/moving-features.md`); where different classes share behaviour, Extract Superclass and Extract Subclass (`techniques/generalization.md`).
**Ignore when:** Nothing on the site.

## Parallel Inheritance Hierarchies
<https://refactoring.guru/smells/parallel-inheritance-hierarchies>

**Signs:** Adding a subclass in one hierarchy forces a matching subclass in another.
**Why it hurts:** Tolerable while both are small, but each new class doubles the work.
**Treatment:** In two steps: let objects of one hierarchy hold a reference to objects of the other, then dissolve the second hierarchy with Move Method and Move Field (`techniques/moving-features.md`).
**Ignore when:** Merging the hierarchies would leave the code messier or cause a bigger architectural trouble — undo it and keep the structure.

## Shotgun Surgery
<https://refactoring.guru/smells/shotgun-surgery>

**Signs:** One ordinary change becomes many small edits across many unrelated classes.
**Why it hurts:** A single responsibility is smeared over several classes, often after splitting too aggressively to cure Divergent Change.
**Treatment:** Move Method and Move Field to gather the behaviour into one class, creating a new one if none fits; Inline Class for the classes left nearly empty (`techniques/moving-features.md`).
**Ignore when:** Nothing on the site.
