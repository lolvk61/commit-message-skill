# Commit Message — a Claude Code skill

A skill that writes **one good git commit message** and hands it to you in the
chat as copyable text. It reads the actual diff, detects the convention your
repository already uses, and produces a message in English — with no AI
attribution, no files written to disk, and no commit created behind your back.

This repository is both a **plugin** and a **marketplace**, so it installs with
two commands and nothing else.

---

## Table of contents

- [Why this exists](#why-this-exists)
- [What the skill guarantees](#what-the-skill-guarantees)
- [Installation](#installation)
- [Usage](#usage)
- [Example output](#example-output)
- [How it works](#how-it-works)
- [Reference files](#reference-files)
- [Customizing the skill](#customizing-the-skill)
- [Updating and uninstalling](#updating-and-uninstalling)
- [Compatibility](#compatibility)
- [License](#license)

---

## Why this exists

Asking an assistant for a commit message usually goes wrong in the same handful
of ways:

- it appends `Co-Authored-By:` and a "Generated with…" banner that then lives in
  your history forever;
- it runs `git commit` when you only wanted the wording;
- it writes the message into a scratch file you have to clean up;
- it answers in the language of the conversation instead of the language of the
  repository;
- it applies Conventional Commits to a project that has never used them — or
  ignores the `commitlint` config that will reject the message in CI;
- it produces `update project files` for a diff that is really three commits.

This skill exists to make all six impossible by construction. The rules that
prevent them are written as explicit overrides, so they win against default
behavior.

## What the skill guarantees

Four constraints are non-negotiable and are checked before every message is
sent:

| # | Rule | What it means in practice |
| --- | --- | --- |
| 1 | **English only** | Subject, body, and footers are always English, whatever language you asked in. Intent is translated, not transliterated — identifiers stay verbatim, and calques like "realize" for *реализовать* are avoided. |
| 2 | **Chat output only** | No `git commit`, no `--amend`, no `git tag -m`, no `COMMIT_MSG.txt`, no staging, stashing, or index changes. Only read-only git commands are used. The single exception: if you explicitly ask for the commit in the same turn, it is created by passing the text inline with repeated `-m` flags — still never through a file on disk. |
| 3 | **No AI traces** | No `Co-Authored-By: Claude`, no "🤖 Generated with…", no session links, no emoji branding, no unrequested trailers. The message is yours. |
| 4 | **Match the project** | The convention is detected from the last 30 subjects, `commitlint` / `commitizen` config, `.gitmessage`, and `CONTRIBUTING.md` — not assumed. Conventional Commits is the fallback only when the repository gives no signal, and that choice is flagged so you can override it. |

On top of that it applies the standard craft rules: an imperative subject that
completes *"If applied, this commit will ___"*, a 50-character target with a
72-character ceiling, no trailing period, a body wrapped at 72 columns that
explains **what and why** rather than **how**, and footers in git trailer format.

If the working tree mixes unrelated work, the skill does not paper over it with
a vague subject — it proposes a split, groups the files, and writes a complete
message for each commit, ordered so that every intermediate state still works.

## Installation

Requires [Claude Code](https://claude.com/claude-code).

```bash
/plugin marketplace add Suuunboy/commit-message-skill
```

```bash
/plugin install commit-message@commit-message-skill
```

That is it — two commands, no further configuration.

<details>
<summary>Installing from a local clone or a non-GitHub host</summary>

```bash
git clone https://github.com/Suuunboy/commit-message-skill.git
```

```bash
/plugin marketplace add ./commit-message-skill
```

`/plugin marketplace add` also accepts a full git URL, so self-hosted GitLab or
Gitea remotes work the same way:

```bash
/plugin marketplace add https://gitlab.example.com/you/commit-message-skill.git
```

</details>

> [!IMPORTANT]
> `/plugin marketplace add` looks specifically for `.claude-plugin/marketplace.json`.
> A repository that ships only `plugin.json` fails with **"Marketplace file not
> found"**, leaving users to install the skill by hand. That is why this
> repository carries both manifests: `marketplace.json` is the catalogue entry
> that makes the repo installable, and `plugin.json` is the plugin's own
> identity.

## Usage

Most of the time you do not invoke anything. The skill's description is written
to trigger on the situation, so it activates when you say things like:

- "write a commit message"
- "напиши коммит" / "что написать в коммите"
- "summarize the staged changes for git"
- "how should I word this change?"
- "review this commit message"
- "split this working tree into commits"
- "message for the revert"

It also triggers on small and obvious changes on purpose — the point is a
consistent message every time, not only for the interesting commits.

To invoke it explicitly, use the slash-command list. A plugin skill is
namespaced as `plugin:skill`, so this one appears as
**`commit-message:commit-message`**.

A typical session:

```text
> git add -p            # stage the hunks you actually want
> напиши коммит         # or: write a commit message
```

The skill reads `git diff --staged` (falling back to the unstaged diff when the
index is empty), checks the history for the convention in use, and prints one
fenced block you can copy straight into `git commit`. If it had to guess
something — the convention, or a rationale the diff does not reveal — it says so
in at most two lines under the block. Otherwise it sends the block alone.

If nothing is staged and the tree is clean, it tells you that instead of
inventing a message for a change that does not exist.

## Example output

A bug fix in a repository that uses Conventional Commits:

```text
fix(celery): stop retrying tasks that raised a validation error

The default retry policy caught every exception, so a task rejected by
the serializer was retried 12 times before landing in the dead-letter
queue. Under load this quadrupled the queue depth during incidents and
buried the real failures.

Retry only on transport and timeout errors; let validation errors fail
immediately so they surface in the error tracker on the first attempt.

Refs: #318
```

The same change in a repository whose history is capitalized imperative prose
gets the plain style instead — no type prefix, capitalized first word:

```text
Stop retrying tasks that raised a validation error
```

The skill picks the shape from the history, not from a template.

## How it works

The skill runs a five-step workflow:

1. **Read the change.** `git status --short`, then the staged diff (or the
   unstaged one). Large diffs start from `--stat` and drill into the interesting
   files. It establishes what changed, why, and what the change deliberately
   does *not* do. When the "why" is in neither the diff nor the conversation, it
   asks one short question rather than inventing a plausible-sounding reason — a
   fabricated rationale in a commit body is worse than no body at all.
2. **Detect the convention.** The last 30 subjects, plus `commitlint.config.*`,
   `.commitlintrc*`, a `commitlint` key in `package.json`, `.gitmessage`,
   `commit.template`, `.pre-commit-config.yaml`, and any commit-message section
   in `CONTRIBUTING.md`, `CLAUDE.md`, or the PR template. Where a config exists
   it is authoritative — `type-enum`, `scope-enum`, `header-max-length` and the
   rest are followed exactly.
3. **Check that this is one commit.** Unrelated work is proposed as a split
   rather than merged under a vague subject.
4. **Write the message** — subject, optional body, optional footers, under the
   rules above.
5. **Deliver it** as a fenced code block, with commentary outside the block only
   when something genuinely needs flagging.

## Reference files

| File | What it covers |
| --- | --- |
| [`conventional-commits.md`](skills/commit-message/references/conventional-commits.md) | The Conventional Commits 1.0.0 specification in full: structure, the Angular type vocabulary, how to pick a type when several fit, scopes, breaking changes via `!` and the footer, SemVer mapping, revert commits, and the `commitlint` rules that most often reject a message. |
| [`style-guide.md`](skills/commit-message/references/style-guide.md) | The seven rules in depth, the imperative-mood test, why the 50/72 limits are real, what belongs in a body and what does not, atomic commits and how to split them, translating non-English intent into idiomatic English, special cases (amend, fixup, revert, merge, WIP), an anti-pattern table, and a checklist for reviewing an existing message. |
| [`examples.md`](skills/commit-message/references/examples.md) | Worked examples for calibration: minimal no-body commits, a bug fix, a feature, a breaking change, a perf change, a refactor, a revert, plain style, a Russian-language request turned into an English message, a mixed diff split into three ordered commits, and a before/after table. |

They are plain Markdown and readable on their own — useful as a team style guide
even if you never install the plugin.

## Customizing the skill

Fork the repository and edit `skills/commit-message/SKILL.md`. The parts most
worth changing:

- **The default convention.** Step 2 falls back to Conventional Commits when the
  history gives no signal. Change that line if your default is plain style.
- **The language rule.** Rule 1 hard-codes English. If your team commits in
  another language, that is the paragraph to rewrite.
- **Subject length.** The 50-character target and the 72-character ceiling are
  stated in Step 4.
- **The trigger conditions.** The `description` in the YAML frontmatter decides
  when the skill fires. Keep it concrete and phrase-rich — it is matched against
  what you actually say, so add the wordings your team uses.

After editing, bump `version` in **both** `.claude-plugin/plugin.json` and the
plugin entry in `.claude-plugin/marketplace.json`, then push. Keep the two in
sync: `plugin.json` is the authority while the marketplace entry uses the
default `strict: true`, but a stale version string in the catalogue is a
confusing thing to hand other people.

Before pushing, check that both manifests still parse and that the skill is
discovered:

```bash
/plugin validate .
```

## Updating and uninstalling

Pull the latest version of the catalogue and the plugin:

```bash
/plugin marketplace update commit-message-skill
```

Remove the plugin but keep the marketplace:

```bash
/plugin uninstall commit-message
```

Drop the marketplace entirely:

```bash
/plugin marketplace remove commit-message-skill
```

## Compatibility

- **Claude Code** — plugin marketplaces are supported in the CLI, the desktop
  app, and the IDE extensions.
- **Git hosting** — nothing is host-specific; the skill only runs read-only git
  commands locally.
- **Conventions** — Conventional Commits, Angular, the plain seven-rule style,
  and ticket-prefixed subjects (`PROJ-123: …`) are all recognized and followed.
  `commitlint`, `commitizen`, and `gitlint` configs are read and obeyed.

## License

[MIT](LICENSE) © Ivan Afanasov
