---
name: "release"
description: "Prepare and create the next release tag"
category: Workflow
tags: [git, release, tag]
---

Prepare a release tag.

## Execution Principles

1. Think Before Coding — verify repository state before tagging
2. Simplicity First — only increment the next valid semantic version
3. Surgical Changes — only operate on git state, no code changes
4. Goal-Driven Execution — never create or push a tag without explicit confirmation

---

## Steps

### 1. Fetch and get latest tag

```bash
git fetch --tags --prune
git tag --sort=-version:refname | head -n 1
```

If no tags: `No existing tags. Starting from: v0.0.1`

### 2. Compute next tag

Format: `vX.Y.Z` or `vX.Y.Z-prod`. X/Y/Z integers, Z/Y max 9.

```
v1.2.3       → v1.2.4
v1.2.9       → v1.3.0
v1.9.9       → v2.0.0
v1.2.3-prod  → v1.2.4-prod
```

Increment Z; if Z > 9 → Z=0, increment Y; if Y > 9 → Y=0, increment X.

Invalid format → `Unable to parse. Expected: vX.Y.Z or vX.Y.Z-prod` Stop.

### 3. Confirm

Show preview and use `AskUserQuestion` tool:

- **header**: `Release`
- **question**: `Create and push tag <next>?`
- **options**: `Yes` / `No`
- **preview** (on `Yes` option):

  ```
  Current: <current>
  Next:    <next>
  ```

If user selects `No` → stop.

### 4. Create and push

```bash
git tag <next> && git push origin <next>
```

Either fails → abort, show error.

### 5. Done

```text
## Release Complete

<current> → <next>
✓ tagged and pushed
```

---

## Guardrails

- Fetch remote tags before computing, never rely on local-only tags
- Never assume tag format valid
- Always show preview and require explicit confirmation
- Abort on any git failure