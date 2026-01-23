---
name: bro-commit-this
description: Analyzes pending changes and creates commits following Conventional Commits. Automatically splits into multiple commits when necessary for better organization.
allowed-tools: ["Bash", "Read", "Grep", "Glob"]
model: sonnet
---

# Auto Commit (Smart Multi-Commit Support)

Analyze pending files, generate descriptive messages, and execute commits. **Automatically splits into multiple commits when necessary for better organization.**

## Step 1: Check repository status

```bash
git status --short
```

### If there are no changes

If the command returns nothing, respond:

```
✅ No pending changes

Working directory is clean. Nothing to commit.
```

Stop here if there are no changes.

## Step 2: Stage files

Check if there are staged files:

```bash
git diff --cached --name-only
```

If no files are staged, add all changes:

```bash
git add -A
```

## Step 3: Analyze changes

Get the full diff of what will be committed:

```bash
git diff --cached --stat
git diff --cached
```

Read and analyze:
- Which files were modified/added/deleted
- What type of changes they are (feature, fix, refactor, docs, etc.)
- What is the main purpose of the change

---

## Step 4: Smart Commit Grouping Analysis

**CRITICAL STEP**: Determine if changes should be split into multiple commits.

### 4.1 Grouping Criteria

Analyze if changes belong to DIFFERENT logical groups:

| Criterion | Split if... | Example |
|-----------|-------------|---------|
| **Type** | Mix of feat + fix + docs | `feat(auth)` + `fix(api)` + `docs(readme)` |
| **Scope/Module** | Changes in unrelated modules | `src/auth/` + `src/payments/` + `src/users/` |
| **Purpose** | Multiple independent features | New login + Bug fix in cart |
| **Layer** | Mix of frontend + backend + config | Components + API routes + package.json |

### 4.2 Grouping Decision Matrix

```
┌─────────────────────────────────────────────────────────────────┐
│ SHOULD I SPLIT INTO MULTIPLE COMMITS?                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ ✅ SINGLE COMMIT if:                                            │
│    - All changes serve ONE purpose                              │
│    - All files are in the SAME module/feature                   │
│    - All changes are the SAME type (all feat, all fix, etc.)   │
│    - Changes are interdependent (one doesn't work without other)│
│                                                                 │
│ 🔀 MULTIPLE COMMITS if:                                         │
│    - Mix of features AND fixes                                  │
│    - Changes to UNRELATED modules                               │
│    - Independent changes that happened to be made together      │
│    - Documentation changes alongside code changes               │
│    - Config/dependency changes alongside feature changes        │
│    - Refactoring alongside new features                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.3 Grouping Algorithm

1. **Group by type first**: Separate feat/fix/docs/chore/refactor/test
2. **Then by scope**: Within each type, group by module/folder
3. **Check dependencies**: Merge groups if changes are interdependent
4. **Validate atomic commits**: Each commit should be deployable on its own

### 4.4 Example Groupings

**Scenario**: Changes in `src/auth/login.ts`, `src/auth/logout.ts`, `src/cart/checkout.ts`, `README.md`, `package.json`

**Analysis**:
```
Group 1: feat(auth) - login.ts, logout.ts (same module, same purpose)
Group 2: fix(cart) - checkout.ts (different module)
Group 3: docs - README.md (documentation)
Group 4: chore(deps) - package.json (dependencies)
```

**Result**: 4 separate commits

---

## Step 5: Generate commit message(s)

Generate a message following **Conventional Commits**:

### Format

```
<type>(<scope>): <short description>

<optional body - what and why>

<optional footer - breaking changes, issues>
```

### Available types

| Type | When to use |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code change that doesn't add a feature or fix a bug |
| `docs` | Documentation changes |
| `style` | Formatting, missing semicolons, etc. (doesn't affect logic) |
| `test` | Adding or modifying tests |
| `chore` | Maintenance tasks, dependencies, config |
| `perf` | Performance improvements |
| `ci` | CI/CD changes |
| `build` | Changes to build system or external dependencies |
| `revert` | Revert previous commit |

### Message rules

1. **Short description**: maximum 50 characters, imperative mood, no period at end
2. **Scope**: optional, indicates affected module/component
3. **Body**: optional, explains what and why (not how)
4. **Language**: match the predominant language in previous commits

### Examples

```bash
# Simple
git commit -m "feat(auth): add Google login"

# With body
git commit -m "fix(api): fix timeout on long requests

The 30s timeout was insufficient for large uploads.
Increased to 120s for files up to 100MB."

# Breaking change
git commit -m "refactor(db)!: migrate from MySQL to PostgreSQL

BREAKING CHANGE: requires new connection configuration"
```

## Step 6: Execute the commit(s)

### 6.1 Single Commit (if changes are cohesive)

```bash
git add -A
git commit -m "<generated message>"
```

### 6.2 Multiple Commits (if changes should be split)

**Execute commits in logical order** (usually: chore/deps → refactor → feat → fix → docs → test):

```bash
# First, unstage everything
git reset HEAD

# Commit 1: [type1]
git add <files-for-commit-1>
git commit -m "<message-1>"

# Commit 2: [type2]
git add <files-for-commit-2>
git commit -m "<message-2>"

# Commit N: [typeN]
git add <files-for-commit-N>
git commit -m "<message-N>"
```

### 6.3 Commit Order Priority

When splitting, follow this order for cleaner history:

1. `chore(deps)` - Dependencies first (others may depend on them)
2. `build` / `ci` - Build configuration
3. `refactor` - Refactoring (prepares codebase)
4. `feat` - New features
5. `fix` - Bug fixes
6. `perf` - Performance improvements
7. `test` - Tests
8. `docs` - Documentation last

---

## Step 7: Confirm

### Single Commit Result:

```
✅ Commit created successfully

**Hash:** [short hash]
**Message:** [commit message]

**Files included:**
- [file list]

**Next step:** `git push` to upload the changes
```

### Multiple Commits Result:

```
✅ Created X commits successfully

**Commits:**
1. [hash1] type(scope): message 1
   - file1.ts, file2.ts

2. [hash2] type(scope): message 2
   - file3.ts

3. [hash3] type(scope): message 3
   - file4.ts, file5.ts

**Summary:**
- Total commits: X
- Total files: Y
- Total lines: +A / -B

**Next step:** `git push` to upload all commits
```

## Rules

1. **Analyze ALL changes** before deciding the type and grouping
2. **One commit = one purpose** — Automatically split if changes are diverse
3. **Clear message** — Someone should understand what was done without seeing the code
4. **Don't use generic messages** — Avoid "update", "fix", "changes"
5. **Detect language** — Use the predominant language from previous commits
6. **Smart splitting by default** — If changes clearly belong to different groups, split them automatically without asking
7. **Ask only when ambiguous** — Only ask user when grouping is unclear
8. **DON'T add "Co-Authored-By"** — Message should be clean, without AI co-authorship lines
9. **Atomic commits** — Each commit should be independently deployable/reversible
10. **Logical order** — When splitting, commit in dependency order (deps → refactor → feat → fix → docs)

## Pre-commit verification

Before executing the commit, check:

```bash
# Check if linter is configured
npm run lint 2>/dev/null || yarn lint 2>/dev/null || true
```

If the linter fails, inform the user but don't stop the commit (it's their decision).

## Special cases

### If sensitive files are detected

If you see files like `.env`, `*.key`, `credentials.*`, `*secret*`:

```
⚠️ WARNING: Detected potentially sensitive files:
- .env.local
- config/secrets.json

Are you sure you want to include them in the commit?
These files should normally be in .gitignore
```

Wait for confirmation before continuing.

### If changes are too large

If there are more than 500 lines changed or more than 20 files:

```
ℹ️ This commit includes many changes:
- X files modified
- +Y lines / -Z lines

Analyzing for optimal split...
```

Then proceed to analyze and split automatically based on the grouping criteria.

---

## Auto-Split Examples

### Example 1: Feature + Fix + Docs

**Changed files:**
- `src/auth/login.ts` (new feature)
- `src/api/users.ts` (bug fix)
- `README.md` (updated docs)

**Result:** 3 commits
```bash
git add src/auth/login.ts
git commit -m "feat(auth): add login functionality"

git add src/api/users.ts
git commit -m "fix(api): resolve null pointer in user endpoint"

git add README.md
git commit -m "docs: update README with auth instructions"
```

### Example 2: Refactor + Feature in same module

**Changed files:**
- `src/utils/helpers.ts` (refactored)
- `src/utils/newHelper.ts` (new file)
- `src/utils/index.ts` (exports updated)

**Result:** 2 commits (refactor first, then feature)
```bash
git add src/utils/helpers.ts
git commit -m "refactor(utils): simplify helper functions"

git add src/utils/newHelper.ts src/utils/index.ts
git commit -m "feat(utils): add new date formatting helper"
```

### Example 3: Dependencies + Code changes

**Changed files:**
- `package.json` (new dependency)
- `package-lock.json` (lockfile)
- `src/services/api.ts` (uses new dependency)

**Result:** 2 commits (deps first)
```bash
git add package.json package-lock.json
git commit -m "chore(deps): add axios for HTTP requests"

git add src/services/api.ts
git commit -m "feat(api): implement new HTTP client with axios"
```

### Example 4: All related changes (single commit)

**Changed files:**
- `src/components/Button.tsx` (new component)
- `src/components/Button.test.tsx` (tests for component)
- `src/components/Button.css` (styles for component)

**Result:** 1 commit (all related to same feature)
```bash
git add src/components/Button.tsx src/components/Button.test.tsx src/components/Button.css
git commit -m "feat(components): add Button component with tests and styles"
```

---

## When to Ask the User

Only ask for confirmation when:

1. **Ambiguous grouping** — Changes could logically go either way
2. **Interdependent across types** — e.g., fix that requires a refactor
3. **Mixed scope in same type** — e.g., feat in both auth and payments that might be related
4. **Very large splits** — More than 5 potential commits

```
🤔 I detected X logical groups, but some might be related:

1. feat(auth): login.ts, session.ts
2. feat(api): auth-endpoint.ts  ← might be related to #1?
3. fix(auth): logout.ts
4. docs: README.md

Options:
A) Split into 4 commits (recommended)
B) Merge #1 and #2 into single feat commit
C) Single commit with everything
D) Let me decide which files go where

Your choice?
```
