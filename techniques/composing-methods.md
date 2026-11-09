# Composing methods

Techniques for cutting a method into pieces and naming what is left. Direction in this skill: extract when the name tells the reader what the code does not (house rule 8), inline when it does not.

## Contents
- Extract Method
- Inline Method
- Extract Variable
- Inline Temp
- Replace Temp with Query
- Split Temporary Variable
- Remove Assignments to Parameters
- Replace Method with Method Object
- Substitute Algorithm

## Extract Method
<https://refactoring.guru/extract-method>

**Signs:** A method past about ten lines; a block set off by a blank line or a comment saying what it does (house rule 7: that comment is a name not yet written); a loop body that is a second algorithm; the inner half of a loop in a loop (house rule 2).
**Do:** Move the block into a method named for its purpose. Locals only the block uses go with it, locals it reads become parameters, the one local it changes and the caller still needs becomes the return value. Two or more changed locals: the cut is wrong, or use Replace Method with Method Object.
**Before / after (PHP):**
```php
// Before
function attendanceLine(string $student, array $lessons): string
{
    // count the lessons the student missed
    $missed = 0;

    foreach ($lessons as $lesson) {
        if ($lesson['absent']) {
            $missed++;
        }
    }

    // share of lessons attended, in percent
    $attended = count($lessons) - $missed;
    $percent = round($attended / count($lessons) * 100);

    return "$student: $attended of " . count($lessons) . " ($percent%)";
}

// After
function attendanceLine(string $student, array $lessons): string
{
    $total = count($lessons);
    $attended = $total - countMissed($lessons);
    $percent = round($attended / $total * 100);

    return "$student: $attended of $total ($percent%)";
}

function countMissed(array $lessons): int
{
    $missed = 0;

    foreach ($lessons as $lesson) {
        if ($lesson['absent']) {
            $missed++;
        }
    }

    return $missed;
}
```
**When not:** The new name would only repeat one line, or wrap a call that already exists (house rule 8) — keep the line. Extracting only to hide a long parameter list moves the smell, it does not remove it.
**Inverse:** Inline Method, when the body reads as clearly as the name would.

## Inline Method
<https://refactoring.guru/inline-method>

**Signs:** A method whose body is as clear as its name; a method that only forwards to another; a private wrapper over a one-line call that already exists (house rule 8); a set of badly cut small methods that must be read together.
**Do:** Check no subclass overrides it. Put the body at every call site, delete the method. If the method came from a bad cut, inline the whole group first, then cut again.
**Before / after (PHP):**
```php
// Before
final class GradeEditor
{
    // ...
    public function canEdit(int $userId): bool
    {
        return $this->isTeacherOfJournal($userId);
    }

    private function isTeacherOfJournal(int $userId): bool
    {
        return $this->journal->hasTeacher($userId);
    }
}

// After
final class GradeEditor
{
    // ...
    public function canEdit(int $userId): bool
    {
        return $this->journal->hasTeacher($userId);
    }
}
```
**When not:** The method is overridden, is part of a public API, or is called from many places and its name carries meaning the body does not.
**Inverse:** Extract Method, when the body needs a name to be understood.

## Extract Variable
<https://refactoring.guru/extract-variable>

**Signs:** A condition with `&&`/`||` plus a negation or two comparisons; a ternary whose test is compound; arithmetic whose parts are unexplained; a comment above one line saying what a piece of it means.
**Do:** Name each piece for what it means in the domain, one named variable per piece, on the lines just before the statement that uses it (house rule 4: conditional values leave the `return`, the array literal and the JSX). Take the name from the UI or error wording (house rule 6). One piece at a time.
**Before / after (PHP):**
```php
// Before
function enrolmentRow(Student $student, ?Profile $profile, string $today): array
{
    return [
        'name' => $student->name,
        'badge' => $profile !== null && $profile->onLeave ? 'AL' : '',
        'can_enrol' => $student->exclusionDate === null || $student->exclusionDate > $today,
    ];
}

// After
function enrolmentRow(Student $student, ?Profile $profile, string $today): array
{
    $onLeave = $profile !== null && $profile->onLeave;
    $badge = $onLeave ? 'AL' : '';
    $canEnrol = $student->exclusionDate === null || $student->exclusionDate > $today;

    return [
        'name' => $student->name,
        'badge' => $badge,
        'can_enrol' => $canEnrol,
    ];
}
```
**When not:** The name would only repeat the code (`$isTrue = $x === true`); the expression is short enough to read at once; the parts guard each other (`$user !== null && $user->active`) or one is expensive — a variable per part runs them all, so keep the short-circuit.
**Inverse:** Inline Temp, for a variable that renames nothing.

## Inline Temp
<https://refactoring.guru/inline-temp>

**Signs:** A variable assigned once from a simple expression and read once, often straight into `return`; a name that restates the expression (`$teachers = $journal->teacherIds`).
**Do:** Put the expression where the variable is read; delete the declaration and the assignment.
**Before / after (PHP):**
```php
// Before
function canApprove(Journal $journal, int $userId): bool
{
    $teachers = $journal->teacherIds;
    $isTeacher = in_array($userId, $teachers, true);

    return $isTeacher;
}

// After
function canApprove(Journal $journal, int $userId): bool
{
    return in_array($userId, $journal->teacherIds, true);
}
```
**When not:** The variable names a compound condition or a calculation (that is Extract Variable, house rule 4 — never undo it); it caches an expensive call read more than once; it freezes a value before a later line changes it.
**Inverse:** Extract Variable, when the expression needs a name.

## Replace Temp with Query
<https://refactoring.guru/replace-temp-with-query>

**Signs:** A local holds a computed value that several blocks read, which makes those blocks hard to extract; the same computation written out in two methods.
**Do:** Check the variable is assigned once (otherwise Split Temporary Variable first) and the computation changes nothing. Move it into a method named for what the value is; read it through the call.
**Before / after (PHP):**
```php
// Before
final class StudentCard
{
    // ...
    public function title(): string
    {
        $fullName = trim("{$this->student->lastName} {$this->student->firstName} {$this->student->middleName}");

        return "Card of $fullName";
    }

    public function mailSubject(): string
    {
        $fullName = trim("{$this->student->lastName} {$this->student->firstName} {$this->student->middleName}");

        return "Grades of $fullName";
    }
}

// After
final class StudentCard
{
    // ...
    public function title(): string
    {
        return 'Card of ' . $this->fullName();
    }

    public function mailSubject(): string
    {
        return 'Grades of ' . $this->fullName();
    }

    private function fullName(): string
    {
        return trim("{$this->student->lastName} {$this->student->firstName} {$this->student->middleName}");
    }
}
```
**When not:** The computation is expensive and read many times — extract the method but keep the variable; the value must be a snapshot (clock, counter, random); the new method would wrap one existing call (house rule 8).
**Inverse:** Inline Temp.

## Split Temporary Variable
<https://refactoring.guru/split-temporary-variable>

**Signs:** One variable assigned for different meanings, under a name that fits none (`$value`, `$tmp`, `$result`, `$data`). Not a sign: loop counters and accumulators (`$sum += …`).
**Do:** From the first assignment on, give each meaning its own variable named for what it holds; reads after a reassignment go to the new one. In TS each one becomes a `const`, so a stray reassignment no longer compiles.
**Before / after (PHP):**
```php
// Before
function sessionSummary(Session $session): string
{
    $value = $session->endMinute - $session->startMinute;
    $line = "Duration: $value min";

    $value = count($session->students);
    $line .= ", students: $value";

    return $line;
}

// After
function sessionSummary(Session $session): string
{
    $duration = $session->endMinute - $session->startMinute;
    $studentCount = count($session->students);

    return "Duration: $duration min, students: $studentCount";
}
```
**When not:** A real accumulator or counter, or a value rebuilt in a loop on purpose.
**Inverse:** Inline Temp, for a piece that ends up used once and named after its own expression.

## Remove Assignments to Parameters
<https://refactoring.guru/remove-assignments-to-parameters>

**Signs:** A parameter overwritten or incremented in the body, so its name means the argument at the top and something else lower down.
**Do:** Copy it into a local named for the new meaning before anything touches it, or compute straight into one; point every later read at the local. A by-value caller sees no difference, so the gain is for the reader.
**Before / after (PHP):**
```php
// Before
function rescale(int $grade, int $fromMax, int $toMax): int
{
    if ($fromMax !== $toMax) {
        $grade = (int) round($grade * $toMax / $fromMax);
    }

    $grade = min($grade, $toMax);

    return $grade;
}

// After
function rescale(int $grade, int $fromMax, int $toMax): int
{
    $rescaled = $grade;

    if ($fromMax !== $toMax) {
        $rescaled = (int) round($grade * $toMax / $fromMax);
    }

    return min($rescaled, $toMax);
}
```
**When not:** A one-line default at the top of a short method reads fine; a by-reference parameter that is the declared output — returning the value is better, but that is a separate change (house rule 13: list it).

## Replace Method with Method Object
<https://refactoring.guru/replace-method-with-method-object>

**Signs:** A long method whose locals are written in one block and read in later ones, so each Extract Method needs many parameters or must return several values.
**Do:** New class named for the computation. Its constructor stores the method's parameters (and the owner, if the body needs it); locals become private fields; the body moves to one public method, which you then cut freely into private methods. The old method shrinks to one line that builds the object and calls it.
**Before / after (PHP):**
```php
// Before
function gradeSummary(array $marks, int $passMark): string
{
    $sum = 0;
    $failed = 0;

    foreach ($marks as $mark) {
        $sum += $mark;

        if ($mark < $passMark) {
            $failed++;
        }
    }

    $line = sprintf('average %.1f', $sum / count($marks));

    if ($failed > 0) {
        $line .= sprintf(', failed %d of %d', $failed, count($marks));
    }

    return $line;
}

// After
final class GradeSummary
{
    private int $sum = 0;
    private int $failed = 0;

    public function __construct(
        private readonly array $marks,
        private readonly int $passMark,
    ) {}

    public function render(): string
    {
        $this->tally();
        $line = sprintf('average %.1f', $this->sum / count($this->marks));

        if ($this->failed > 0) {
            $line .= sprintf(', failed %d of %d', $this->failed, count($this->marks));
        }

        return $line;
    }

    private function tally(): void
    {
        foreach ($this->marks as $mark) {
            $this->sum += $mark;

            if ($mark < $this->passMark) {
                $this->failed++;
            }
        }
    }
}

function gradeSummary(array $marks, int $passMark): string
{
    return (new GradeSummary($marks, $passMark))->render();
}
```
**When not:** Extract Method already works — a class is the heavier tool, and this example is shrunk to fit: a real candidate is far longer. Helpers that fill fields make call order matter (`tally()` must run before the totals are read); have them return values where you can.

## Substitute Algorithm
<https://refactoring.guru/substitute-algorithm>

**Signs:** A long way to a result a simpler method gives; a hand-written copy of what the language or an installed library already does; logic that no longer matches the requirement.
**Do:** Simplify the surroundings first so only the algorithm is left. Write the new version beside the old; run both on the same inputs, awkward ones included (empty, duplicates, mixed types) — house rule 12. Switch, delete the old one last. On a mismatch suspect the new code first.
**Before / after (PHP):**
```php
// Before
function hasDuplicates(array $ids): bool
{
    $count = count($ids);

    for ($left = 0; $left < $count; $left++) {
        for ($right = $left + 1; $right < $count; $right++) {
            if ($ids[$left] === $ids[$right]) {
                return true;
            }
        }
    }

    return false;
}

// After
function hasDuplicates(array $ids): bool
{
    $seen = [];

    foreach ($ids as $id) {
        if (isset($seen[$id])) {
            return true;
        }

        $seen[$id] = true;
    }

    return false;
}
```
**When not:** Outputs cannot be compared; the new version is a dense composition of collection functions (house rules 3 and 11) — that trades one hard read for another. This swap treats `'1'` and `1` as the same key where `===` did not: fine for a list of ints, not for mixed input.
