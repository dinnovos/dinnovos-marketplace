---
name: bro-review-this
description: Code audit - accepts paths or natural descriptions. Multi-language support. Read-only.
model: opus
allowed-tools: ["Bash(read-only)", "Read", "Grep", "Glob"]
---

# Code Review

Analyze the specified code and generate a detailed report. **Read-only, doesn't modify anything. Multi-language support.**

## User Input

The user can specify what to review in various ways:

**Exact path:**
- `/bro-review-this src/components/Button.tsx`
- `/bro-review-this src/hooks/`
- `/bro-review-this internal/handlers/`

**Natural language (illustrative examples):**
- `/bro-review-this review the <name> component`
- `/bro-review-this review the <feature> services`
- `/bro-review-this analyze the <feature> module`
- `/bro-review-this review everything related to <topic>`

**No arguments:**
- `/bro-review-this` → analyzes the entire project

> **Note:** Terms like "login", "users", "payments" are just examples. Interpret what the user requests and search for the corresponding files in the project.

---

## Step 1: Interpret the Request

### If it's an exact path:
Use directly.

### If it's natural language:
Search for files matching the description:

```bash
# Explore project structure (all extensions)
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.go" -o -name "*.rs" -o -name "*.php" -o -name "*.rb" -o -name "*.java" -o -name "*.cs" -o -name "*.kt" \) | grep -v node_modules | grep -v vendor | grep -v target | grep -v __pycache__ | grep -v dist | grep -v .git

# Search by related name (replace <term> with what the user requested)
find . -type f -iname "*<term>*" | grep -v node_modules
find . -type d -iname "*<term>*" | grep -v node_modules

# Search related content
grep -ril "<term>" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.py" --include="*.go" --include="*.rs" --include="*.php" | grep -v node_modules | head -20
```

**Confirm with the user** if you find multiple matches:
```
Found these files related to "<term>":
1. src/components/[File1].tsx
2. src/hooks/[File2].ts
3. src/services/[File3].ts

Should I review all of them or a specific one?
```

If there's only one clear match, proceed directly.

---

## Step 2: Project Context

Search and read configuration and standards files:

```bash
# Project standards and guides
cat CLAUDE.md 2>/dev/null
cat AGENTS.md 2>/dev/null
cat .cursor/rules.md 2>/dev/null

# Detect stack and configuration
cat package.json pyproject.toml go.mod Cargo.toml composer.json Gemfile pom.xml 2>/dev/null

# Linting and types configuration
cat .eslintrc* 2>/dev/null
cat tsconfig.json 2>/dev/null
cat biome.json 2>/dev/null
cat pyproject.toml 2>/dev/null
```

Use this information to evaluate code against project-specific standards.

**Limit:** Maximum 50 files. If there are more, ask to narrow down.

---

## Step 3: Read and Analyze

```bash
cat [file]
wc -l [file]
```

---

## Step 4: Analysis by Categories

Review each file looking for problems in these categories, ordered by priority:

---

### P0 - CRITICAL: Bugs and Logic Errors

Problems that will cause production failures:

- **Uninitialized variables** or access to `null`/`undefined` properties
- **Impossible conditions** or inverted logic
- **Off-by-one errors** in iterations and boundaries
- **Race conditions** in asynchronous code
- **Uncaught exceptions** that can crash the application
- **Memory leaks** or unreleased resources (connections, listeners, timers)
- **Incorrect types** that will pass at runtime but fail
- **Inconsistent states** that corrupt data

#### Examples by language:

| Problem | JS/TS | Python | Go | Rust | PHP | Ruby |
|---------|-------|--------|-----|------|-----|------|
| Null access | `?.` missing | `None` check | `nil` check | `Option` | `??` | `&.` |
| Error handling | `.catch()` | `try/except` | `if err != nil` | `Result` | `try/catch` | `rescue` |
| Race conditions | ✓ | ✓ | goroutines | threads | - | threads |

**JavaScript/TypeScript:**
```javascript
// ❌ Null without check
user.profile.name
// ✅ With optional chaining
user?.profile?.name
```

**Python:**
```python
# ❌ None without check
user.profile.name
# ✅ With verification
user.profile.name if user and user.profile else None
```

**Go:**
```go
// ❌ Error ignored
result, _ := getData()
// ✅ Error handled
result, err := getData()
if err != nil { return err }
```

**Rust:**
```rust
// ❌ Unwrap in production
let value = result.unwrap()
// ✅ With handling
let value = result?
```

---

### P0 - CRITICAL: Security

Vulnerabilities that expose the system:

- **Hardcoded credentials**: API keys, passwords, tokens, secrets
- **Injection**: SQL injection, command injection, XSS
- **Unvalidated inputs**: user data used without sanitization
- **Sensitive data exposure**: logs with private information
- **Insecure configurations**: permissive CORS, disabled HTTPS
- **Vulnerable dependencies**: check package.json/requirements.txt if in scope

#### SQL Injection by language:

**JavaScript/TypeScript:**
```javascript
// ❌ Vulnerable
`SELECT * FROM users WHERE id = ${id}`
// ✅ Safe
db.query('SELECT * FROM users WHERE id = ?', [id])
```

**Python:**
```python
# ❌ Vulnerable
f"SELECT * FROM users WHERE id = {id}"
# ✅ Safe
cursor.execute("SELECT * FROM users WHERE id = %s", (id,))
```

**Go:**
```go
// ❌ Vulnerable
fmt.Sprintf("SELECT * FROM users WHERE id = %s", id)
// ✅ Safe
db.Query("SELECT * FROM users WHERE id = $1", id)
```

**PHP:**
```php
// ❌ Vulnerable
"SELECT * FROM users WHERE id = " . $id
// ✅ Safe
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
```

**Ruby:**
```ruby
# ❌ Vulnerable
"SELECT * FROM users WHERE id = #{id}"
# ✅ Safe
User.where(id: id)
```

#### Command Injection:

| Language | ❌ Vulnerable | ✅ Safe |
|----------|--------------|---------|
| JS | `exec(cmd)` | `execFile(cmd, args)` |
| Python | `os.system(f"ping {h}")` | `subprocess.run(['ping', h])` |
| Go | `exec.Command("sh", "-c", input)` | `exec.Command("ping", host)` |
| PHP | `system($cmd)` | `escapeshellarg()` |
| Ruby | `` `#{cmd}` `` | `system('cmd', arg)` |

#### XSS:

| Language | ❌ Vulnerable |
|----------|--------------|
| JS/React | `dangerouslySetInnerHTML`, `innerHTML` |
| PHP | `echo $input` without escaping |
| Ruby/Rails | `raw()`, `html_safe` misused |
| Go/templ | `template.HTML()` with input |

---

### P1 - HIGH: Performance Issues

Code that will degrade user experience:

- **O(n²) or worse operations** where O(n) solution exists
- **N+1 queries**: multiple DB/API calls in loops
- **Blocking operations** in code that should be async
- **Expensive calculations** repeated without memoization
- **Unnecessary re-renders** in React components
- **Bloated bundles**: imports that bring entire libraries
- **Missing pagination** in potentially large lists

#### O(n²) → O(n) by language:

**JavaScript/TypeScript:**
```javascript
// ❌ O(n²)
arr1.forEach(a => arr2.find(b => b.id === a.id))
// ✅ O(n)
const map = new Map(arr2.map(b => [b.id, b]))
arr1.forEach(a => map.get(a.id))
```

**Python:**
```python
# ❌ O(n²)
[x for x in list1 if x in list2]
# ✅ O(n)
set2 = set(list2)
[x for x in list1 if x in set2]
```

**Go:**
```go
// ❌ O(n²)
for _, a := range slice1 {
    for _, b := range slice2 { ... }
}
// ✅ O(n)
m := make(map[string]Item)
for _, b := range slice2 { m[b.ID] = b }
```

#### String concatenation:

| Language | ❌ Bad (in loop) | ✅ Good |
|----------|-----------------|---------|
| JS/TS | `result += str` | `parts.join('')` |
| Python | `result += s` | `''.join(strings)` |
| Go | `result += s` | `strings.Builder` |
| Rust | multiple `push_str` | `String::with_capacity` |
| Java | `result += s` | `StringBuilder` |

---

### P1 - HIGH: Typing and Contract Errors

Problems that will cause subtle bugs:

| Language | Problem | Example |
|----------|---------|---------|
| TypeScript | implicit `any` | `function fn(x)` without type |
| Python | Wrong type hint | `def fn(x: str) -> int: return x` |
| Go | `interface{}` | Use generics if Go 1.18+ |
| Rust | Trait bounds | Unnecessarily complex bounds |

- **Implicit any** or forced casts without validation
- **Optional types** used without checking existence
- **Incomplete interfaces** that don't reflect reality
- **Parameters with incorrect types** in function calls
- **Inconsistent return types** with what's actually returned

---

### P2 - MEDIUM: Quality and Maintainability

Code that will make future work difficult:

- **Duplicate code**: repeated blocks that should be extracted (>5 lines)
- **Functions too long** (>50 lines): hard to understand and test
- **Excessive nesting** (>3 levels): high cognitive complexity
- **Nested ternaries**: prefer switch/if-else for clarity
- **Non-descriptive names**: single-letter variables, cryptic abbreviations
- **Magic numbers/strings**: values without explanation or constants
- **Outdated comments**: worse than having no comments
- **High coupling**: circular dependencies, modules that know too much

### Standards by language:

| Language | Standard |
|----------|----------|
| JS/TS | ESLint, Prettier |
| Python | PEP 8, Black, Ruff |
| Go | gofmt, golint |
| Rust | rustfmt, clippy |
| PHP | PSR-12 |
| Ruby | RuboCop |

---

### P2 - MEDIUM: Project Standards Violations

If CLAUDE.md, AGENTS.md or linting configuration exists, verify:

- **Unordered imports** or without extensions (if the project requires them)
- **Arrow functions** where `function` keyword is expected (or vice versa)
- **Missing return types** on public functions
- **React components** without defined Props types
- **Error handling patterns** inconsistent with rest of code
- **Naming conventions** not followed

---

### P3 - LOW: Code Trash and Cleanup

Noise that should be removed:

| Language | Debugging to remove |
|----------|---------------------|
| JS/TS | `console.log`, `debugger` |
| Python | `print()`, `breakpoint()` |
| Go | `fmt.Println` debug |
| Rust | `println!`, `dbg!` |
| PHP | `var_dump()`, `dd()` |
| Ruby | `puts`, `binding.pry` |

- **Commented code**: if not useful, delete it; Git keeps history
- **Declared unused variables**: dead code
- **Unused imports**: bloat the bundle unnecessarily
- **Obsolete TODOs**: without date or owner, never get resolved
- **Dead functions**: never called from anywhere
- **Empty or placeholder files**: if they don't have useful content

---

### P3 - LOW: Simplification Opportunities

Optional improvements that increase elegance:

- **Logic that can be simplified** without losing clarity
- **Abstractions that can be consolidated**
- **Modern patterns** available (optional chaining, nullish coalescing)
- **Existing utilities** in the project that could be reused

---

## Step 5: Generate Report

**Respond directly in the chat with this format:**

```markdown
# BRO CODE-REVIEW REPORT

**Date:** [current date]
**Scope:** `[path or description]`
**Language(s):** [detected]
**Files analyzed:** [number]
**Total lines:** ~[approximate number]

---

## Executive Summary

| Severity | Count | Description |
|----------|-------|-------------|
| P0 Critical | X | Bugs and security - Require immediate attention |
| P1 High | X | Performance and types - Should be fixed |
| P2 Medium | X | Quality - Recommended to fix |
| P3 Low | X | Cleanup - Optional |

**Rating:** [Excellent | Good | Acceptable | Needs Work | Critical]

---

## Detailed Findings

### [path/to/file.ext]

#### P0: [Descriptive problem title]

**Lines:** XX-XX
**Category:** [Bug | Security | Performance | Types | Quality | Cleanup]

**Problem:**
[Clear description of what's wrong]

**Current code:**
```[language]
[problematic snippet with sufficient context]
```

**Suggested fix:**
```[language]
[corrected code]
```

**Impact if not fixed:**
[Concrete description of failure scenario: what will happen, under what conditions, what consequences for users/system]

---

[Repeat for each finding, grouped by file]

---

## Action Plan

**Immediate (P0):** [list of critical actions]
**Short term (P1):** [list of important improvements]
**Optional (P2-P3):** [list of minor improvements]

---

## Positive Aspects

[Good practices found in the code - not everything is criticism]

---

## Final Recommendations

[Brief list of priority actions to improve the code]
```

---

## Operating Rules

1. **Read-only**: Don't modify any files, only analyze and report
2. **Detect the language**: Adapt analysis and examples to the project's language
3. **Interpret intelligently**: Search for files related to what the user requests
4. **Confirm if ambiguous**: If there are multiple matches, ask
5. **Be specific**: Indicate exact lines, show concrete code, don't generalize
6. **Prioritize correctly**: A critical bug matters more than 10 style improvements
7. **Explain real impact**: Don't say "may cause problems", describe the exact scenario
8. **Propose idiomatic solutions**: Patterns of the detected language
9. **Avoid false positives**: If unsure, mark as "possible issue to verify"
10. **Context matters**: Test code has different rules than production
11. **Respect project standards**: Use CLAUDE.md/AGENTS.md as reference
12. **Clarity over brevity**: Explicit code is better than cryptic one-liners
13. **Recognize the good**: Mention positive practices found, not just problems
14. **Be pragmatic**: Not everything needs to be perfect, focus on what really matters
