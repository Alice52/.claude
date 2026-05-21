---
name: "commit"
description: "Analyze git changes, generate a conventional commit, commit, and push"
category: Workflow
tags: [git, commit, workflow]
---

Analyze current git changes and generate a commit message.

---

## Steps

### 1. Check changes

```bash
git status --short
```

If empty:

```text
Nothing to commit.
```

Stop.

---

### 2. Generate commit message

Format:

```text
type(scope): [image] message
```

Rules:

- type (required): feat / fix / refactor / docs / test / chore / perf
- scope (required): infer from changed files
  - rpc-ums → ums
  - rpc-pay → payment
  - internal/auth → auth
  - unclear → ask user and stop
- message(required):
  - lowercase
  - imperative
  - ≤ 72 chars
  - no ending period
- [image]&#58; only if any *.go file changed

---

If any rule cannot be satisfied → stop and ask.

---

### 3. Confirm

Show:

```text
<commit-message>
```

Use `AskUserQuestion`:

- **header**: `Commit`
- **question**: `Commit and push?`
- **options**: `Yes` / `No`
- **preview** (on `Yes`):

If `No` → stop.

---

### 4. Execute

```bash
git add -A
git commit -m "<message>"
git push
```

If any step fails → show error and stop.

---

### 5. Done

```text
## Commit Complete

<message>
✓ done
```

---

## Guardrails

- Always check git status and diff first
- Scope is mandatory
- Never guess scope
- Keep message ≤72 chars
- Must confirm before push
- Stop on any error