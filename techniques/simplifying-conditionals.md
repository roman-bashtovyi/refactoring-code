# Simplifying conditionals

Techniques for making a branch read as one decision: guards instead of nesting, a name instead of an expression, a method per case instead of a switch repeated in many places.

## Contents
- Consolidate Conditional Expression
- Consolidate Duplicate Conditional Fragments
- Decompose Conditional
- Replace Conditional with Polymorphism
- Remove Control Flag
- Replace Nested Conditional with Guard Clauses
- Introduce Null Object
- Introduce Assertion

## Consolidate Conditional Expression
<https://refactoring.guru/consolidate-conditional-expression>

**Signs:** Several `if`s in a row that end in the same result (the same `return`, the same action), or an `if` whose only content is another `if` — one decision cut into pieces.
**Do:** First check that no condition has a side effect: one that changes state must not be merged or reordered. Conditions in a row join with `||`, nested ones with `&&`. Then name the result: a named variable when only this function asks (house rule 4), as in Extract Variable (composing-methods.md); a method when other code asks the same question, as in Extract Method (composing-methods.md).
**Before / after (PHP):**
```php
// Before
function countedAbsenceHours(array $absence): int
{
    if ($absence['has_medical_note']) {
        return 0;
    }

    if ($absence['has_dean_order']) {
        return 0;
    }

    if ($absence['is_competition']) {
        return 0;
    }

    return $absence['hours'];
}

// After
function countedAbsenceHours(array $absence): int
{
    $isExcused = $absence['has_medical_note'] || $absence['has_dean_order'] || $absence['is_competition'];

    if ($isExcused) {
        return 0;
    }

    return $absence['hours'];
}
```
**When not:** The checks differ in result or in reason (each guard has its own message or exception) — separate guards read better. The merged condition would still be long and have no name.

## Consolidate Duplicate Conditional Fragments
<https://refactoring.guru/consolidate-duplicate-conditional-fragments>

**Signs:** The same statements at the start, the end or the middle of every branch of an `if` or `match`; a later edit reaches one copy and misses the other.
**Do:** Lift the shared statements out: leading ones go before the conditional, trailing ones after it. A copy in the middle moves only if the statements around it do not depend on it — reorder the branch until the copy is first or last, then lift it. A shared block of several lines may deserve a method: Extract Method (composing-methods.md).
**Before / after (PHP):**
```php
// Before
function approveLesson(array $lesson, Log $log): void
{
    if ($lesson['date'] === null) {
        $log->add('lock journal ' . $lesson['journal']);
        $log->add('set date to today');
        $log->add('approve');
        $log->add('clear cache journal:' . $lesson['journal']);
    } else {
        $log->add('lock journal ' . $lesson['journal']);
        $log->add('keep date ' . $lesson['date']);
        $log->add('approve');
        $log->add('clear cache journal:' . $lesson['journal']);
    }
}

// After
function approveLesson(array $lesson, Log $log): void
{
    $log->add('lock journal ' . $lesson['journal']);

    if ($lesson['date'] === null) {
        $log->add('set date to today');
    } else {
        $log->add('keep date ' . $lesson['date']);
    }

    $log->add('approve');
    $log->add('clear cache journal:' . $lesson['journal']);
}
```
**When not:** The statement reads something the conditional changes, or runs only because a branch was taken — lifting it changes the order or the outcome. Branches that end in `return` or `throw` have no shared tail.

## Decompose Conditional
<https://refactoring.guru/decompose-conditional>

**Signs:** An `if` whose condition needs a comment or mixes several tests; branches so long that reading one makes you forget the condition that led there.
**Do:** Extract the condition into a method named for the question it answers, then each branch into a method named for what it does. Same for the cases of a `switch` or `match`. A one-line branch stays inline (house rule 8); a condition used once may only need a named variable (house rule 4).
**Before / after (PHP):**
```php
// Before
function lessonPay(array $lesson, array $holidays): float
{
    if ($lesson['weekday'] >= 6 || in_array($lesson['date'], $holidays, true)) {
        $pay = $lesson['hours'] * $lesson['off_day_rate'];
        $pay += $lesson['travel_fee'];
    } else {
        $pay = $lesson['hours'] * $lesson['workday_rate'];
        $pay += $lesson['materials_fee'];
    }

    return $pay;
}

// After
function lessonPay(array $lesson, array $holidays): float
{
    if (isOffDay($lesson, $holidays)) {
        return offDayPay($lesson);
    }

    return workdayPay($lesson);
}

function isOffDay(array $lesson, array $holidays): bool
{
    return $lesson['weekday'] >= 6 || in_array($lesson['date'], $holidays, true);
}

function offDayPay(array $lesson): float
{
    $pay = $lesson['hours'] * $lesson['off_day_rate'];

    return $pay + $lesson['travel_fee'];
}

function workdayPay(array $lesson): float
{
    $pay = $lesson['hours'] * $lesson['workday_rate'];

    return $pay + $lesson['materials_fee'];
}
```
**When not:** Condition and branches already read at a glance — a method per line is noise (house rule 8). A name that only repeats the code (`isWeekdayAtLeastSix`) explains nothing.

## Replace Conditional with Polymorphism
<https://refactoring.guru/replace-conditional-with-polymorphism>

**Signs:** The same `switch`, `match` or `instanceof` chain over one type, field or call result sits in several methods; a new case means finding every copy; behaviour that belongs to the type is decided outside it.
**Do:** A hierarchy must exist first. When the branches test a type code, build it with Replace Type Code with Subclasses or Replace Type Code with State/Strategy (organizing-data.md). A conditional inside a longer method is first pulled out into its own method: Extract Method (composing-methods.md). Then per subclass: copy its branch into an override and delete the branch; when the conditional is empty, delete it and make the parent method abstract.
**Before / after (PHP):**
```php
// Before
// BasicLesson and ControlLesson already exist; the callers ask which one they got
function label(Lesson $lesson): string
{
    if ($lesson instanceof ControlLesson) {
        return "Control lesson, $lesson->hours h";
    }

    return "Lesson, $lesson->hours h";
}

function plannedHours(Lesson $lesson): int
{
    if ($lesson instanceof ControlLesson) {
        return 0;
    }

    return $lesson->hours;
}

// After
abstract class Lesson
{
    // ...
    abstract public function label(): string;

    abstract public function plannedHours(): int;
}

final class BasicLesson extends Lesson
{
    public function label(): string
    {
        return "Lesson, $this->hours h";
    }

    public function plannedHours(): int
    {
        return $this->hours;
    }
}

final class ControlLesson extends Lesson
{
    public function label(): string
    {
        return "Control lesson, $this->hours h";
    }

    public function plannedHours(): int
    {
        return 0;
    }
}
```
**When not:** One conditional in one place, or two or three branches over an enum — a `match` is shorter and just as clear. A hierarchy built for a single switch costs more than it saves.

## Remove Control Flag
<https://refactoring.guru/remove-control-flag>

**Signs:** A boolean set in one place and tested in others to decide whether to go on (`$found`, `$done`); a loop that keeps running after the answer is known, with `!$found &&` guarding its body; a flag set in an inner loop to stop the outer one.
**Do:** At the spot where the flag is set, put the jump it imitated: `break` to leave the loop, `continue` to skip the iteration, `return` to leave the function with the value. Then delete the flag and every test of it. A flag that stops an outer loop from inside an inner one: move both loops into a method and `return` (house rule 2).
**Before / after (PHP):**
```php
// Before
function firstUngradedStudent(array $students): ?string
{
    $found = false;
    $name = null;

    foreach ($students as $student) {
        if (!$found && $student['grade'] === null) {
            $found = true;
            $name = $student['name'];
        }
    }

    return $name;
}

// After
function firstUngradedStudent(array $students): ?string
{
    foreach ($students as $student) {
        if ($student['grade'] === null) {
            return $student['name'];
        }
    }

    return null;
}
```
**When not:** The boolean is a fact other code reads after the loop (`$hadErrors`), not a stand-in for a jump. Cleanup that must run after the loop is a reason to `break`, not to `return`.

## Replace Nested Conditional with Guard Clauses
<https://refactoring.guru/replace-nested-conditional-with-guard-clauses>

**Signs:** `if` inside `if` inside `if` with the normal path indented deepest; a `$result` default at the top returned from the one exit at the bottom; an `else` that only wraps a value.
**Do:** Settle side effects first, or reordering changes what happens: Separate Query from Modifier (simplifying-method-calls.md). Then turn each special case into an `if` that ends in `return`, `throw` or `continue`, in order, at the top; the normal path stays unindented below them. No `else` after a jump (house rules 1, 2). Guards with the same result merge (Consolidate Conditional Expression, in this file).
**Before / after (PHP):**
```php
// Before
function gradeStatus(?array $session, bool $isLocked, ?int $grade): string
{
    $status = '';

    if ($session !== null) {
        if (!$isLocked) {
            if ($grade !== null) {
                $status = "Saved: $grade for {$session['topic']}";
            } else {
                $status = 'Grade is empty';
            }
        } else {
            $status = 'Locked';
        }
    }

    return $status;
}

// After
function gradeStatus(?array $session, bool $isLocked, ?int $grade): string
{
    if ($session === null) {
        return '';
    }

    if ($isLocked) {
        return 'Locked';
    }

    if ($grade === null) {
        return 'Grade is empty';
    }

    return "Saved: $grade for {$session['topic']}";
}
```
**When not:** Two paths of equal weight (neither is the exception) — a plain `if`/`else` says so, guards would call one of them special.

## Introduce Null Object
<https://refactoring.guru/introduce-null-object>

**Signs:** A finder returns `null` for "nothing there" and every caller starts with `if ($x === null)`; the same fallback (empty list, default label, do nothing) is written again at each caller.
**Do:** A class with the real one's interface whose methods do the harmless default; the finder returns it instead of `null`. To find the checks, add `isNull()` to both classes and route each `=== null` through it; then move the fallback into the null object's methods and delete the checks. It is Replace Conditional with Polymorphism (this file) for the case "nothing".
**When not:** One caller checks once — `??` or `?->` is shorter. No caller can really get `null` (house rule 9) — delete the check. The caller must react to absence (a 404, an error message): a stand-in hides it. The extra class costs more than the checks it removes.

## Introduce Assertion
<https://refactoring.guru/introduce-assertion>

**Signs:** A comment says what must hold ("never empty", "only called for approved lessons"), so the code leans on a fact it does not state; a wrong value surfaces three calls later.
**Do:** State the fact where the code relies on it: `assert($hours > 0)`, in TS a small `invariant()` that throws. While the fact holds the program behaves as before (house rule 12). The comment goes, since the code now says it (house rule 7). Assert only what the code needs; if it still works when the check fails, drop the check. PHP skips `assert()` when `zend.assertions` is off, which is the production default: it documents and catches in tests, nothing more.
**When not:** User input, a file, the network or another service can break the fact — that is validation or an exception the caller can handle: Replace Error Code with Exception (simplifying-method-calls.md). Asserting everything buries the conditions that matter.
