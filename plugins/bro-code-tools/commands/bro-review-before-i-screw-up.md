---
name: bro-review-before-i-screw-up
description: Complete audit of pending changes before commit - detects bugs, quality issues, security problems and improvement opportunities. Multi-language support.
model: opus
allowed-tools: ["Bash", "Read", "Grep"]
---

# Complete Pre-Commit Review

Exhaustively analyze all pending changes before committing. Detect critical issues, evaluate code quality and suggest concrete improvements. **Multi-language support.**

## Step 1: Identify Pending Changes

Run these commands to get the current state:

```bash
# Modified files (tracked)
git diff --name-only HEAD

# New files (untracked)
git ls-files --others --exclude-standard

# Staged files
git diff --cached --name-only
```

### If there are no changes

If all commands return empty, stop and respond:

```
✅ Clean working directory

No pending changes to review. Nothing to audit.
```

**Don't continue if there are no files to review.**

## Step 2: Get Complete Context

```bash
# Full diff of unstaged changes
git diff HEAD

# Diff of staged changes (if any)
git diff --cached

# See project structure for context
ls -la

# Detect stack
cat package.json pyproject.toml go.mod Cargo.toml composer.json Gemfile 2>/dev/null

# Project standards
cat CLAUDE.md AGENTS.md 2>/dev/null
```

If `CLAUDE.md`, `AGENTS.md`, `README.md` or project configuration files (`.eslintrc`, `tsconfig.json`, etc.) exist, read them to understand project standards.

## Step 3: Analysis by Categories

Review each modified file looking for problems in these categories, ordered by priority:

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

| Language | Problem | Example |
|----------|---------|---------|
| JS/TS | null/undefined without check | `user.profile.name` without optional chaining |
| Python | Mutable default args | `def f(lst=[])` |
| Go | Ignored error | `result, _ := fn()` |
| Rust | unwrap() in production | `value.unwrap()` |
| PHP | Weak comparison | `$a == "0"` vs `$a === "0"` |

---

### P0 - CRITICAL: Security

Vulnerabilities that expose the system:

- **Hardcoded credentials**: API keys, passwords, tokens, secrets
- **Injection**: SQL injection, command injection, XSS
- **Unvalidated inputs**: user data used without sanitization
- **Sensitive data exposure**: logs with private information
- **Insecure configurations**: permissive CORS, disabled HTTPS
- **Vulnerable dependencies**: if package.json/requirements.txt was modified

#### Injections by language:

| Language | SQL Injection | Command Injection | XSS |
|----------|---------------|-------------------|-----|
| JS/TS | Template strings in queries | `exec(cmd + input)` | `innerHTML = input` |
| Python | f-strings in queries | `os.system(f"cmd {input}")` | N/A |
| Go | Concatenation in queries | `exec.Command("sh", "-c", input)` | `template.HTML()` |
| Rust | format! in queries | N/A | N/A |
| PHP | Concatenation in queries | `system($input)` | `echo $_GET['x']` |

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

#### Common problems by language:

| Language | Common problem | Solution |
|----------|----------------|----------|
| JS/TS | await in loop | Promise.all |
| Python | list concat in loop | ''.join() |
| Go | append without pre-allocate | make([]T, 0, cap) |
| Rust | unnecessary .clone() | use references |
| PHP | query in loop | whereIn() |

---

### P1 - HIGH: Typing and Contract Errors

Problems that will cause subtle bugs:

| Language | Problem | Solution |
|----------|---------|----------|
| TypeScript | implicit any | explicit types |
| Python | no type hints | add hints |
| Go | interface{} without check | type assertion with ok |
| PHP | no type declarations | PHP 7+ types |

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

Noise that should be removed before commit:

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

## Step 4: Generate Report

Use this exact format:

```markdown
# BRO PRE-COMMIT REVIEW REPORT

**Date:** [current date]
**Branch:** [branch name]
**Language(s):** [detected]
**Files analyzed:** [number]
**Lines modified:** ~[approximate number]

---

## Executive Summary

| Severity | Count | Description |
|----------|-------|-------------|
| P0 Critical | X | Bugs and security - BLOCK the commit |
| P1 High | X | Performance and types - Should be fixed |
| P2 Medium | X | Quality - Recommended to fix |
| P3 Low | X | Cleanup - Optional |

**Verdict:** [APPROVED | WITH OBSERVATIONS | REJECTED]

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

## Pre-Commit Checklist

- [ ] All P0 (critical) are resolved
- [ ] P1 (high) are resolved or have justification
- [ ] No credentials or sensitive data
- [ ] No debugging code (console.log, debugger)
- [ ] Tests pass (if applicable)
- [ ] Code compiles without errors

---

## Final Recommendations

[Brief list of priority actions before committing]
```

---

## Operating Rules

1. **Be specific**: Indicate exact lines, show concrete code, don't generalize
2. **Detect the language**: Adapt analysis and examples to the project's language
3. **Prioritize correctly**: A critical bug matters more than 10 style improvements
4. **Explain real impact**: Don't say "may cause problems", describe the exact scenario
5. **Propose idiomatic solutions**: Patterns of the detected language
6. **Avoid false positives**: If unsure, mark as "possible issue to verify"
7. **Context matters**: Test code has different rules than production
8. **Preserve functionality**: Improvement suggestions should never change behavior
9. **Clarity over brevity**: Explicit code is better than cryptic one-liners
10. **Respect project standards**: If CLAUDE.md, AGENTS.md or linting config exists, follow it
11. **Be pragmatic**: Not everything needs to be perfect, focus on what really matters
