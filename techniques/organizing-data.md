# Organizing data

Techniques for giving data a shape the reader can see: named fields instead of positions, objects instead of bare strings, closed sets instead of loose codes, hidden state behind methods.

## Contents
- Change Value to Reference
- Change Reference to Value
- Duplicate Observed Data
- Self Encapsulate Field
- Replace Data Value with Object
- Replace Array with Object
- Change Unidirectional Association to Bidirectional
- Change Bidirectional Association to Unidirectional
- Encapsulate Field
- Encapsulate Collection
- Replace Magic Number with Symbolic Constant
- Replace Type Code with Class
- Replace Type Code with Subclasses
- Replace Type Code with State/Strategy
- Replace Subclass with Fields

## Change Value to Reference
<https://refactoring.guru/change-value-to-reference>

**Signs:** Many equal instances of one class stand for one real thing (a user, a group) and drift apart when one is edited.
**Do:** Turn the constructor into a factory method, keep the instances in one registry or repository that hands out the shared one, decide whether they load up front or on first use, and make the factory look up before it creates.
**When not:** The object is small and never changes — that is a value. The ORM already gives entities identity.
**Inverse:** Change Reference to Value.

## Change Reference to Value
<https://refactoring.guru/change-reference-to-value>

**Signs:** A small object fetched from a registry only to be read; nobody needs its identity; the lookup costs more than the object is worth.
**Do:** Make it immutable (no setters, values set in the constructor), compare by content, then see whether the factory method can go and the constructor become public.
**When not:** The value still changes and every holder must see the change; copies would go stale.
**Inverse:** Change Value to Reference.

## Duplicate Observed Data
<https://refactoring.guru/duplicate-observed-data>

**Signs:** Domain data lives in a class that also builds the screen, and a second screen (mobile, report) needs the same data.
**Do:** Move the data into a domain class and keep a copy in the screen class in sync through observers. Route all access through accessors first, and have each side write straight to its own fields when it is notified, or the updates loop.
**When not:** Code rebuilt on every request (most PHP): the observer form does not survive between requests, so pull the domain logic into a service instead.

## Self Encapsulate Field
<https://refactoring.guru/self-encapsulate-field>

**Signs:** A class reads and writes its own private fields directly, and now needs lazy loading, validation, or a subclass that changes the value.
**Do:** Add a getter (and a setter only if it must change), make them protected or public so subclasses can override, and send every read and write inside the class through them.
**When not:** Nothing needs the hook; direct access in a small class is easier to read. PHP 8.4 property hooks cover the same need with no accessor methods.

## Replace Data Value with Object
<https://refactoring.guru/replace-data-value-with-object>

**Signs:** A string or number with rules around it — normalising, validating, comparing, formatting — repeated at every use (`strtolower(trim($email))` in three places).
**Do:** A small immutable class; the rules move into it; the owner stores the object, and a change means a new instance. If the owner touches the field in many places, wrap those in accessors first. Normalising in the constructor changes what a plain read returns — check nobody needs the raw text. Add validation as a separate, deliberate step (house rule 12: behaviour unchanged).
**Before / after (PHP):**
```php
// Before
final class Teacher
{
    public function __construct(
        public readonly string $email,
    ) {}
}

function isDuplicate(Teacher $first, Teacher $second): bool
{
    return strtolower(trim($first->email)) === strtolower(trim($second->email));
}

function mailto(Teacher $teacher): string
{
    return 'mailto:' . strtolower(trim($teacher->email));
}

// After
final class Email
{
    public readonly string $address;

    public function __construct(string $address)
    {
        $this->address = strtolower(trim($address));
    }

    public function equals(Email $other): bool
    {
        return $this->address === $other->address;
    }
}

final class Teacher
{
    public function __construct(
        public readonly Email $email,
    ) {}
}

function isDuplicate(Teacher $first, Teacher $second): bool
{
    return $first->email->equals($second->email);
}

function mailto(Teacher $teacher): string
{
    return 'mailto:' . $teacher->email->address;
}
```
**When not:** A plain label with no rule (a class per string is a Lazy Class, smells/dispensables.md); no rule repeats yet.

## Replace Array with Object
<https://refactoring.guru/replace-array-with-object>

**Signs:** An associative array passed through several functions and read by string key in many places; a tuple whose positions carry meaning; a mistyped key fails quietly (`$row['emial']`); the behaviour on that data sits in helpers elsewhere.
**Do:** A class with a typed property per key (`readonly`). Build it in one place — the query row, the request, the decoded JSON. Move callers over one key at a time, then delete the array and move the helpers that work on it into the class.
**Before / after (PHP):**
```php
// Before
function mailLabel(array $student): string
{
    return $student['last_name'] . ' ' . $student['first_name'] . ' <' . $student['email'] . '>';
}

// After
final class StudentContact
{
    public function __construct(
        public readonly string $lastName,
        public readonly string $firstName,
        public readonly string $email,
    ) {}

    public function mailLabel(): string
    {
        return "$this->lastName $this->firstName <$this->email>";
    }
}
```
**Before / after (TS):** an object literal with an interface is already the replacement; the smell is the positional tuple.
```ts
// Before
function hoursLabel(entry: [string, number, boolean]): string {
  const mark = entry[2] ? ' (approved)' : '';

  return `${entry[0]}: ${entry[1]} h${mark}`;
}

// After
interface HoursEntry {
  subject: string;
  hours: number;
  approved: boolean;
}

function hoursLabel({ subject, hours, approved }: HoursEntry): string {
  const mark = approved ? ' (approved)' : '';

  return `${subject}: ${hours} h${mark}`;
}
```
**When not:** A list of same-kind items (ids) or a map id → name stays an array; so does an array that never leaves its function.

## Change Unidirectional Association to Bidirectional
<https://refactoring.guru/change-unidirectional-association-to-bidirectional>

**Signs:** Class B needs to reach A, but only A holds the link, so callers search for the owner.
**Do:** Pick the dominant side that owns creating and changing the link, add the back-reference on the other side, and set it from the dominant side's mutators through a helper whose name says it is only for that, so nobody else calls it.
**When not:** The reverse link can be asked for on demand (a query) — prefer that. Mutual links tie the two classes together: neither can be used alone.
**Inverse:** Change Bidirectional Association to Unidirectional.

## Change Bidirectional Association to Unidirectional
<https://refactoring.guru/change-bidirectional-association-to-unidirectional>

**Signs:** Two classes hold each other, but one never uses the link; both must be created and released together; a change in one ripples to the other.
**Do:** Confirm the second class can get the object by parameter or query. Switch its users to that, stop assigning the link, then delete the field.
**When not:** Both directions are really used.
**Inverse:** Change Unidirectional Association to Bidirectional.

## Encapsulate Field
<https://refactoring.guru/encapsulate-field>

**Signs:** A public property written from anywhere, so no check is possible; logic for the data lives in its callers.
**Do:** Add the accessors, switch every read and write to them, only then make the field private. Afterwards look at the callers: logic around the call often belongs in the accessor.
**When not:** A `public readonly` property of a small immutable object cannot change, so it needs no getter. A hot path where the call cost is measured to matter.

## Encapsulate Collection
<https://refactoring.guru/encapsulate-collection>

**Signs:** A public array property, or a getter and setter on a list as if it were one value; callers that fetch the list, change it and put it back; the same "already in there?" check at every call site.
**Do:** Make the property private. Add methods to add and remove items that carry the rules. The getter must not let callers change the stored list; the setter that swaps everything is renamed for what it does or removed. Move the logic from callers into the owner. In PHP an array is copied on return, so the getter is safe for the list itself (objects inside are still shared); in TS an array is a reference, so return `readonly T[]` or a copy.
**Before / after (PHP):**
```php
// Before
final class Journal
{
    public array $teacherIds = [];
}

// every caller repeats the check, and one of them may forget it
if (!in_array($userId, $journal->teacherIds, true)) {
    $journal->teacherIds[] = $userId;
}

// After
final class Journal
{
    private array $teacherIds = [];

    public function addTeacher(int $userId): void
    {
        if (in_array($userId, $this->teacherIds, true)) {
            return;
        }

        $this->teacherIds[] = $userId;
    }

    public function teacherIds(): array
    {
        return $this->teacherIds;
    }
}

$journal->addTeacher($userId);
```
**Before / after (TS):**
```ts
// Before
class Journal {
  teacherIds: number[] = [];
}

// After
class Journal {
  readonly #teacherIds: number[] = [];

  addTeacher(userId: number): void {
    if (this.#teacherIds.includes(userId)) {
      return;
    }

    this.#teacherIds.push(userId);
  }

  get teacherIds(): readonly number[] {
    return this.#teacherIds;
  }
}
```
**When not:** A list that is plain data with no rule about its contents and a single writer.

## Replace Magic Number with Symbolic Constant
<https://refactoring.guru/replace-magic-number-with-symbolic-constant>

**Signs:** A literal in a comparison or arithmetic whose meaning needs the surrounding code (`>= 4`, `* 45`); the same literal in several places; a bare `0`/`1` or boolean in a call (house rule 10).
**Do:** A constant named for the meaning, not the value (`PASSING_GRADE`, not `FOUR`). Check each occurrence alone: another `4` may mean something else and gets its own name. A flag in a call is not fixed by a constant — use a named factory or one method per case (house rule 10); a closed set of values becomes a backed enum.
**Before / after (PHP):**
```php
// Before
final class Grading
{
    public function isPassing(int $grade): bool
    {
        return $grade >= 4;
    }

    public function endMinute(int $startMinute, int $hours): int
    {
        return $startMinute + $hours * 45;
    }
}

// After
final class Grading
{
    private const PASSING_GRADE = 4;
    private const ACADEMIC_HOUR_MINUTES = 45;

    public function isPassing(int $grade): bool
    {
        return $grade >= self::PASSING_GRADE;
    }

    public function endMinute(int $startMinute, int $hours): int
    {
        return $startMinute + $hours * self::ACADEMIC_HOUR_MINUTES;
    }
}
```
**When not:** The meaning is obvious in place (`0` as a start index, `1` as a step). A number that is really a type code steering behaviour belongs to Replace Type Code with Class and its siblings below.

## Replace Type Code with Class
<https://refactoring.guru/replace-type-code-with-class>

**Signs:** A field holding one of a fixed set of ints or strings that is only stored, compared or shown, never steering an `if` or `switch`; its setter accepts anything.
**Do:** A class with a private value and one named case per code (a backed enum in PHP 8.1+, a union of literals in TS). The field takes the new type, incoming values are wrapped, the old constants go.
**When not:** The code steers behaviour — Replace Type Code with Subclasses, Replace Type Code with State/Strategy, or Replace Conditional with Polymorphism (simplifying-conditionals.md). A `match` over an enum is fine for a few branches.

## Replace Type Code with Subclasses
<https://refactoring.guru/replace-type-code-with-subclasses>

**Signs:** A code field that conditionals test to pick behaviour, and that never changes during the object's life.
**Do:** Put a getter on the field. Make the constructor private and add a factory method that picks the subclass by code (the one large conditional that stays). One subclass per code overriding the getter, push fields and methods down, then Replace Conditional with Polymorphism (simplifying-conditionals.md).
**When not:** The class already has a parent, or the code changes while the object lives — Replace Type Code with State/Strategy. A few branches over an enum are simpler.
**Inverse:** Replace Subclass with Fields.

## Replace Type Code with State/Strategy
<https://refactoring.guru/replace-type-code-with-state-strategy>

**Signs:** As above, but the code changes during the object's life, or the class already inherits from a parent.
**Do:** Getter first. An abstract state class with one subclass per code and a factory mapping code to state; the field holds a state and its setter asks the factory. Push the behaviour down, then Replace Conditional with Polymorphism (simplifying-conditionals.md).
**When not:** A code that steers nothing — Replace Type Code with Class; adding a class per code is overkill there.

## Replace Subclass with Fields
<https://refactoring.guru/replace-subclass-with-fields>

**Signs:** Subclasses that differ only in methods returning constants.
**Do:** Turn the subclass constructors into factory methods on the parent (Replace Constructor with Factory Method, simplifying-method-calls.md) and have callers use those. Give the parent a protected constructor and fields for the values, pass each subclass's values up, fold a subclass constructor that does more into the factory with Inline Method, delete the subclasses.
**When not:** The subclasses carry behaviour that differs, not just constants.
**Inverse:** Replace Type Code with Subclasses.
