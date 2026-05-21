---
name: "commit"
description: "Analyze git changes, generate a conventional commit, commit, and push"
category: Workflow
tags: [git, commit, workflow]
---

Analyze current git changes and generate a commit message.

## Steps

### 1. Check changes

```bash
git status --short
```

No changes → `Nothing to commit.` Stop.

```bash
git diff HEAD
```

### 2. Generate message

Format: `type(scope): [image] message`

- **type**: feat / fix / refactor / docs / test / chore / perf
- **scope**: infer from changed files (e.g. `rpc-ums` → `ums`, `internal/auth` → `auth`). Multiple scopes → pick dominant one. Unclear → ask.
- **tag**: include `[image]` iff any `*.go` file changed
- **message**: lowercase, imperative, ≤72 chars, no ending period, describe intent

Examples: `feat(user): [image] support avatar crop` / `refactor(admin): simplify rpc client wiring`

### 3. Confirm

Use `AskUserQuestion`:

- **header**: `Commit`
- **question**: `Commit and push?`
- **options**: `Yes` / `No`
- **preview** (on `Yes`):

  ```
  <commit-message>
  ```

If `No` → stop.

### 4. Commit and push

```bash
git add -A && git commit -m "<message>" && git push
```

Any step fails → show error, stop.

### 5. Done

```text
## Commit Complete

<message>
✓ committed and pushed
```

---

## Guardrails

- Always inspect git status and diff before generating message
- Never invent scope — ask if unclear
- Message ≤72 chars, lowercase, imperative
- Never push unless commit succeeds