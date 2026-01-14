---
name: bro-commit-this
description: Analyzes pending changes and creates a commit with a descriptive message following Conventional Commits
allowed-tools: ["Bash", "Read", "Grep"]
model: haiku
---

# Auto Commit

Analyze pending files, generate a descriptive message, and execute the commit.

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

## Step 4: Generate commit message

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

## Step 5: Execute the commit

Once the diff is analyzed and the appropriate message is generated:

```bash
git commit -m "<generated message>"
```

## Step 6: Confirm

Show the result:

```
✅ Commit created successfully

**Hash:** [short hash]
**Message:** [commit message]

**Files included:**
- [file list]

**Next step:** `git push` to upload the changes
```

## Rules

1. **Analyze ALL changes** before deciding the type
2. **One commit = one purpose** — If changes are too diverse, suggest splitting them
3. **Clear message** — Someone should understand what was done without seeing the code
4. **Don't use generic messages** — Avoid "update", "fix", "changes"
5. **Detect language** — Use the predominant language from previous commits
6. **If there are many diverse changes**, ask the user if they want:
   - A single general commit
   - Split into multiple commits
7. **DON'T add "Co-Authored-By"** — Message should be clean, without AI co-authorship lines

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

Would you prefer to split it into smaller commits?
```
