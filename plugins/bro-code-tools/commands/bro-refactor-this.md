---
name: bro-refactor-this
description: Analyzes code for duplications, similar logic and refactoring opportunities. Multi-language support. Read-only.
model: opus
allowed-tools: ["Bash(read-only)", "Read", "Grep", "Glob"]
---

# Refactoring Analysis

Analyze code for duplications and refactoring opportunities. **Read-only, doesn't modify anything. Multi-language support.**

## User Input

The user can specify what to analyze in various ways:

**Exact path:**
- `/bro-refactor-this src/components/`
- `/bro-refactor-this src/services/userService.ts`
- `/bro-refactor-this app/services/`

**Natural language (illustrative examples):**
- `/bro-refactor-this analyze the <area> components`
- `/bro-refactor-this find duplicates in <module>`
- `/bro-refactor-this review opportunities in <feature> services`
- `/bro-refactor-this analyze everything related to <topic>`

**No arguments:**
- `/bro-refactor-this` → analyzes the entire project

> **Note:** Terms like "UI", "authentication", "payments" are just examples. Interpret what the user requests and search for the corresponding files in the project.

---

## Step 1: Interpret the Request

### If it's an exact path:
Use directly.

### If it's natural language:
Search for files matching the description:

```bash
# Explore project structure
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.go" -o -name "*.rs" -o -name "*.php" -o -name "*.rb" -o -name "*.java" -o -name "*.cs" \) \
  ! -path "*/node_modules/*" ! -path "*/vendor/*" ! -path "*/target/*" ! -path "*/__pycache__/*" ! -path "*/dist/*" ! -path "*/.git/*"

# Search by related name (replace <term> with what the user requested)
find . -type f -iname "*<term>*" | grep -v node_modules
find . -type d -iname "*<term>*" | grep -v node_modules

# Search related content
grep -ril "<term>" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.py" --include="*.go" | grep -v node_modules | head -30
```

**Confirm with the user** if you find multiple matches:
```
Found these files/folders related to "<term>":
1. src/components/[Folder1]/
2. src/services/[File1].ts
3. src/hooks/[File2].ts

Should I analyze all of them or a specific one?
```

If there's only one clear match, proceed directly.

**Limit:** Maximum 100 files. If there are more, ask to narrow down or prioritize by size.

---

## Step 2: Project Context

Search and read configuration and standards files:

```bash
# Project standards and guides
cat CLAUDE.md 2>/dev/null
cat AGENTS.md 2>/dev/null
cat .cursor/rules.md 2>/dev/null

# Detect stack
cat package.json pyproject.toml go.mod Cargo.toml composer.json Gemfile 2>/dev/null

# Linting and types configuration
cat .eslintrc* 2>/dev/null
cat tsconfig.json 2>/dev/null
cat biome.json 2>/dev/null
```

Use this information to understand the project's structure and conventions.

---

## Step 3: Read and Analyze

```bash
cat [file]
wc -l [file]
```

Read each file and perform the complete analysis.

---

## Step 4: Analysis Categories

### 1. Duplicate Code

Identical or nearly identical code blocks (>5 lines) in multiple places.

**Look for:**
- Functions with the same body
- Copy-paste blocks
- Repeated logic with different names

#### Examples by language:

**JavaScript/TypeScript:**
```typescript
// ❌ Duplicated in UserCard.tsx and AdminCard.tsx
const formatName = (user) => `${user.first} ${user.last}`
// ✅ Extract to utils/formatters.ts
export const formatName = (user: User) => `${user.first} ${user.last}`
```

**Python:**
```python
# ❌ Duplicated in user_service.py and admin_service.py
def format_name(user):
    return f"{user.first} {user.last}"
# ✅ Extract to utils/formatters.py
```

**Go:**
```go
// ❌ Duplicated in handlers/
func formatName(u User) string {
    return u.First + " " + u.Last
}
// ✅ Extract to pkg/formatters/
```

---

### 2. Similar Logic

Functions or blocks that do similar things with small variations.

**Look for:**
- Same pattern with different data
- Similar validations
- Analogous data transformations
- Handlers with repeated structure

#### Examples by language:

**JavaScript/TypeScript:**
```typescript
// ❌ Similar
function validateUser(d) {
  if (!d.email) return {error: 'Email required'}
  if (!d.pass) return {error: 'Pass required'}
}
function validateAdmin(d) {
  if (!d.email) return {error: 'Email required'}
  if (!d.pass) return {error: 'Pass required'}
  if (!d.role) return {error: 'Role required'}
}
// ✅ Unified
function validate(data, fields) {
  for (const f of fields) {
    if (!data[f]) return {error: `${f} required`}
  }
}
```

**Python:**
```python
# ❌ Similar
def get_user_by_email(email): return db.query(User).filter_by(email=email).first()
def get_user_by_id(id): return db.query(User).filter_by(id=id).first()
# ✅ Unified
def get_user_by(**kwargs): return db.query(User).filter_by(**kwargs).first()
```

**Go:**
```go
// ❌ Similar handlers
func GetUserHandler(w http.ResponseWriter, r *http.Request) { /*...*/ }
func GetProductHandler(w http.ResponseWriter, r *http.Request) { /*...*/ }
// ✅ Generic handler (Go 1.18+)
func MakeGetHandler[T any](svc Service[T]) http.HandlerFunc { /*...*/ }
```

---

### 3. Repeated Functions

Functions with the same purpose in different files.

**Look for:**
- Duplicated utilities (formatDate, capitalize, slugify, etc.)
- Repeated helpers
- Similar validation functions

| Utility | Search in |
|---------|-----------|
| formatDate | Multiple files |
| capitalize | utils/, helpers/ |
| slugify | various services |
| validateEmail | forms |

---

### 4. Similar Classes/Components

Classes or components with similar structure or behavior.

#### Examples by language:

**React:**
```tsx
// ❌ Similar components
const UserCard = ({user}) => <Card><Avatar/><Name/></Card>
const AdminCard = ({admin}) => <Card><Avatar/><Name/><Badge/></Card>
// ✅ Base component
const PersonCard = ({person, badge}) => <Card><Avatar/><Name/>{badge}</Card>
```

**Python:**
```python
# ❌ Duplicated repositories
class UserRepo:
    def find_all(self): return db.query(User).all()
class ProductRepo:
    def find_all(self): return db.query(Product).all()
# ✅ Generic base
class BaseRepo(Generic[T]):
    def find_all(self) -> List[T]: return db.query(self.model).all()
```

**Go:**
```go
// ❌ Similar services
type UserService struct { db *DB }
func (s *UserService) GetAll() []User { /*...*/ }
type ProductService struct { db *DB }
func (s *ProductService) GetAll() []Product { /*...*/ }
// ✅ Common interface
type Repository[T any] interface {
    GetAll() []T
}
```

---

### 5. Constants and Magic Numbers

Repeated hardcoded values.

**Look for:**
- Repeated magic numbers (timeouts, limits, etc.)
- Duplicated strings (URLs, messages, keys)
- Scattered configurations

| Type | JS/TS | Python | Go | Rust |
|------|-------|--------|-----|------|
| URL | `const API = ''` | `API = ''` | `const API = ""` | `const API: &str` |
| Timeout | `TIMEOUT = 30000` | `TIMEOUT = 30` | `Timeout = 30*time.Second` | `TIMEOUT: u64 = 30` |

---

### 6. Naming Inconsistencies

Variables representing the same thing with different names.

**Look for:**
- `user` vs `currentUser` vs `loggedUser` for the same thing
- `isLoading` vs `loading` vs `isLoad`
- Inconsistencies in conventions (camelCase vs snake_case)

| Aspect | JS/TS | Python | Go | Rust |
|--------|-------|--------|-----|------|
| Variables | camelCase | snake_case | camelCase | snake_case |
| Functions | camelCase | snake_case | PascalCase | snake_case |
| Constants | UPPER_SNAKE | UPPER_SNAKE | PascalCase | UPPER_SNAKE |

---

### 7. Repeated Patterns

Code structures that repeat with the same purpose.

#### Examples by language:

**JavaScript/TypeScript:**
```typescript
// ❌ Repeated: fetch + loading + error
const [loading, setLoading] = useState(false)
const [data, setData] = useState(null)
useEffect(() => { fetch()... }, [])
// ✅ Custom hook or React Query
const { data, loading } = useFetch(url)
```

**Python:**
```python
# ❌ Repeated: try + log + raise
try: result = operation()
except Exception as e:
    logger.error(e)
    raise
# ✅ Decorator
@log_errors
def operation(): ...
```

**Go:**
```go
// ❌ Repeated: error wrapping
if err != nil {
    log.Printf("error: %v", err)
    return fmt.Errorf("failed: %w", err)
}
// ✅ Helper
if err != nil {
    return errors.Wrap(err, "context")
}
```

---

## Step 5: Generate Report

**Respond directly in the chat:**

```markdown
# BRO REFACTOR ANALYSIS REPORT

**Date:** [current date]
**Scope:** `[path, description or "entire project"]`
**Language(s):** [detected]
**Files analyzed:** [number]
**Total lines of code:** ~[number]

---

## Executive Summary

| Category | Occurrences | Impact |
|----------|-------------|--------|
| Duplicate code | X | High/Medium/Low |
| Similar logic | X | High/Medium/Low |
| Repeated functions | X | High/Medium/Low |
| Similar components | X | High/Medium/Low |
| Duplicated constants | X | High/Medium/Low |
| Naming inconsistencies | X | High/Medium/Low |
| Repeated patterns | X | High/Medium/Low |

**Potentially reducible lines:** ~[number]
**Refactoring priority:** [High | Medium | Low]

---

## Duplicate Code

### Duplication #1: [Descriptive name]

**Similarity:** X%
**Affected lines:** X

| Location | Lines |
|----------|-------|
| `src/components/UserCard.tsx` | 45-67 |
| `src/components/AdminCard.tsx` | 32-54 |

**Duplicated code:**
```[lang]
// Representative snippet (max 15 lines)
```

**Suggestion:** Extract to `components/shared/ProfileCard.tsx` with props for variations.

---

[Repeat for each duplication found]

---

## Similar Logic

### Case #1: [Descriptive name]

**Files involved:**
- `src/services/userService.ts` → function `validateUser()`
- `src/services/adminService.ts` → function `validateAdmin()`

**What they do:**
[Brief description]

**Differences:**
[List of differences]

**Current code:**
```[lang]
// Snippet from the first
```
```[lang]
// Snippet from the second
```

**Suggestion:** [How to unify]

---

[Repeat for each category with findings]

---

## Recommended Action Plan

### High Priority (do first)
1. [Specific action] — Impact: X lines, X files
2. [Specific action] — Impact: X lines, X files

### Medium Priority
1. [Specific action]
2. [Specific action]

### Low Priority (nice to have)
1. [Specific action]
2. [Specific action]

---

## Additional Statistics

| Metric | Top 5 |
|--------|-------|
| Files with most duplication | [list] |
| Longest functions | [list with lines] |
| Largest files | [list with lines] |

---

## Good Practices Found

[Positive patterns, well-done abstractions, clean code identified]
```

---

## Operating Rules

1. **Read-only**: Don't modify any files, only analyze and report
2. **Detect the language**: Adapt examples and suggestions to the project's language
3. **Interpret intelligently**: Search for files related to what the user requests
4. **Confirm if ambiguous**: If there are multiple matches, ask
5. **Be exhaustive**: Read all files in scope
6. **Be specific**: Indicate exact files and lines
7. **Prioritize by impact**: What repeats most, first
8. **Suggest concrete solutions**: Don't just point out problems, propose extractions
9. **Ignore false positives**: Similar code by necessity (tests, migrations) doesn't count
10. **Consider context**: Sometimes duplication is intentional
11. **Respect project standards**: Use CLAUDE.md/AGENTS.md as reference
12. **Recognize the good**: Mention well-done abstractions
13. **Quantify the impact**: "X reducible lines" helps prioritize
