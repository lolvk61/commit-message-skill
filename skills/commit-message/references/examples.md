# Worked examples

These illustrative details are not evidence about the user's repository.

## Supplied prose without repository access

Input: «Исправил падение при пустой корзине. Напиши сообщение коммита».
No history or diff is supplied. English and Conventional Commits are defaults:

```text
fix(checkout): handle an empty cart without crashing
```

Do not add a root cause, tests, or issue number absent from the input.
If the user explicitly asks for Russian:

```text
fix(checkout): устранить сбой при пустой корзине
```

## Staged changes with unrelated unstaged work

Evidence: the staged patch rejects expired refresh tokens; the unstaged patch
changes dashboard colors. History uses plain imperative subjects:

```text
Reject expired refresh tokens during rotation
```

The dashboard changes do not belong in the message for the index.

## A supported body

Evidence: the diff removes retries for validation errors. The user explains
that invalid requests were retried until they reached the dead-letter queue.

```text
fix(worker): stop retrying validation errors

Invalid requests were retried until they reached the dead-letter queue.
Fail them immediately so they can be corrected without repeated work.
```

## One feature across code, tests, and documentation

Evidence: an upload-size limit, boundary tests, and API documentation ship
together. Treat these as one logical change:

```text
feat(upload): reject files larger than 50 MB
```

Do not invent how size is checked, an environment variable, or an HTTP status
unless the evidence establishes it.

## Unrelated work

Evidence: a client timeout fix and an independent formatting sweep. Suggest
two groups without changing the index.

Group 1: timeout handling and its regression test.

```text
fix(webhook): close the client after a delivery timeout
```

Group 2: formatting-only hunks.

```text
style: format the application package
```

## Wording versus execution

"Write a commit message" produces text only. "Commit the staged changes" also
authorizes creating that commit, subject to environment permissions. A successful
commit response may include its actual SHA; a draft must not claim a commit was
created. Neither request independently authorizes pushing.
