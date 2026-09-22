# Commit message style guide

The craft rules that apply regardless of whether the project uses Conventional
Commits. Built on the widely adopted seven rules (Chris Beams, "How to Write a
Git Commit Message"), the git project's own `SubmittingPatches`, and the Linux
kernel's `submitting-patches` guidance.

## Contents

- [The seven rules](#the-seven-rules)
- [Imperative mood](#imperative-mood)
- [Why the 50/72 limits are real](#why-the-5072-limits-are-real)
- [What belongs in the body](#what-belongs-in-the-body)
- [Atomic commits and how to split](#atomic-commits-and-how-to-split)
- [Writing in English from a non-English request](#writing-in-english-from-a-non-english-request)
- [Special cases: amend, fixup, revert, merge, WIP](#special-cases-amend-fixup-revert-merge-wip)
- [Anti-patterns](#anti-patterns)
- [Reviewing an existing message](#reviewing-an-existing-message)

## The seven rules

1. Separate subject from body with a blank line.
2. Limit the subject line to 50 characters.
3. Capitalize the subject line.
4. Do not end the subject line with a period.
5. Use the imperative mood in the subject line.
6. Wrap the body at 72 characters.
7. Use the body to explain what and why, not how.

Rule 3 is the one that Conventional Commits overrides: `fix(auth): reject
expired tokens` keeps the description lowercase. Rules 1, 2, 4, 5, 6, and 7
apply to both styles unchanged.

## Imperative mood

Git itself writes in the imperative — `Merge branch 'x'`, `Revert "..."` — and a
consistent history reads as a list of instructions applied to the codebase.

The test: the subject must complete the sentence

> If applied, this commit will **\_\_\_**

- *If applied, this commit will* `add retry logic to the webhook sender` ✓
- *If applied, this commit will* `added retry logic to the webhook sender` ✗
- *If applied, this commit will* `adds retry logic to the webhook sender` ✗
- *If applied, this commit will* `retry logic for webhooks` ✗ (a noun phrase,
  not an instruction)

The imperative applies to the subject only. The body is normal prose in whatever
tense reads naturally, usually past tense for describing the old behavior and
present for the new.

Useful opening verbs: add, remove, drop, fix, prevent, rename, extract, inline,
move, replace, introduce, revert, bump, restore, enforce, allow, disallow,
skip, cache, retry, validate, normalize, deduplicate, document, simplify.

Avoid weak openers that say nothing: `update`, `change`, `improve`, `refactor`,
`handle`, `support`, `work on`. They are not always wrong — `update dependencies
to fix CVE-2026-1234` is fine — but if the verb is `update` and the object is
generic, the subject is probably empty of information.

## Why the 50/72 limits are real

- `git log --oneline`, GitHub, GitLab, and most UIs truncate subjects around
  50–72 characters; the part past the cut is invisible where people actually
  read history.
- `git log` indents the body by four spaces. A body wrapped at 72 renders at 76
  columns, which fits an 80-column terminal. Unwrapped bodies become a single
  very long line that wraps unpredictably.
- 50 is a target that forces a summary; 72 is the ceiling. If the subject will
  not fit in 72 characters, the commit is usually doing more than one thing —
  see splitting below.

Editors do not enforce this. Count the characters.

## What belongs in the body

Write a body when a reader six months from now — often the author — would ask
"why?". Good material, roughly in order of value:

- **The problem.** What was broken or missing, with the symptom as observed:
  a 500 on a specific endpoint, a race under concurrency, a slow query.
- **The root cause**, when it is not obvious from the diff.
- **Why this approach.** The alternatives considered and the reason they were
  rejected. This is the single most valuable thing a commit body can contain,
  because it is nowhere else in the repository.
- **Consequences.** Performance characteristics, behavior changes for callers,
  data migrations required, feature flags involved.
- **Limitations.** What the change deliberately does not fix, and what the
  follow-up is.
- **Links.** Issue, RFC, ADR, upstream bug, or the discussion that drove it.

Deliberately leave out:

- A restatement of the diff in prose ("changed line 42 to use a dict") — the
  diff is right there.
- A list of the files touched, unless the grouping itself is the point.
- Process noise: "as requested in review", "after rebasing on main", "second
  attempt".
- Anything about the tooling or process used to author the change.

Trivial commits need no body at all. A typo fix, a lockfile refresh, or a
version bump is fully described by its subject, and an invented paragraph
beneath it only adds noise.

## Atomic commits and how to split

One commit is one logical change that leaves the tree buildable and the tests
passing. That makes `git bisect` meaningful, `git revert` safe, and
`git log -- <path>` readable.

Signals that a diff should be more than one commit:

- The subject needs "and", a comma, or a slash to be accurate.
- The diff mixes a behavior change with a formatting sweep or an import
  reorder — the real change is then invisible inside the noise.
- The diff mixes a dependency bump with the code that uses the new version.
- Unrelated files change for unrelated reasons.
- Part of the diff could be reverted on its own without touching the rest.

When splitting is warranted, do not commit anything. Present the plan: group the
changes, state which paths or hunks belong to each group, and give a complete
message per group, ordered so every intermediate state works (refactor before
the feature that relies on it, dependency bump before its use). Mention that
`git add -p` lets the user stage hunks selectively when the split runs through a
single file.

If the user declines the split, write one honest message that names both parts
rather than a vague one that hides them.

## Writing in English from a non-English request

When the change is described in another language, translate the intent, not the
sentence.

- Identify the technical term the English-speaking ecosystem uses. "Личный
  кабинет" is `account area` or `dashboard` depending on the product, not
  `personal cabinet`. "Заявка" is `request`, `application`, or `ticket`. "Отчёт"
  is `report`. "Справочник" is usually `reference data` or `lookup table`.
- Keep identifiers exactly as they appear in the code, in backticks if the
  surrounding sentence would otherwise be ambiguous — never translate a class,
  field, or table name.
- Prefer the term the codebase already uses over a dictionary-correct synonym.
  Grep the code if unsure.
- Watch for calques: "realize" for "реализовать" (use `implement`), "correct"
  for "исправить" (use `fix`), "actualize"/"actual" for "актуализировать"/
  "актуальный" (use `update` / `current`), "delete the error" (use `fix the
  bug`), "add the possibility to" (use `allow` or `let`).
- Articles and plural agreement matter for readability; re-read the finished
  subject as English prose before sending it.

## Special cases: amend, fixup, revert, merge, WIP

**Amend.** Rewrite the whole message for the amended content — the message must
describe the commit's final state, with no trace of what it looked like before.
Never write "fix previous commit" as an amend subject.

**Fixup and squash.** `fixup!` / `squash!` followed by the exact subject of the
target commit lets `git rebase --autosquash` do the work. These are temporary by
construction; the message that survives is the target's.

**Revert.** Keep the `Revert "<original subject>"` subject that `git revert`
generates and the reverted SHA that it records. Add the body it cannot know:
why the revert is happening, what broke, and whether a fixed version is coming.

**Merge.** Leave the generated `Merge branch 'x' into y` subject alone. If the
merge is significant — a long-lived branch, a conflicting integration — add a
body describing what the branch brought and how conflicts were resolved.

**Work in progress.** A commit that does not work is not worth a good message,
but it still gets an honest one: `wip: <what is half-done>`, with a body naming
what is missing. Better still, say plainly that the change is not in a
committable state.

## Anti-patterns

| Bad | Why | Better |
| --- | --- | --- |
| `fix bug` | Which bug? | `fix crash when the cart is empty at checkout` |
| `update` / `changes` / `misc` | Says nothing | Name what changed |
| `Fixed the parser so that it now handles nested arrays correctly in all cases` | Past tense, 78 chars | `fix nested array parsing` |
| `WIP` | Survives into main forever | Finish, or say what is unfinished |
| `asdf` / `.` / `commit` | Noise in the history | Any real sentence |
| `feat: add endpoint and fix migration and bump deps` | Three commits | Split them |
| `refactor(api): update code` | Type and scope carry all the meaning | `refactor(api): extract pagination into a mixin` |
| `fix: as discussed` | Meaningless without the chat log | Summarize the discussion in the body |
| Subject repeated verbatim as the body | Padding | Drop the body |
| `feat: implement user story PROJ-123` | Ticket text is not in the repo | Describe the change, reference the ticket in a footer |
| Emoji prefixes in a project that does not use them | Inconsistent history | Follow the history |

## Reviewing an existing message

When asked to review or rewrite a message rather than write one, check in this
order and report only what actually fails:

1. Language — English throughout.
2. Subject — imperative, length, no period, specific enough to be useful alone.
3. Structure — blank line before the body, body wrapped at 72.
4. Content — does the body explain why, or just restate the diff?
5. Convention — matches the surrounding history and any commitlint config.
6. Footers — correct trailer format, correct issue-reference keyword.
7. Contamination — no attribution trailers, no session links, no tooling
   references.

Then give the corrected message in full, rather than a list of instructions for
the user to apply.
