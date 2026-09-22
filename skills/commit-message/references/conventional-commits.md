# Conventional Commits 1.0.0

Read this when the project uses (or defaults to) the Conventional Commits
convention. Source: https://www.conventionalcommits.org/en/v1.0.0/

## Contents

- [Structure](#structure)
- [Type vocabulary](#type-vocabulary)
- [Choosing a type when several fit](#choosing-a-type-when-several-fit)
- [Scope](#scope)
- [Description](#description)
- [Body](#body)
- [Footers](#footers)
- [Breaking changes](#breaking-changes)
- [SemVer mapping](#semver-mapping)
- [Revert commits](#revert-commits)
- [Commitlint interaction](#commitlint-interaction)

## Structure

```
<type>[optional scope][optional !]: <description>

[optional body]

[optional footer(s)]
```

Rules that the specification states as MUST:

1. The commit is prefixed with a type — a noun such as `feat` or `fix` —
   followed by an optional scope, an optional `!`, and a required colon and
   space.
2. `feat` is used when the commit adds a new feature.
3. `fix` is used when the commit patches a bug.
4. A scope, if present, is a noun in parentheses naming a section of the
   codebase: `fix(parser):`.
5. The description follows immediately after the colon and space.
6. A body, if present, begins one blank line after the description.
7. Footers, if present, begin one blank line after the body, in git trailer
   format: a token, then `: ` or ` #`, then a value. Tokens use `-` instead of
   spaces (`Reviewed-by`), which is what distinguishes the footer block from a
   multi-paragraph body. `BREAKING CHANGE` is the single exception allowed to
   contain a space.
8. Breaking changes are indicated in the prefix with `!`, in a footer with
   `BREAKING CHANGE:`, or both.
9. Everything is case-insensitive to tooling except `BREAKING CHANGE`, which
   must be uppercase. `BREAKING-CHANGE` is a synonym.

## Type vocabulary

Only `feat` and `fix` are defined by the specification itself. The rest of this
list is the Angular convention, which `@commitlint/config-conventional` follows
and which most projects adopt:

| Type | Use for |
| --- | --- |
| `feat` | New user-visible capability or API surface |
| `fix` | Bug fix — behavior was wrong and is now correct |
| `docs` | Documentation only: README, docstrings, comments, ADRs |
| `style` | Formatting with no semantic effect: whitespace, quotes, import order, linter autofixes |
| `refactor` | Restructuring that changes neither behavior nor the public API |
| `perf` | A change made specifically to improve performance |
| `test` | Adding or correcting tests, fixtures, factories |
| `build` | Build system, packaging, dependencies: `pyproject.toml`, `Dockerfile`, lockfiles |
| `ci` | CI/CD configuration: GitHub Actions, GitLab CI, pre-commit hooks |
| `chore` | Maintenance that fits nowhere else: `.gitignore`, tooling config, routine bumps |
| `revert` | Reverting a previous commit |

Some projects add their own — `deps`, `security`, `wip`, `improvement`, `i18n`.
If a custom type already appears in the history or in a commitlint config, use
it. Do not invent new types.

## Choosing a type when several fit

The change often touches more than one category. Pick by the primary intent, in
this order of precedence:

1. Did it fix incorrect behavior? → `fix`
2. Did it add capability that did not exist? → `feat`
3. Was speed the point of the change? → `perf`
4. Did the public behavior stay identical? → `refactor`, `style`, `test`,
   `docs`, `build`, `ci`, or `chore` by subject matter

A refactor that incidentally fixes a bug is a `fix` if the fix is the reason for
the commit, a `refactor` if the bug fix was an accident worth mentioning in the
body. Adding tests for a new feature belongs in the `feat` commit if they ship
together; a separate `test` commit is for tests added to existing code.

If the change genuinely belongs to two types, that is usually a sign it should
be two commits.

## Scope

A scope is optional context in parentheses, naming the part of the codebase
touched: `feat(parser):`, `fix(auth):`, `perf(db):`.

- Take scope names from the history. Consistency matters more than precision:
  if the project already says `api`, do not introduce `backend`.
- Common bases: module or app name (`users`, `billing`), layer (`api`, `db`,
  `ui`), or package in a monorepo (`core`, `web`, `cli`).
- Omit the scope when the change is global or when the project does not use
  scopes. An omitted scope is better than a vague one.
- Never use a file path or a branch name as a scope.
- If a commitlint config sets `scope-enum`, only those values are legal.

## Description

The text after `: ` — the same rules as any subject line. Imperative mood,
lowercase first word by convention, no trailing period, specific. Keep the whole
subject line within 72 characters including the `type(scope): ` prefix, which
can eat 15–20 characters on its own; prefer a shorter scope over a truncated
description.

## Body

Free-form, any number of blank-line-separated paragraphs, one blank line after
the description. Wrap at 72 characters. Same content rules as the plain style:
what and why, not how.

## Footers

Each footer is a token, a separator, and a value:

```
Refs: #123
Reviewed-by: Z
Closes #45
BREAKING CHANGE: environment variables now take precedence over
config files
```

A footer value may contain spaces and newlines; parsing ends at the next valid
token. Use the issue-reference form the project already uses — GitHub's
auto-closing keywords (`Closes`, `Fixes`, `Resolves` followed by `#N`) actually
close issues on merge, so do not use them for a reference that should stay open;
use `Refs: #N` for that.

## Breaking changes

Two mechanisms, usable together:

```
feat(api)!: send an email to the customer when a product is shipped
```

```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending
other config files
```

- The `!` sits immediately before the colon, after the scope if there is one.
- When `!` is used, the `BREAKING CHANGE:` footer may be omitted and the
  description itself describes the break — but the footer is more useful,
  because it is where a changelog generator looks for the migration note.
- A breaking change can accompany any type, not only `feat`.
- Write the footer for the consumer: what breaks, and what they must do about
  it.

## SemVer mapping

This is why the convention exists — release tooling reads it:

- `fix` → PATCH
- `feat` → MINOR
- anything with `BREAKING CHANGE` or `!` → MAJOR
- other types → no release on their own

A mistyped `fix` that should have been `feat` means a missed minor version, so
the type choice has real consequences in a project with automated releases.

## Revert commits

The specification leaves reverts to tooling, but recommends:

```
revert: let us never again speak of the noodle incident

Refs: 676104e, a215868
```

`git revert` generates its own message in the form
`Revert "<original subject>"` plus the reverted SHA. Keep that generated
subject — it is what tooling and humans grep for — and add a body explaining why
the revert was necessary, which the generated message never includes.

## Commitlint interaction

If `commitlint` runs in a hook or in CI, its config is authoritative and will
reject a message that violates it. The rules that most often bite:

- `type-enum` — only the listed types are allowed
- `scope-enum` — only the listed scopes are allowed
- `subject-case` — usually forbids sentence-case or start-case subjects
- `subject-full-stop` — forbids the trailing period
- `header-max-length` — defaults to 100 in config-conventional, but projects
  routinely tighten it to 72 or 50
- `body-max-line-length` — defaults to 100; a long URL on its own line is the
  usual violation

When such a config is present, read it and conform to it rather than to the
defaults in this reference.
