# refactoring-code

A [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) skill for code a human reads in one pass.

When you ask Claude to review, clean up, simplify or refactor code — or say it is hard to read — the skill loads before the answer and gives Claude:

- **13 house rules** that differ from default habits: guard clauses instead of nested `if`/`else`, a loop instead of dense `array_*` / `.filter().map().reduce()` chains, no nested ternaries, conditional values named before `return`, names from the domain, comments that say *why*, behaviour kept identical.
- **All 23 code smells and 66 refactoring techniques** of the [refactoring.guru](https://refactoring.guru/refactoring) catalog, each in a few lines of its own words: signs, what to do, when not to, and a link to the page.
- **29 techniques with before/after examples** in PHP 8.3 and TypeScript; every pair was run on the same inputs to confirm the output does not change.
- **Expressions and React** sections the catalog does not cover: collection-function chains, nested loops, ternaries, long boolean conditions, silent array union, Split Loop, derived state in `useEffect`, conditions in JSX, Extract Component.
- A review checklist, a symptom → entry router and ready `grep` commands for a diff.

It also tells Claude when to leave code alone: "already readable, no change" is a valid review.

## Install

Personal, for every project:

```bash
git clone https://github.com/roman-bashtovyi/refactoring-code.git ~/.claude/skills/refactoring-code
```

For one project, shared with the team through its repository: clone into `.claude/skills/refactoring-code` inside the project.

Update with `git pull` in that folder.

## Use

Nothing to call: ask as usual — "review this", "this is hard to read", "clean up this function" — and Claude loads the skill first. To call it by hand, type `/refactoring-code`. You can also ask Claude to self-review its own diff with it before reporting.

## Files

| File | Holds |
|---|---|
| `SKILL.md` | goal, house rules, review checklist, router, quick search — the only file always loaded |
| `smells/*.md` | the 23 smells in 5 groups |
| `techniques/*.md` | the 66 techniques in 6 groups |
| `expressions.md` | rules inside one function: chains, loops, ternaries, boolean conditions, literals |
| `react.md` | React components: derived state, JSX conditions, handlers, extraction, props drilling |

Claude reads `SKILL.md` (about 120 lines) and opens another file only when an entry needs it.

## How it was checked

Five review tasks (four PHP, one React), run with no other instructions, memory or plugins, graded against fixed criteria:

| | Sonnet | Opus |
|---|---|---|
| With the skill | 25/26 | 25/26 |
| Without it | 19/26 | 17/26 |

Without the skill both models kept dense `array_filter` chains and nested ternaries and did not notice a boolean parameter that picks behaviour. One run per configuration, so take the numbers as a sign, not a benchmark.

## Contributing

Pull requests are welcome. Please keep to how the skill is written:

- Entries are in your own words, never copied from refactoring.guru or another source, and link to their page.
- A before/after example is plain PHP or TypeScript without a framework, and the after half follows the house rules itself.
- `SKILL.md` stays under 150 lines and every other file under 400; a new file is listed in `SKILL.md`.

## Credits

The smells and techniques come from Martin Fowler's book [*Refactoring: Improving the Design of Existing Code*](https://martinfowler.com/books/refactoring.html) (with Kent Beck), as catalogued on [refactoring.guru](https://refactoring.guru/refactoring). Entries summarise those pages in our own words and link to each one, as the site's [content usage policy](https://refactoring.guru/content-usage-policy) asks. The React section follows [react.dev](https://react.dev/learn). This project is not affiliated with Refactoring.Guru, Martin Fowler or the React team.

## License

[MIT](LICENSE) — for the text and examples of this repository.
