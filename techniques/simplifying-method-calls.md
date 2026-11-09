# Simplifying method calls

Techniques for making a call say what it does: a name that tells the truth, a signature that carries only what the method needs, a failure that cannot be mistaken for a value.

## Contents
- Add Parameter
- Remove Parameter
- Rename Method
- Separate Query from Modifier
- Parameterize Method
- Introduce Parameter Object
- Preserve Whole Object
- Remove Setting Method
- Replace Parameter with Explicit Methods
- Replace Parameter with Method Call
- Hide Method
- Replace Constructor with Factory Method
- Replace Error Code with Exception
- Replace Exception with Test

## Add Parameter
<https://refactoring.guru/add-parameter>

**Signs:** A change to a method needs data the method cannot reach.
**Do:** First ask whether the data belongs on the object or is reachable through objects the method already holds — then nothing is added. Otherwise copy the method with the new parameter (a placeholder default), make the old one forward to it, move callers over, delete the old one; a public API keeps it marked deprecated. Do the same in parents and children that define the method.
**When not:** The list is already long — parameters are easy to add and hard to remove (Long Parameter List, smells/bloaters.md). A flag that switches behaviour (house rule 10) — Replace Parameter with Explicit Methods (this file).
**Inverse:** Remove Parameter.

## Remove Parameter
<https://refactoring.guru/remove-parameter>

**Signs:** A parameter the body never reads, often added for a change that never came; callers computing a value only to hand it over.
**Do:** Check parents and children first: if any version of the method reads it, stop — the signature belongs to the family. Otherwise copy the method without it, make the old one forward, move callers, delete the old one (deprecate it if it is public).
**When not:** An interface or an override fixes the signature and another implementation uses the parameter.
**Inverse:** Add Parameter.

## Rename Method
<https://refactoring.guru/rename-method>

**Signs:** A name that says less than, or something other than, what the method does (`total()` that returns an average, `check()` that also saves, `process`, `handle`); a name that needs a comment; a name that differs from the word the UI or the error message uses (house rule 6).
**Do:** Name it for what it does, in domain words. Rename in every class of the hierarchy that defines it. For a public API add the new name, make the old one a deprecated forwarder, move callers, delete it later. A text search for the old name also hits unrelated methods of the same name: use the IDE rename, or check each hit.
**Before / after (PHP):**
```php
// Before
function total(array $grades): float
{
    return array_sum($grades) / count($grades);
}

// ...
return total($grades) > $level;

// After
function averageGrade(array $grades): float
{
    return array_sum($grades) / count($grades);
}

// ...
return averageGrade($grades) > $level;
```
**When not:** The name is fine and the change is taste. The method is a public API and the gain is small.

## Separate Query from Modifier
<https://refactoring.guru/separate-query-from-modifier>

**Signs:** A method both answers and changes state (`takeNext()`, a getter that creates or saves); asking twice gives different answers; a caller wants the answer without the change.
**Do:** Add a query that only returns, and leave the original as the pure modifier. Replace each call of the original with the query, and put the modifier call exactly where the old call stood, so the change still happens once even inside a condition or a loop. Only state others can see counts (public fields, the database, files); filling a private cache is not a modifier.
**Before / after (PHP):**
```php
// Before
public function takeNextLesson(array $lessonIds): ?int
{
    foreach ($lessonIds as $lessonId) {
        if (!in_array($lessonId, $this->sent, true)) {
            $this->sent[] = $lessonId;

            return $lessonId;
        }
    }

    return null;
}

// ...
$lessonId = $queue->takeNextLesson($lessonIds);

if ($lessonId !== null) {
    $reminded[] = $lessonId;
}

// After
public function nextLesson(array $lessonIds): ?int
{
    foreach ($lessonIds as $lessonId) {
        if (!in_array($lessonId, $this->sent, true)) {
            return $lessonId;
        }
    }

    return null;
}

public function markSent(int $lessonId): void
{
    $this->sent[] = $lessonId;
}

// ...
$lessonId = $queue->nextLesson($lessonIds);

if ($lessonId !== null) {
    $queue->markSent($lessonId);
    $reminded[] = $lessonId;
}
```
**When not:** The caller needs the outcome of the change in the same breath (the number of rows a delete removed), and a second call would repeat the work or race.

## Parameterize Method
<https://refactoring.guru/parameterize-method>

**Signs:** Two or more methods with the same body that differ only in a value: a status, a limit, a unit, a format.
**Do:** If only part of the bodies match, extract that part first. In the merged method turn the differing value into a parameter, move callers over with the right argument, and only then delete the old methods.
**Before / after (PHP):**
```php
// Before
function hasActiveGroup(array $groups): bool
{
    foreach ($groups as $group) {
        if ($group['status'] === 'active') {
            return true;
        }
    }

    return false;
}

function hasArchivedGroup(array $groups): bool
{
    foreach ($groups as $group) {
        if ($group['status'] === 'archived') {
            return true;
        }
    }

    return false;
}

// After
function hasGroupWithStatus(array $groups, string $status): bool
{
    foreach ($groups as $group) {
        if ($group['status'] === $status) {
            return true;
        }
    }

    return false;
}
```
**When not:** The parameter would become a flag choosing whole branches — that is Replace Parameter with Explicit Methods (this file). The cases only look alike today and will drift apart: two methods are the honest shape.
**Inverse:** Replace Parameter with Explicit Methods.

## Introduce Parameter Object
<https://refactoring.guru/introduce-parameter-object>

**Signs:** The same two or three parameters travel together through several signatures (`$start, $end`; `$lastName, $firstName`) — Data Clumps (smells/bloaters.md); the same checks on them repeated in each method.
**Do:** An immutable class holding them. Add it as an extra parameter first, then remove the old ones one at a time, reading from the object. Afterwards look for logic around the group that moves in with Move Method (moving-features.md); a class that stays pure data is a Data Class (smells/dispensables.md).
**Before / after (PHP):**
```php
// Before
function contains(string $start, string $end, string $date): bool
{
    return $date >= $start && $date <= $end;
}

function overlaps(string $start, string $end, string $otherStart, string $otherEnd): bool
{
    return $start <= $otherEnd && $otherStart <= $end;
}

// After
final class DateRange
{
    public function __construct(
        public readonly string $start,
        public readonly string $end,
    ) {}

    public function contains(string $date): bool
    {
        return $date >= $this->start && $date <= $this->end;
    }

    public function overlaps(DateRange $other): bool
    {
        return $this->start <= $other->end && $other->start <= $this->end;
    }
}
```
**When not:** One method uses the group, once. The values are already the fields of an object the callers hold — Preserve Whole Object (this file).

## Preserve Whole Object
<https://refactoring.guru/preserve-whole-object>

**Signs:** Call sites pull several values out of one object only to pass them on (`f($lesson->startMinute, $lesson->endMinute)`); each new need adds a parameter and touches every caller.
**Do:** Pass the object. Add the object parameter first, swap uses of the old parameters for reads from it one at a time, then drop the old parameters and the extraction at the callers.
**Before / after (PHP):**
```php
// Before
function slotLabel(int $startMinute, int $endMinute): string
{
    return clock($startMinute) . '-' . clock($endMinute);
}

// ...
$label = slotLabel($lesson->startMinute, $lesson->endMinute);

// After
function slotLabel(Lesson $lesson): string
{
    return clock($lesson->startMinute) . '-' . clock($lesson->endMinute);
}

// ...
$label = slotLabel($lesson);
```
**When not:** The function must also serve data that does not come from that class; taking the object ties it to the class. The values come from different objects — Introduce Parameter Object (this file).

## Remove Setting Method
<https://refactoring.guru/remove-setting-method>

**Signs:** A setter on a field that must not change after the object exists (an id, a creation date, a type); callers write `new X()` and call the setter on the next line.
**Do:** Make the constructor take the value (a promoted `readonly` property in PHP, `readonly` in TS), pass it where objects are created, assign it directly inside the class instead of through the setter, delete the setter.
**When not:** The value does change during the object's life, or a framework fills it after construction (hydration) — check before removing.

## Replace Parameter with Explicit Methods
<https://refactoring.guru/replace-parameter-with-explicit-methods>

**Signs:** A flag or a code parameter picks the branch and call sites pass a literal (`setApproval($id, true)`, house rule 10); the method is one big `if` or `match` over its argument; the call does not say what happens.
**Do:** One method per branch, named for it. Keep the original as a dispatcher until the last caller has moved, then delete it.
**Before / after (PHP):**
```php
// Before
public function setApproval(int $lessonId, bool $approved): void
{
    if ($approved) {
        $this->approved[$lessonId] = true;

        return;
    }

    unset($this->approved[$lessonId]);
}

// ...
$journal->setApproval(5, true);
$journal->setApproval(6, true);
$journal->setApproval(5, false);

// After
public function approve(int $lessonId): void
{
    $this->approved[$lessonId] = true;
}

public function unapprove(int $lessonId): void
{
    unset($this->approved[$lessonId]);
}

// ...
$journal->approve(5);
$journal->approve(6);
$journal->unapprove(5);
```
**When not:** The caller passes a variable, not a literal (`$request->approved`): the flag is data, and an `if` at the caller only rebuilds the dispatcher. The branches share most of their body and new values keep coming — Parameterize Method.
**Inverse:** Parameterize Method.

## Replace Parameter with Method Call
<https://refactoring.guru/replace-parameter-with-method-call>

**Signs:** A caller asks an object (or runs a query) and passes the answer to a method that could ask the same question itself; the argument is always that same expression.
**Do:** Check the query needs nothing but what the receiving method already reaches — not the caller's locals. Move the call inside the receiver (extract it first if it is more than one call), replace the parameter's uses with it, remove the parameter.
**When not:** The value will be needed from other places later, or the receiver should not know where it comes from. The argument really differs between callers.

## Hide Method
<https://refactoring.guru/hide-method>

**Signs:** A public or protected method that nobody outside the class (or its family) calls; getters and setters left from when the class only held data; so many public methods that the real entry points are lost.
**Do:** Give each method the narrowest visibility it works with: `private`, then `protected`. A static analyser or the test suite shows who calls what. An accessor that became private and is used only inside: read the field directly and delete it.
**When not:** A framework or a test reaches it by name or by reflection (callbacks, listeners); an interface requires it.

## Replace Constructor with Factory Method
<https://refactoring.guru/replace-constructor-with-factory-method>

**Signs:** A constructor with flag or mode parameters (`new Report(true, false)`) or one that does lookups and I/O beyond assigning; creation that must return a subclass or a shared instance; two creations with the same parameter types and different meanings.
**Do:** Wrap `new` in a static method named for the case (`Report::forTeacher()`, `Money::zero()`), move callers to it, make the constructor private, then move whatever is not construction out of the constructor into the factory. One named factory per case beats a flag (house rule 10).
**When not:** There is one way to create the object and the constructor already reads clearly.

## Replace Error Code with Exception
<https://refactoring.guru/replace-error-code-with-exception>

**Signs:** A method returns a sentinel for failure (`-1`, `false`, `null`, `0` meaning OK); every caller opens with `if ($code === ERROR)` and often passes it up; the success value and the error share one type; a constructor cannot return a code at all.
**Do:** One error at a time. Throw a named exception class, change callers to catch where they can act — and only there — then delete the constants. Layers that only passed the code up lose their checks. Document `@throws` in PHP.
**Before / after (PHP):**
```php
// Before
public const OK = 0;
public const MISSING_NAME = 1;
public const BAD_EMAIL = 2;

public function check(array $row): int
{
    if ($row['name'] === '') {
        return self::MISSING_NAME;
    }

    if (!str_contains($row['email'], '@')) {
        return self::BAD_EMAIL;
    }

    return self::OK;
}

// ...
$code = $checker->check($row);

if ($code === RowChecker::MISSING_NAME) {
    return 'Name is missing';
}

if ($code === RowChecker::BAD_EMAIL) {
    return 'Email is not valid';
}

return 'OK';

// After
final class InvalidRow extends \RuntimeException {}

function checkRow(array $row): void
{
    if ($row['name'] === '') {
        throw new InvalidRow('Name is missing');
    }

    if (!str_contains($row['email'], '@')) {
        throw new InvalidRow('Email is not valid');
    }
}

// ...
try {
    checkRow($row);
} catch (InvalidRow $e) {
    return $e->getMessage();
}

return 'OK';
```
**When not:** The failure is routine for the caller — a lookup that often finds nothing, input users mistype every day: return `null`, or test first (Replace Exception with Test, this file). Exceptions that steer ordinary flow are a hidden `goto`.

## Replace Exception with Test
<https://refactoring.guru/replace-exception-with-test>

**Signs:** A `try`/`catch` around code that fails in a case the caller can check beforehand (division by zero, an empty list, a closed handle), and the catch block returns a default.
**Do:** Prove the case is reachable (house rule 9), put the check before the call and move the catch block's fallback into it. To find out whether anything else leans on the exception, make the catch rethrow a bare exception and run the tests. When nothing fails, delete the `try`/`catch`.
**Before / after (PHP):**
```php
// Before
function attendancePercent(int $attended, int $total): int
{
    try {
        return intdiv($attended * 100, $total);
    } catch (\DivisionByZeroError) {
        return 0;
    }
}

// After
function attendancePercent(int $attended, int $total): int
{
    if ($total === 0) {
        return 0;
    }

    return intdiv($attended * 100, $total);
}
```
**When not:** The state can change between the check and the use (a file removed, a parallel update), the check costs as much as the call, or the failure is truly unexpected and should be loud (Replace Error Code with Exception, this file).
