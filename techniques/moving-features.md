# Moving features

Techniques for putting fields and methods in the class that uses them, and for deciding how much one class shows of another. Two pairs pull opposite ways: Extract Class and Inline Class, Hide Delegate and Remove Middle Man.

## Contents
- Move Method
- Move Field
- Extract Class
- Inline Class
- Hide Delegate
- Remove Middle Man
- Introduce Foreign Method
- Introduce Local Extension

## Move Method
<https://refactoring.guru/move-method>

**Signs:** A method works mostly with another class's data or calls (Feature Envy, see smells/couplers.md); a change in class A keeps forcing an edit in a method of B.
**Do:** List what the method uses in its own class: a feature only it uses moves along, a shared one may drag more methods with it. Check no subclass overrides it. Write it in the target class under a name that fits there, make the target reachable from the old place, then update the callers and delete the old method.
**When not:** The method is declared in a superclass or overridden in a subclass — moving it then needs polymorphism in the target, or no move at all. Moving drags a whole group of methods — that is Extract Class. A forwarding method left behind for good is a wrapper (house rule 8).

## Move Field
<https://refactoring.guru/move-field>

**Signs:** A field read and written mostly by methods of another class; getters on it called from everywhere but its own class. Usually found in the middle of Extract Class.
**Do:** If it is public, hide it behind accessors first (Encapsulate Field in organizing-data.md). Create it in the target, decide how the old class reaches the target, send every access there, delete the old field. A non-private field also needs its subclasses and superclasses updated.
**When not:** Both classes use it about equally: move the field only together with the methods that read it, or leave it where most of them are.

## Extract Class
<https://refactoring.guru/extract-class>

**Signs:** A cluster of fields only one group of methods touches; names sharing a prefix (`address*`, `phone*`); a class that changes for two unrelated reasons; the same group of fields in a second class (Data Clumps).
**Do:** Decide the split before moving anything. Create the class, link old to new one way only, move fields first and then the methods using them, one at a time, checking after each. Rename the old class if its role narrowed. Callers that now reach through (`$student->address->label()`) may call for Hide Delegate.
**Before / after (PHP):**
```php
// Before
final class Student
{
    public function __construct(
        public readonly string $name,
        public readonly string $street,
        public readonly string $city,
        public readonly string $postalCode,
    ) {}

    public function addressLabel(): string
    {
        return "$this->street, $this->postalCode $this->city";
    }
}

// After
final class Address
{
    public function __construct(
        public readonly string $street,
        public readonly string $city,
        public readonly string $postalCode,
    ) {}

    public function label(): string
    {
        return "$this->street, $this->postalCode $this->city";
    }
}

final class Student
{
    public function __construct(
        public readonly string $name,
        public readonly Address $address,
    ) {}
}
```
**When not:** The new class would hold one field and no behaviour (Lazy Class, smells/dispensables.md). If the new class needs a link back to the old one, the split is in the wrong place.
**Inverse:** Inline Class, when a class has been emptied.

## Inline Class
<https://refactoring.guru/inline-class>

**Signs:** A class left with a few fields and little behaviour after other techniques moved things out; a name that no longer says what it does; no growth planned.
**Do:** Mirror its public features on the receiving class as forwarders, point every user at the receiver, then Move Method and Move Field until the old class is empty and delete it.
**When not:** The class marks a boundary on purpose (a value object, a transfer object at a layer edge, a seam tests rely on), or its responsibility is about to return.
**Inverse:** Extract Class.

## Hide Delegate
<https://refactoring.guru/hide-delegate>

**Signs:** A client writes `$a->b()->c()`, and `b()` exists only to hand over an object to ask; the client breaks when the structure behind `a` changes (Message Chains, smells/couplers.md).
**Do:** Add a method to the server for each delegate method the client calls, named for what the client wants (`curatorEmail()`), not for the path. Switch the clients. When none uses the getter any more, delete it so the chain cannot come back.
**Before / after (PHP):**
```php
// Before
final class Group
{
    public function __construct(
        private readonly Teacher $curator,
    ) {}

    public function curator(): Teacher
    {
        return $this->curator;
    }
}

function curatorMailto(Group $group): string
{
    return 'mailto:' . $group->curator()->email;
}

// After
final class Group
{
    public function __construct(
        private readonly Teacher $curator,
    ) {}

    public function curatorEmail(): string
    {
        return $this->curator->email;
    }
}

function curatorMailto(Group $group): string
{
    return 'mailto:' . $group->curatorEmail();
}
```
**When not:** House rule 8 still holds: a forwarder is for cutting a chain through another object, not for renaming a call on the same object. Many forwarders turn the server into a Middle Man. A fluent chain on one value (`$date->modify(…)->format(…)`) is not a delegate chain.
**Inverse:** Remove Middle Man, when forwarders pile up and the server adds nothing.

## Remove Middle Man
<https://refactoring.guru/remove-middle-man>

**Signs:** Most methods of a class are one-line forwards to another object; every new method of the target needs a twin here; the class does no work of its own.
**Do:** Give clients access to the target through a getter, switch them to call it directly, delete the forwarders as they lose callers.
**When not:** The forwarder cuts a chain clients should not know about (Hide Delegate doing its job); a couple of forwards keep a layer honest.
**Inverse:** Hide Delegate.

## Introduce Foreign Method
<https://refactoring.guru/introduce-foreign-method>

**Signs:** The same few lines operate on an object from a library or the framework everywhere, and you wish the class had a method for it but cannot edit it.
**Do:** Write the method in the class that needs it with the library object as its first argument, mark it in a short comment as a foreign method and say where it should live if the library allowed it (house rule 7: it explains why). Replace the repeated lines with the call.
**When not:** Two or more such methods, or other classes need them — Introduce Local Extension. PHP and TS have no extension methods, so this is a plain function or a method of the client class.

## Introduce Local Extension
<https://refactoring.guru/introduce-local-extension>

**Signs:** Several foreign methods for the same library class; every user of that class needs the same additions.
**Do:** Make a subclass, or a wrapper holding the library object when subclassing is blocked. Give it a constructor taking the library object, move the foreign methods in, switch the users that need them to the extension.
**When not:** One or two methods are enough (Introduce Foreign Method). A `final` class cannot be subclassed; a wrapper must forward the original's whole public interface, which is a Middle Man in the making.
