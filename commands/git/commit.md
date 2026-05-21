---
name: 'commit'
description: 'Analyze git changes, generate a conventional commit, commit, and push'
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

Show file list (from `changed_files`):

| File | Status |
| ---- | ------ |
| ...  | ...    |

---

### 2. Generate commit message

Format:

```text
type(scope): [image] message
```

Rules:

- type (required): feat / fix / refactor / docs / test / chore / perf
- scope (required):
  - rpc-ums → ums
  - rpc-pay → payment
  - unclear → STOP and ask user
- message (required):
  - lowercase
  - imperative
  - ≤ 72 chars
  - no ending period
- [image] only if any \*.go file changed

---

### 3. Validate (MUST DO BEFORE CONTINUE)

Check ALL rules:

- type exists
- scope exists
- message valid
- format correct
- length ≤ 72
- lowercase enforced
- image rule correct

If any rule fails → STOP immediately.

---

### 4. Confirm

Show commit message:

```text
<message>
```

Ask:

Use `AskUserQuestion`:

- header: `Commit`
- question: `Commit and push?`
- options: `Yes` / `No`

If `No` → stop.

If `Yes` → continue.

---

### 5. Execute

```bash
git add -A
git commit -m "<message>"
git push
```

If any step fails → show error and stop.

---

### 6. Done

```text
## Commit Complete

<message>
✓ done
```

---

## Guardrails

- Always check git status before anything
- Always validate rules before confirmation
- Never guess scope
- Scope is mandatory
- Keep message ≤ 72 chars
- Must confirm before push
- Stop on any error
