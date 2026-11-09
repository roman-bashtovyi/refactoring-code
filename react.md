# React components

Rules for components written in React with TypeScript. The language-neutral rules (early return, named variables, one collection function per expression) are in expressions.md; this file keeps what React adds. Snippets leave out the `react` imports. Each pair was rendered in a browser DOM (jsdom, effects run) and its HTML and handler calls compared. House rule numbers are those of SKILL.md.

## Contents
- Derived state in useEffect
- Nested ternaries in JSX
- Long JSX conditions
- Inline handlers with logic
- Extract Component
- Props drilling

## Derived state in useEffect
<https://react.dev/learn/you-might-not-need-an-effect>

**Signs:** A `useState` whose value is always a function of props or of other state, and a `useEffect` that does nothing but call its setter. The first render shows the empty value, a second one the real value, and two copies of one fact can drift apart.
**Do:** Delete the state and the effect and compute the value in the component body (house rule 4). A calculation that is measurably slow may be cached with `useMemo`; measure first, and skip it where the build already memoizes (React Compiler). State that must start over when an id changes: give the component that id as its `key`, not an effect that resets fields one by one. A "selected item": keep the id in state and find the item in the current list, so nothing needs adjusting. What a click causes (save, navigate, tell the parent) belongs in the handler, not in an effect watching state.
**Before / after (TSX):**
```tsx
// Before
function GroupStudents({ students, groupId }: Props) {
  const [visible, setVisible] = useState<Student[]>([]);

  useEffect(() => {
    setVisible(students.filter((student) => student.groupId === groupId));
  }, [students, groupId]);

  return (
    <ul>
      {visible.map((student) => (
        <li key={student.id}>{student.name}</li>
      ))}
    </ul>
  );
}

// After
function GroupStudents({ students, groupId }: Props) {
  const visible = students.filter((student) => student.groupId === groupId);

  return (
    <ul>
      {visible.map((student) => (
        <li key={student.id}>{student.name}</li>
      ))}
    </ul>
  );
}
```
**When not:** An effect that talks to something outside React — a request, a subscription, a timer, the DOM, a widget from another library — is what effects are for. A request needs cleanup, so a late answer cannot overwrite a newer one.

## Nested ternaries in JSX
<https://react.dev/learn/conditional-rendering>

**Signs:** `a ? <A /> : b ? <B /> : <C />` in the returned markup, or the class and the text of one element each chosen by a chain of ternaries.
**Do:** When the whole output depends on a state (loading, error, empty, data), give each state its own `if` with a `return` (house rule 1). When only parts of one element change, work out one status in a small function of guard clauses, then name the derived values above the `return` (house rules 4 and 5); a lookup object replaces a chain of texts. Return `null` only when the parent cannot decide to leave the component out.
**Before / after (TSX):**
```tsx
// Before
function StudentList({ students, loading, error }: ListProps) {
  return loading ? (
    <p>Loading…</p>
  ) : error ? (
    <p role="alert">{error}</p>
  ) : students.length === 0 ? (
    <p>No students found.</p>
  ) : (
    <ul>
      {students.map((student) => (
        <li key={student.id}>{student.name}</li>
      ))}
    </ul>
  );
}

// After
function StudentList({ students, loading, error }: ListProps) {
  if (loading) {
    return <p>Loading…</p>;
  }

  if (error) {
    return <p role="alert">{error}</p>;
  }

  if (students.length === 0) {
    return <p>No students found.</p>;
  }

  return (
    <ul>
      {students.map((student) => (
        <li key={student.id}>{student.name}</li>
      ))}
    </ul>
  );
}
```
```tsx
// Before
function LessonBadge({ lesson }: BadgeProps) {
  return (
    <span className={lesson.cancelled ? 'cancelled' : lesson.replaced ? 'replaced' : 'normal'}>
      {lesson.cancelled ? 'Cancelled' : lesson.replaced ? 'Replaced' : lesson.title}
    </span>
  );
}

// After
const STATUS_TEXT = { cancelled: 'Cancelled', replaced: 'Replaced' } as const;

function statusOf(lesson: Lesson): 'cancelled' | 'replaced' | 'normal' {
  if (lesson.cancelled) {
    return 'cancelled';
  }

  if (lesson.replaced) {
    return 'replaced';
  }

  return 'normal';
}

function LessonBadge({ lesson }: BadgeProps) {
  const status = statusOf(lesson);
  const text = status === 'normal' ? lesson.title : STATUS_TEXT[status];

  return <span className={status}>{text}</span>;
}
```
**When not:** One choice between two plain values held in a named variable (`const label = open ? 'Hide' : 'Show';`). Strings a tool collects for translation by pattern (`Drupal.t('…')`, `t('…')`) stay literals inside the call, so a lookup object cannot hold them: use guard returns with the call in each.

## Long JSX conditions
<https://react.dev/learn/conditional-rendering>

**Signs:** Three or more operands joined by `&&` and `||` inside `{ … && <X /> }`; one condition written twice in the markup; a number on the left of `&&`.
**Do:** Name each condition as a boolean above the `return`, in the words of the screen (`canEdit`, `hasMaterials`), and leave only the name left of `&&` (house rule 4). The left side must be a real boolean: a number there (`items.length && …`) is drawn as a `0`, so compare it (`items.length > 0`). Such a change alters the output, so report it as a fix rather than slipping it into a readability edit (house rules 12 and 13).
**Before / after (TSX):**
```tsx
// Before
function SessionRow({ session, viewer }: RowProps) {
  return (
    <li>
      <span>{session.topic}</span>
      {viewer.isTeacher && !session.archived && session.date >= viewer.blockingDay && <button>Edit</button>}
      {session.materials !== null && session.materials !== '' && <p>{session.materials}</p>}
    </li>
  );
}

// After
function SessionRow({ session, viewer }: RowProps) {
  const canEdit = viewer.isTeacher && !session.archived && session.date >= viewer.blockingDay;
  const hasMaterials = session.materials !== null && session.materials !== '';

  return (
    <li>
      <span>{session.topic}</span>
      {canEdit && <button>Edit</button>}
      {hasMaterials && <p>{session.materials}</p>}
    </li>
  );
}
```
**When not:** One short flag (`{isOpen && <Menu />}`) is already a name.

## Inline handlers with logic

**Signs:** `onClick={() => { … }}` holding `if`s, several statements or state updates; the same body pasted into two elements.
**Do:** Declare a function in the component above the `return`, named for what the user does in the words of the screen (`handleApproveClick`), with guard clauses inside, and pass it as `onClick={handleApproveClick}`. Logic that touches neither props nor state moves out of the component altogether.
**Before / after (TSX):**
```tsx
// Before
function ApproveButton({ session, onApprove }: ApproveProps) {
  const [confirming, setConfirming] = useState(false);
  const label = confirming ? 'Confirm' : 'Approve';

  return (
    <button
      onClick={() => {
        if (session.approved) {
          return;
        }

        if (!confirming) {
          setConfirming(true);

          return;
        }

        onApprove(session.id);
        setConfirming(false);
      }}
    >
      {label}
    </button>
  );
}

// After
function ApproveButton({ session, onApprove }: ApproveProps) {
  const [confirming, setConfirming] = useState(false);
  const label = confirming ? 'Confirm' : 'Approve';

  function handleApproveClick() {
    if (session.approved) {
      return;
    }

    if (!confirming) {
      setConfirming(true);

      return;
    }

    onApprove(session.id);
    setConfirming(false);
  }

  return <button onClick={handleApproveClick}>{label}</button>;
}
```
**When not:** A one-call arrow (`onClick={() => setOpen(true)}`), or an arrow that only binds a list item (`() => onSelect(item.id)`).

## Extract Component

**Signs:** A component that works out numbers, lays out a section and draws rows with conditions of their own — more than one thing to name. A `.map` whose callback holds a screenful of markup; a comment that splits the markup into parts.
**Do:** The same move as Extract Method (composing-methods.md): lift the piece into a component named for what it shows (`SessionRow`, `ApprovedHours`), give it only what it reads as props, and take the values it alone uses along with it. The `key` stays on the element the list creates.
**Before / after (TSX):**
```tsx
// Before
function SessionList({ sessions }: ListProps) {
  let hours = 0;

  for (const session of sessions) {
    if (session.approved) {
      hours += session.hours;
    }
  }

  return (
    <section>
      <h2>Lessons ({sessions.length})</h2>
      <ul>
        {sessions.map((session) => (
          <li key={session.id}>
            <b>{session.topic}</b>
            {session.approved ? <em> approved</em> : <em> draft</em>}
            {session.homework !== '' && <p>Homework: {session.homework}</p>}
          </li>
        ))}
      </ul>
      <p>Approved hours: {hours}</p>
    </section>
  );
}

// After
function SessionRow({ session }: RowProps) {
  const state = session.approved ? 'approved' : 'draft';
  const hasHomework = session.homework !== '';

  return (
    <li>
      <b>{session.topic}</b>
      <em> {state}</em>
      {hasHomework && <p>Homework: {session.homework}</p>}
    </li>
  );
}

function ApprovedHours({ sessions }: ListProps) {
  let hours = 0;

  for (const session of sessions) {
    if (session.approved) {
      hours += session.hours;
    }
  }

  return <p>Approved hours: {hours}</p>;
}

function SessionList({ sessions }: ListProps) {
  return (
    <section>
      <h2>Lessons ({sessions.length})</h2>
      <ul>
        {sessions.map((session) => (
          <SessionRow key={session.id} session={session} />
        ))}
      </ul>
      <ApprovedHours sessions={sessions} />
    </section>
  );
}
```
**When not:** A piece used once with three lines of markup and no logic. A cut that needs ten props — the seam is in the wrong place. Pieces so small that reading means jumping between files for every line.

## Props drilling
<https://react.dev/learn/passing-data-deeply-with-context>

**Signs:** A prop handed through several components that never read it, only to reach a deeper one.
**Do:** Restructure first: build the consumer where the data lives and pass it down as `children` or a prop, so the components in between forward markup, not data. Take context when many distant components in different branches need the same value (the current user, a theme).
**When not:** Two levels of explicit props. They show who uses what; context hides where a value comes from and redraws every reader when it changes.
