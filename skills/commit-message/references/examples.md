# Worked examples

Read this for calibration on length, tone, and how much body a change deserves.

## Contents

- [Minimal commits with no body](#minimal-commits-with-no-body)
- [A bug fix that needs a body](#a-bug-fix-that-needs-a-body)
- [A feature](#a-feature)
- [A breaking change](#a-breaking-change)
- [A performance change](#a-performance-change)
- [A refactor](#a-refactor)
- [A revert](#a-revert)
- [Plain style, no Conventional Commits](#plain-style-no-conventional-commits)
- [From a Russian-language request](#from-a-russian-language-request)
- [Splitting a mixed diff](#splitting-a-mixed-diff)
- [Before and after](#before-and-after)

## Minimal commits with no body

```
docs: correct spelling of CHANGELOG
```

```
build(deps): bump cryptography from 42.0.5 to 42.0.8
```

```
test(billing): add case for zero-amount invoices
```

```
ci: run the test matrix on Python 3.13
```

## A bug fix that needs a body

```
fix(celery): stop retrying tasks that raised a validation error

The default retry policy caught every exception, so a task rejected by
the serializer was retried 12 times before landing in the dead-letter
queue. Under load this quadrupled the queue depth during incidents and
buried the real failures.

Retry only on transport and timeout errors; let validation errors fail
immediately so they surface in the error tracker on the first attempt.

Refs: #318
```

## A feature

```
feat(search): add semantic reranking to the retrieval pipeline

Lexical search alone returned the right document outside the top five
for most multi-sentence questions, which is below the cutoff the answer
prompt uses.

Rerank the top 50 lexical hits with a cross-encoder before truncating
to the final five. This adds roughly 120 ms per query, which stays
inside the existing 2 s budget, and is disabled by default behind
SEARCH_RERANK_ENABLED so it can be rolled out per tenant.
```

## A breaking change

```
feat(api)!: return ISO 8601 timestamps in all responses

Timestamps were serialized as Unix epoch integers in some endpoints and
as ISO strings in others, depending on which serializer the view used.
Clients had to branch per endpoint.

BREAKING CHANGE: every timestamp field is now an ISO 8601 string in
UTC. Clients parsing integers must switch to a date parser. The v1
endpoints keep the old format until 2026-12-01.
```

## A performance change

```
perf(reports): batch the per-row permission check

Rendering a 5,000-row report issued one permission query per row, which
put the p95 at 14 s and saturated the connection pool during the
morning peak.

Resolve permissions once per distinct object type and cache the result
for the duration of the request. The p95 drops to 1.1 s on the same
fixture; output is byte-identical, covered by the existing snapshot
test.
```

## A refactor

```
refactor(langflow): extract document parsing into a shared base class

DocxParser and ImageParser had drifted into three near-identical copies
of the chunking and metadata code, and a fix applied to one of them was
missed in the others twice.

Move the shared path into BaseDocumentParser and leave only format
specific extraction in the subclasses. No behavior change; the parser
test suite passes unchanged.
```

## A revert

```
Revert "feat(auth): switch session storage to Redis"

This reverts commit 4f2a1c8e9b3d5a7c1e0f2b4d6a8c0e2f4b6d8a0c.

Sessions were dropped whenever Redis failed over, logging every active
user out. The change goes back in once replication is configured with
AOF persistence.

Refs: #401
```

## Plain style, no Conventional Commits

For a repository whose history is capitalized imperative prose:

```
Reject expired refresh tokens during rotation

Rotation accepted a token whose exp had already passed, so a leaked
token stayed usable indefinitely as long as it was rotated before the
access token expired.

Validate exp before issuing the new pair and revoke the whole token
family on a replay attempt.
```

Note the capitalized first word and the absence of a type prefix — the only
differences from the Conventional Commits version of the same change.

## From a Russian-language request

> Пользователь: «Сделал так, что при загрузке документа в базу знаний теперь
> проверяется размер файла, если больше 50 мегабайт — возвращаем 413 вместо
> того чтобы падать по таймауту. Напиши коммит.»

```
feat(upload): reject documents larger than 50 MB with 413

Files above roughly 50 MB kept the worker busy until the gateway
timeout and returned a 504, so the client could not tell an oversized
file from an outage.

Check Content-Length before the file is read and return 413 with the
limit in the error body. The limit is configurable through
MAX_UPLOAD_SIZE_MB.
```

What the translation did: `база знаний` never appears as "knowledge base"
because the scope `upload` already locates the change; `падать по таймауту`
became the concrete symptom rather than a literal "fall by timeout"; the limit
is stated as a number in both the subject and the body.

## Splitting a mixed diff

A working tree containing a bug fix, a dependency bump, and a formatting sweep
should be presented as three commits rather than one:

**Commit 1** — `pyproject.toml`, `poetry.lock`

```
build(deps): bump httpx from 0.27.0 to 0.28.1

0.28 fixes the connection leak on cancelled requests that showed up as
growing socket counts in the worker containers.
```

**Commit 2** — `app/clients/webhook.py`, `tests/test_webhook.py`

```
fix(webhook): close the client on delivery timeout

The client was left open when delivery timed out, so each failed
webhook leaked a connection until the worker was recycled.
```

**Commit 3** — 34 files, formatting only

```
style: apply ruff format to the app package
```

Ordering matters: the dependency bump comes first because the fix relies on the
new version, and the formatting sweep comes last so it does not bury the other
two in `git log -p`.

## Before and after

| Before | After |
| --- | --- |
| `fixes` | `fix(parser): handle empty input without raising` |
| `Updated the readme.` | `docs: document the RERANK_ENABLED flag` |
| `feat: new endpoint` | `feat(api): add GET /v1/documents/{id}/chunks` |
| `refactor: cleanup` | `refactor(tasks): drop the unused retry wrapper` |
| `Fixed a bug where the user could not log in if their email had uppercase letters in it` | `fix(auth): normalize email case before lookup` |
| `feat: add feature and tests and docs` | Three commits, or one `feat` commit if they ship together |
