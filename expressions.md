# Expressions

Rules for the code inside one function: collection functions, loops, ternaries, boolean conditions, literals. Every entry gives the signs, the rewrite and when to leave the code alone. The examples are plain PHP and TypeScript, run before and after on the same inputs. House rule numbers are those of SKILL.md.

## Contents
- Dense collection-function composition
- Nested loops with conditions
- Nested and long ternaries
- Long boolean expressions
- Conditional values inside return and literals
- Silent array union
- Split Loop
- Replace Loop with Pipeline

## Dense collection-function composition

**Signs:** Two or more collection functions in one expression — `array_unique(array_filter(array_map(fn…)))`, `array_diff_assoc(array_column(…), array_unique(…))`, a TS chain ending in `.reduce`. The reader decodes it from the inside out, and often a comment explains it. `array_filter` with no callback tests truthiness and `array_unique` removes repeats, both silently, so the line never says what is kept.
**Do:** Write the loop (house rule 3). A test on an item is an `if`; "seen before" is an array keyed by the value (`isset($seen[$id])`); one loop can fill several results. `array_keys(array_filter($x))` is no exception: moving the filter into its own variable keeps the trick, so it is always a loop with an `if`.
**TS:** One call per expression is fine. Three calls or any `.reduce` become a loop, or named steps (one call per line, each named) when every step is plain. Two calls are a grep hit: rewrite them unless both callbacks are one property read or one comparison.
**Edges:** Read what the old line did before replacing it. `array_filter` also drops `'0'`, and so does `if ($value)`, while `!== ''` keeps `'0'` and `NULL`; `array_column` renumbers keys and skips rows without the column; in the second example the result keys become the values instead of positions, harmless for an `IN` condition, so check other callers; store the value as the value too (`$seen[$name] = $name`), as `array_keys($seen)` returns `'123'` as the int `123`.
**Before / after (PHP):**
```php
// Before
function tickedIds(array $column): array
{
    $ids = [];

    foreach (array_keys(array_filter($column)) as $id) {
        $ids[] = (int) $id;
    }

    return $ids;
}

// After
function tickedIds(array $column): array
{
    $ids = [];

    foreach ($column as $id => $ticked) {
        if ($ticked) {
            $ids[] = (int) $id;
        }
    }

    return $ids;
}
```
```php
// Before
function distinctValues(array $students): array
{
    $lastNames = array_unique(array_filter(array_map(fn(Student $s) => $s->lastName, $students)));
    $emails = array_unique(array_filter(array_map(fn(Student $s) => $s->email, $students)));

    return [$lastNames, $emails];
}

// After
function distinctValues(array $students): array
{
    $lastNames = [];
    $emails = [];

    foreach ($students as $student) {
        if ($student->lastName) {
            $lastNames[$student->lastName] = $student->lastName;
        }

        if ($student->email) {
            $emails[$student->email] = $student->email;
        }
    }

    return [$lastNames, $emails];
}
```
```php
// Before
function repeatedTeachers(array $teachers): array
{
    $ids = array_column($teachers, 'target_id');

    return array_diff_assoc($ids, array_unique($ids));
}

// After
function repeatedTeachers(array $teachers): array
{
    $seen = [];
    $repeats = [];

    foreach ($teachers as $delta => $teacher) {
        $id = $teacher['target_id'];

        if (isset($seen[$id])) {
            $repeats[$delta] = $id;
        }

        $seen[$id] = true;
    }

    return $repeats;
}
```
**Before / after (TS):**
```ts
// Before
function plannedHours(lessons: Lesson[]): number {
  return lessons.filter((lesson) => !lesson.cancelled).reduce((sum, lesson) => sum + lesson.hours, 0);
}

// After
function plannedHours(lessons: Lesson[]): number {
  let hours = 0;

  for (const lesson of lessons) {
    if (!lesson.cancelled) {
      hours += lesson.hours;
    }
  }

  return hours;
}
```
**When not:** One collection function with a plain callback or a named function (`array_column($rows, 'id')`, `rows.map(toId)`) reads in one pass.
**Inverse:** Replace Loop with Pipeline, below, in its one-function form.

## Nested loops with conditions

**Signs:** A loop inside a loop with an `if` around the inner one, or around both: `foreach`, `if`, `foreach`, `if`, four levels, and the statement that does the work is pushed to the right edge.
**Do:** Turn each condition into a guard that skips the item (`if (!$group->active) { continue; }`), placed first in its own loop (house rules 1 and 2). A comment that names what is skipped ("skip excluded") is the guard's name in disguise; delete it. If the inner loop is a thing with a name of its own, lift it into a method and keep one call in the outer loop (Extract Method, composing-methods.md).
**Before / after (PHP):**
```php
// Before
function enrolledNames(array $groups): array
{
    $names = [];

    foreach ($groups as $group) {
        if ($group->active) {
            foreach ($group->students as $student) {
                if ($student->excludedOn === null) {
                    $names[] = $student->name;
                }
            }
        }
    }

    return $names;
}

// After
function enrolledNames(array $groups): array
{
    $names = [];

    foreach ($groups as $group) {
        if (!$group->active) {
            continue;
        }

        foreach ($group->students as $student) {
            if ($student->excludedOn !== null) {
                continue;
            }

            $names[] = $student->name;
        }
    }

    return $names;
}
```
**When not:** One loop with one `if` is already flat. Two loops with no condition between them need no guard.

## Nested and long ternaries

**Signs:** A ternary inside a ternary (PHP 8 refuses the bare form, so it appears parenthesised); a ternary that wraps onto a second line, or whose branches hold calls and concatenation.
**Do:** A ternary picks one of two plain values (house rule 5). Three or more outcomes are a function of guard clauses, one `return` each; a long ternary becomes a guard for the special case plus named variables for the pieces. A choice by key can be a lookup array. In a literal or in JSX the value goes into a variable first (Conditional values inside return and literals, below, and react.md).
**Before / after (PHP):**
```php
// Before
function teacherLabel(?Teacher $teacher): string
{
    return $teacher === null ? 'Not assigned' : trim($teacher->lastName . ' ' . $teacher->firstName . ' ' . $teacher->fatherName);
}

// After
function teacherLabel(?Teacher $teacher): string
{
    if ($teacher === null) {
        return 'Not assigned';
    }

    $fullName = "$teacher->lastName $teacher->firstName $teacher->fatherName";

    return trim($fullName);
}
```
**Before / after (TS):**
```ts
// Before
function studentBadge(student: Student): string {
  return student.excluded ? 'EXCL' : student.onLeave ? 'AL' : student.individualSchedule ? 'ILS' : '';
}

// After
function studentBadge(student: Student): string {
  if (student.excluded) {
    return 'EXCL';
  }

  if (student.onLeave) {
    return 'AL';
  }

  if (student.individualSchedule) {
    return 'ILS';
  }

  return '';
}
```
**When not:** One choice between two plain values held in a named variable (`$title = $archived ? 'Restore' : 'Archive';`) is the ternary the rules allow.

## Long boolean expressions

**Signs:** A condition of four or more operands, or `&&` and `||` mixed under parentheses, or a negated group `!( … )`; an `if` you must read twice to know when its branch runs.
**Do:** Name each meaningful part as a variable above the condition (house rules 4 and 6). Use the words of the UI or the error message (`$isOnLeave`, not `$flag`), keep names positive and put the `!` at the use. When the parts guard the start of a function, make them guard clauses instead (Decompose Conditional or Consolidate Conditional Expression, simplifying-conditionals.md).
**Before / after (PHP):**
```php
// Before
function canTakeExam(Student $student, ?Profile $profile, string $today): bool
{
    return $student->enrolled && ($student->exclusionDate === null || $student->exclusionDate > $today) && !($profile !== null && $profile->onLeave) && $student->debts === 0;
}

// After
function canTakeExam(Student $student, ?Profile $profile, string $today): bool
{
    $notExcludedYet = $student->exclusionDate === null || $student->exclusionDate > $today;
    $isOnLeave = $profile !== null && $profile->onLeave;
    $hasDebts = $student->debts > 0;

    return $student->enrolled && $notExcludedYet && !$isOnLeave && !$hasDebts;
}
```
**When not:** Two operands that read as a sentence (`$user->isTeacher() && !$journal->isArchived()`) need no names.

## Conditional values inside return and literals

**Signs:** A ternary, `&&`, `??` chain or comparison computed inside `return [ … ]`, an object literal, the arguments of a call or the attributes of JSX; the literal mixes keys and logic, so it no longer reads as a table.
**Do:** Compute every condition into a named variable above the `return` and leave only names in the literal (house rule 4). Name it for the key's meaning. This is Extract Variable (composing-methods.md) applied to a literal.
**Before / after (PHP):**
```php
// Before
function archiveButton(Journal $journal, Viewer $viewer): array
{
    return [
        '#type' => 'submit',
        '#value' => $journal->archived ? 'Restore' : 'Archive',
        '#access' => $journal->taughtByViewer || ($viewer->manager && !$journal->locked),
        '#disabled' => $journal->sessionCount === 0 && !$viewer->manager,
    ];
}

// After
function archiveButton(Journal $journal, Viewer $viewer): array
{
    $title = $journal->archived ? 'Restore' : 'Archive';
    $mayChange = $journal->taughtByViewer || ($viewer->manager && !$journal->locked);
    $hasNothingToKeep = $journal->sessionCount === 0 && !$viewer->manager;

    return [
        '#type' => 'submit',
        '#value' => $title,
        '#access' => $mayChange,
        '#disabled' => $hasNothingToKeep,
    ];
}
```
**When not:** Plain values and property reads. A `??` default on one value (`$row['title'] ?? ''`) is a value, not a computation.

## Silent array union

**Signs:** `[ … ] + $other` or `$base + buildItem()` where both sides can hold the same key. PHP keeps the left value and drops the right one without a word; on lists the indexes collide and the right items vanish.
**Do:** Write who wins: copy the base into a variable and assign the keys that must change (`$record['groups'] = $groupIds`), or `array_merge()` with string keys when the later array should win (it renumbers integer keys). Mind the order of keys: a literal on the left puts its keys first, while assigning to an existing key keeps its place; check that nothing reads the order.
**Before / after (PHP):**
```php
// Before
function sessionRecord(array $row, array $groupIds): array
{
    return ['groups' => $groupIds] + buildItem($row);
}

// After
function sessionRecord(array $row, array $groupIds): array
{
    $record = buildItem($row);
    $record['groups'] = $groupIds;

    return $record;
}
```
**When not:** `$options + $defaults`, where the right side is a named defaults array and "the left wins" is the whole point.

## Split Loop
<https://refactoring.com/catalog/splitLoop.html>

**Signs:** One loop fills two unrelated results, a total and a list of ids, each behind its own `if`; naming the loop needs an "and".
**Do:** Copy the loop, delete from each copy what belongs to the other result, then let each loop become a method named for its result (Extract Method, composing-methods.md). Walking the list twice costs next to nothing beside the gain in reading; measure before keeping the merged loop for speed.
**Before / after (PHP):**
```php
// Before
function summary(array $sessions): array
{
    $hours = 0;
    $unapprovedIds = [];

    foreach ($sessions as $session) {
        $hours += $session->hours;

        if (!$session->approved) {
            $unapprovedIds[] = $session->id;
        }
    }

    return ['hours' => $hours, 'unapproved' => $unapprovedIds];
}

// After
function summary(array $sessions): array
{
    $hours = 0;

    foreach ($sessions as $session) {
        $hours += $session->hours;
    }

    $unapprovedIds = [];

    foreach ($sessions as $session) {
        if (!$session->approved) {
            $unapprovedIds[] = $session->id;
        }
    }

    return ['hours' => $hours, 'unapproved' => $unapprovedIds];
}
```
**When not:** Several lists filled by one rule from the same item (distinct non-empty last names and emails, above) are one job in one loop. A source that can be walked once (a generator, a result set read in a stream).

## Replace Loop with Pipeline
<https://refactoring.com/catalog/replaceLoopWithPipeline.html>

**Signs:** The technique turns a loop that fills a list into a chain of `filter` and `map` calls, and a reviewer or a model applies it by reflex.
**Do:** Take it only while the pipeline stays one collection function with a plain callback (`groups.map((group) => group.name)`, `array_column($rows, 'id')`). A condition plus a second step, an accumulator, or a `reduce` that builds an object stays a loop, so this skill mostly goes the other way, as below (house rule 3).
**Before / after (TS):** the direction this skill takes, from the pipeline back to the loop.
```ts
// Before
function hoursBySubject(lessons: Lesson[]): Record<string, number> {
  return lessons
    .filter((lesson) => lesson.approved)
    .reduce<Record<string, number>>(
      (totals, lesson) => ({ ...totals, [lesson.subject]: (totals[lesson.subject] ?? 0) + lesson.hours }),
      {},
    );
}

// After
function hoursBySubject(lessons: Lesson[]): Record<string, number> {
  const totals: Record<string, number> = {};

  for (const lesson of lessons) {
    if (!lesson.approved) {
      continue;
    }

    totals[lesson.subject] = (totals[lesson.subject] ?? 0) + lesson.hours;
  }

  return totals;
}
```
**When not:** A loop whose whole body is one `push` of a plain value, or one `if` and a `push`, is one `map` or `filter`; keep the single call.
**Inverse:** Dense collection-function composition, above.
