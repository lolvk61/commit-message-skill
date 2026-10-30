# Conventional Commits reference

Use when the project adopts Conventional Commits or no convention is available.
Practical summary, not the complete specification:
https://www.conventionalcommits.org/en/v1.0.0/

```text
<type>[optional scope][optional !]: <description>

[optional body]

[optional footers]
```

| Type | Primary intent |
| --- | --- |
| `feat` | Add a capability |
| `fix` | Correct incorrect behavior |
| `perf` | Improve performance |
| `refactor` | Restructure without changing behavior |
| `docs` | Change documentation |
| `test` | Add or correct tests |
| `style` | Formatting without semantic changes |
| `build` | Build system or dependencies |
| `ci` | Automation configuration |
| `chore` | Other maintenance |
| `revert` | Undo a change, when the project uses this form |

The specification assigns release meaning to `feat`, `fix`, and breaking
changes. Other type names and release effects depend on project tooling.
Use project-configured types and scopes instead of this default vocabulary.
Choose by primary intent; a feature's tests do not make it a `test` commit.

Scopes are optional. Reuse existing module/package names; omit vague scopes.
Lowercase descriptions are common, not a specification requirement. Project
commitlint rules determine case, allowed types/scopes, and length limits.

Use `!` before the colon and/or a `BREAKING CHANGE:` footer for breaking changes.
Explain consumer impact and any known migration:

```text
feat(api)!: return timestamps as ISO 8601 strings

BREAKING CHANGE: timestamp fields now contain strings instead of Unix
integers. Parse these values as ISO 8601 timestamps.
```

Footers follow a blank line. Tokens use hyphens instead of spaces, except for
`BREAKING CHANGE` (`BREAKING-CHANGE` is also accepted):

```text
Refs: #123
Closes #456
```

Use closing keywords only for issues actually resolved. Do not invent references,
identities, or attestations in trailers.

For reverts, follow the repository: its `revert:` form or Git's generated
`Revert "<original subject>"` with the original SHA. For autosquash, the exact
`fixup! <target subject>` or `squash! <target subject>` form takes precedence
over ordinary subject style and length preferences.
