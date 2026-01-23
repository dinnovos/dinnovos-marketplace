---
name: bro-check-standards
description: Audits project architecture and validates compliance with CLAUDE.md and AGENTS.md standards. Multi-language support. Read-only.
model: opus
allowed-tools: ["Bash(read-only)", "Read", "Grep", "Glob"]
---

# Architecture and Standards Compliance Audit

Analyze project architecture and validate compliance with established standards in CLAUDE.md and AGENTS.md. **Read-only, doesn't modify anything. Multi-language support.**

## User Input

The user can specify what to audit in various ways:

**Exact path:**
- `/bro-check-standards src/`
- `/bro-check-standards src/modules/payments/`
- `/bro-check-standards app/services/`

**Natural language (illustrative examples):**
- `/bro-check-standards check the <module> architecture`
- `/bro-check-standards validate <feature> against standards`
- `/bro-check-standards audit compliance of <area>`
- `/bro-check-standards review if <module> follows the rules`

**No arguments:**
- `/bro-check-standards` → audits the entire project

> **Note:** Terms like "authentication", "payments", "users" are just examples. Interpret what the user requests and search for the corresponding files in the project.

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

Should I audit all of them or a specific one?
```

If there's only one clear match, proceed directly.

---

## Step 2: Read Standards Files (CRITICAL)

**This step is mandatory.** Read all standards and configuration files before analyzing code:

```bash
# Primary standards files
cat CLAUDE.md 2>/dev/null
cat claude.md 2>/dev/null
cat AGENTS.md 2>/dev/null
cat agents.md 2>/dev/null
cat AGENT.md 2>/dev/null
cat agent.md 2>/dev/null

# Secondary standards and guides (multi-tool support)
cat .cursor/rules.md 2>/dev/null
cat .cursor/rules/*.md 2>/dev/null
cat .cursorrules 2>/dev/null
cat .github/copilot-instructions.md 2>/dev/null
cat .github/instructions/*.md 2>/dev/null
cat .junie/guidelines.md 2>/dev/null
cat .continuerules 2>/dev/null
cat .continue/rules/*.md 2>/dev/null
cat CONTRIBUTING.md 2>/dev/null
cat ARCHITECTURE.md 2>/dev/null
cat CONVENTIONS.md 2>/dev/null
cat STYLE_GUIDE.md 2>/dev/null
cat ADR/*.md 2>/dev/null
cat docs/adr/*.md 2>/dev/null

# Project configuration
cat package.json pyproject.toml go.mod Cargo.toml composer.json Gemfile pom.xml 2>/dev/null

# Linting and formatting configuration
cat .eslintrc* 2>/dev/null
cat eslint.config.* 2>/dev/null
cat tsconfig.json 2>/dev/null
cat biome.json 2>/dev/null
cat .prettierrc* 2>/dev/null
cat .editorconfig 2>/dev/null
cat pyproject.toml 2>/dev/null
cat .golangci.yml 2>/dev/null
cat rustfmt.toml 2>/dev/null
cat .rubocop.yml 2>/dev/null
cat phpcs.xml 2>/dev/null
```

**IMPORTANT:** If no CLAUDE.md or AGENTS.md exists, inform the user and suggest creating them. Still proceed with architectural analysis using industry best practices.

---

## Step 3: Extract Rules from Standards Files

Parse the standards files and extract:

### From CLAUDE.md / AGENTS.md:
- **Naming conventions**: file names, variables, functions, classes
- **Folder structure rules**: where to place components, services, utils, etc.
- **Import conventions**: order, aliases, extensions
- **Code patterns**: preferred patterns (functional vs class, arrow vs function)
- **Error handling conventions**: how to handle errors
- **Type conventions**: strict types, interfaces vs types
- **Testing conventions**: file naming, structure, coverage
- **Documentation conventions**: comments, JSDoc, docstrings
- **Commit conventions**: message format, scope
- **Forbidden patterns**: what NOT to do

Create a mental checklist of all rules to verify.

---

## Step 4: Analyze Project Structure

```bash
# Directory structure
find . -type d -maxdepth 4 | grep -v node_modules | grep -v vendor | grep -v target | grep -v __pycache__ | grep -v .git | sort

# File distribution by type
find . -type f \( -name "*.ts" -o -name "*.tsx" \) | grep -v node_modules | wc -l
find . -type f \( -name "*.js" -o -name "*.jsx" \) | grep -v node_modules | wc -l
find . -type f -name "*.py" | grep -v __pycache__ | wc -l
find . -type f -name "*.go" | grep -v vendor | wc -l

# Entry points
ls -la src/index.* src/main.* src/app.* main.* app.* 2>/dev/null
```

---

## Step 5: Architectural Analysis

### 5.1 Verify Folder Structure Compliance

Compare actual structure against documented structure in CLAUDE.md/AGENTS.md:

| Expected (per standards) | Actual | Status |
|--------------------------|--------|--------|
| `src/components/` | ✓ exists | ✅ Compliant |
| `src/services/` | ✗ missing | ❌ Violation |
| `src/utils/` | ✓ exists but wrong location | ⚠️ Partial |

### 5.2 Verify Architectural Patterns

Check if the code follows the documented architectural pattern:

**Layered Architecture:**
- Presentation layer only calls business layer
- Business layer only calls data layer
- No layer skipping

**Clean Architecture:**
- Dependencies point inward
- Domain has no external dependencies
- Use cases orchestrate entities

**MVC / MVP / MVVM:**
- Proper separation of concerns
- Controller/Presenter doesn't contain business logic
- Model is independent

**Microservices / Modular:**
- Services are properly isolated
- No circular dependencies between modules
- Clear API boundaries

---

## Step 6: Code Conventions Analysis

### 6.1 Naming Conventions

| Category | Rule (from standards) | Pattern to check | Example violation |
|----------|----------------------|------------------|-------------------|
| Files | `kebab-case.ts` | `grep -r "\.ts$" \| grep -v kebab` | `UserService.ts` instead of `user-service.ts` |
| Components | `PascalCase.tsx` | `find -name "*.tsx"` | `button.tsx` instead of `Button.tsx` |
| Functions | `camelCase` | AST analysis | `get_user()` instead of `getUser()` |
| Constants | `SCREAMING_SNAKE_CASE` | `grep "const [A-Z]"` | `const apiKey` instead of `const API_KEY` |
| Classes | `PascalCase` | `grep "class "` | `class userService` instead of `class UserService` |

### 6.2 Import Conventions

Check for:
- Import order (external → internal → relative)
- Path aliases usage (`@/` vs `../../../`)
- Extension requirements (`.js` extension in ESM)
- Named vs default exports
- Barrel files usage (`index.ts`)

```bash
# Check import patterns
grep -r "^import" --include="*.ts" --include="*.tsx" | head -50
```

### 6.3 Type Conventions

Verify:
- Strict mode enabled
- No `any` types (or documented exceptions)
- Interface vs Type usage
- Props types for components
- Return types on functions

```bash
# Find any usage
grep -r ": any" --include="*.ts" --include="*.tsx" | grep -v node_modules
grep -r "as any" --include="*.ts" --include="*.tsx" | grep -v node_modules
```

---

## Step 7: Pattern Compliance Analysis

### 7.1 Documented Patterns

For each pattern documented in CLAUDE.md/AGENTS.md, verify implementation:

**Example patterns to check:**
- Error handling pattern
- API call pattern
- State management pattern
- Form handling pattern
- Validation pattern
- Authentication pattern

### 7.2 Anti-patterns Detection

Check for patterns explicitly forbidden in standards:

| Anti-pattern | How to detect |
|--------------|---------------|
| God objects | Files > 500 lines with multiple responsibilities |
| Circular dependencies | Import analysis |
| Magic numbers | Hardcoded values without constants |
| Deep nesting | > 4 levels of indentation |
| Copy-paste code | Repeated blocks > 10 lines |

---

## Step 8: Dependency and Module Analysis

### 8.1 Module Boundaries

Verify modules respect documented boundaries:

```bash
# Check for cross-module imports
grep -r "from '.*modules/auth" --include="*.ts" src/modules/payments/
```

### 8.2 Circular Dependencies

```bash
# Look for potential cycles
grep -r "import.*from" --include="*.ts" | awk -F: '{print $1, $2}' | sort | uniq
```

### 8.3 External Dependencies

Verify dependencies align with standards:
- Approved libraries only
- No deprecated packages
- Consistent versions

---

## Step 9: Documentation Compliance

Check documentation requirements from standards:

- README exists and is updated
- JSDoc/docstrings on public APIs
- Type definitions complete
- Examples provided where required
- Changelog maintained

---

## Step 10: Code Metrics Analysis (Inspired by ArchUnitTS)

Calculate quality metrics to complement standards compliance:

### 10.1 Complexity Metrics

| Metric | Description | Threshold | How to detect |
|--------|-------------|-----------|---------------|
| **Cyclomatic Complexity** | Number of independent paths | < 10 per function | Count if/else, switch, loops, ternaries |
| **Cognitive Complexity** | How hard to understand | < 15 per function | Nested structures, recursion, breaks in flow |
| **Lines per File** | File size | < 300 lines | `wc -l [file]` |
| **Lines per Function** | Function size | < 50 lines | Count lines between function boundaries |
| **Nesting Depth** | Max indentation levels | < 4 levels | Count nested blocks |

### 10.2 Coupling Metrics

| Metric | Description | Threshold |
|--------|-------------|-----------|
| **Afferent Coupling (Ca)** | Incoming dependencies | Varies by module type |
| **Efferent Coupling (Ce)** | Outgoing dependencies | < 10 for most modules |
| **Instability (I)** | Ce / (Ca + Ce) | Domain: 0, Infrastructure: 1 |
| **Fan-in** | How many modules use this | High for utils, low for features |
| **Fan-out** | How many modules this uses | < 7 (Miller's Law) |

### 10.3 Cohesion Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| **LCOM (Lack of Cohesion)** | How related are methods | < 0.5 (closer to 0 is better) |
| **Single Responsibility** | One reason to change | Each class/module has one purpose |

```bash
# Detect potential low cohesion (files with many unrelated exports)
grep -l "export" --include="*.ts" | xargs -I {} sh -c 'echo {}; grep -c "export" {}'

# Count dependencies per file
grep -c "^import" [file]
```

---

## Step 11: Dependency Rule Validation (Inspired by dependency-cruiser)

### 11.1 Layer Dependency Rules

Define and validate layer rules based on documented architecture:

```
┌─────────────────────────────────────────────────────────┐
│ RULES: Who can import whom?                             │
├─────────────────────────────────────────────────────────┤
│ ✅ pages → components, hooks, services, utils, types    │
│ ✅ components → hooks, utils, types                      │
│ ✅ hooks → services, utils, types                        │
│ ✅ services → repositories, utils, types                 │
│ ✅ repositories → models, utils, types                   │
│ ❌ services → components (FORBIDDEN)                     │
│ ❌ repositories → services (FORBIDDEN)                   │
│ ❌ utils → anything except types (FORBIDDEN)             │
│ ❌ types → anything (FORBIDDEN - must be leaf nodes)     │
└─────────────────────────────────────────────────────────┘
```

### 11.2 Module Boundary Rules

For modular/microservices architecture:

```bash
# Detect cross-module violations
# Rule: modules/auth should NOT import from modules/payments
grep -r "from.*modules/payments" --include="*.ts" src/modules/auth/

# Rule: modules should only communicate through public APIs
grep -r "from.*modules/[^/]*/internal" --include="*.ts"
```

### 11.3 Circular Dependency Detection

```bash
# Build import graph and detect cycles
# Look for A → B → C → A patterns
grep -r "^import.*from" --include="*.ts" | \
  awk -F: '{print $1}' | sort | uniq -c | sort -rn | head -20
```

### 11.4 Forbidden Dependency Patterns

Check for explicitly forbidden imports in CLAUDE.md/AGENTS.md:

| Pattern | Reason | Detection |
|---------|--------|-----------|
| Direct DB access from UI | Bypass business layer | `grep "prisma\|mongoose" src/components/` |
| env vars in components | Security risk | `grep "process.env" src/components/` |
| Internal module imports | Breaks encapsulation | `grep "/internal/" --include="*.ts"` |
| Test utils in prod code | Bundle bloat | `grep "test-utils\|@testing" src/` |

---

## Step 12: Over-Engineering Detection (Inspired by Code Quality Pragmatist)

### 12.1 Unnecessary Abstractions

| Pattern | Indicator | Check |
|---------|-----------|-------|
| **Single-use abstractions** | Interface with 1 implementation | Count implementations per interface |
| **Premature generalization** | Generic<T> used once | Find generic types with single usage |
| **Over-configuration** | 20+ config options, 3 used | Review config files complexity |
| **Factory for everything** | Factories creating simple objects | Review factory patterns |
| **Too many layers** | Request passes through 10+ files | Trace a simple CRUD operation |

### 12.2 Unnecessary Complexity

```bash
# Files with too many exports (potential god objects)
find . -name "*.ts" -exec sh -c 'count=$(grep -c "^export" "$1" 2>/dev/null); if [ "$count" -gt 15 ]; then echo "$1: $count exports"; fi' _ {} \;

# Deeply nested callbacks/promises
grep -n "then.*then.*then" --include="*.ts" --include="*.js"

# Excessive type gymnastics
grep -n "extends.*extends.*extends" --include="*.ts"
```

### 12.3 Developer Experience Issues

- Functions with > 5 parameters
- Required setup before simple operations
- Inconsistent patterns for same operations
- Missing error messages or unhelpful errors

---

## Step 13: Generate CLAUDE.md/AGENTS.md (If Missing)

If no standards files exist, offer to generate a starter template:

```markdown
# CLAUDE.md - Project Standards

## Project Overview
[Brief description of what this project does]

## Tech Stack
- Language: [TypeScript/Python/Go/etc.]
- Framework: [React/FastAPI/etc.]
- Database: [PostgreSQL/MongoDB/etc.]

## Architecture
[Pattern used: Clean Architecture / MVC / etc.]

### Folder Structure
```
src/
├── components/     # UI components only
├── pages/          # Route pages
├── hooks/          # Custom React hooks
├── services/       # Business logic
├── repositories/   # Data access
├── utils/          # Pure utility functions
├── types/          # TypeScript types/interfaces
└── config/         # Configuration files
```

## Coding Standards

### Naming Conventions
- Files: `kebab-case.ts`
- Components: `PascalCase.tsx`
- Functions: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE`
- Types/Interfaces: `PascalCase`

### Import Order
1. External dependencies (react, lodash, etc.)
2. Internal absolute imports (@/components, etc.)
3. Relative imports (./utils, ../hooks)
4. Type imports

### Error Handling
[Document your error handling patterns]

### Testing
[Document testing requirements and patterns]

## Forbidden Patterns
- ❌ Never use `any` type
- ❌ No console.log in production code
- ❌ Services cannot import from components
- ❌ No direct database access from UI layer

## Common Commands
```bash
npm run dev      # Start development server
npm run build    # Build for production
npm run test     # Run tests
npm run lint     # Run linter
```
```

---

## Step 14: Generate Report

**Respond directly in the chat:**

```markdown
# BRO STANDARDS COMPLIANCE REPORT

**Date:** [current date]
**Scope:** `[path, description or "entire project"]`
**Language(s):** [detected]
**Files analyzed:** [number]
**Standards files found:** [list: CLAUDE.md, AGENTS.md, etc.]

---

## Executive Summary

| Category | Compliant | Violations | Compliance Rate |
|----------|-----------|------------|-----------------|
| Folder Structure | X/Y | Z | XX% |
| Naming Conventions | X/Y | Z | XX% |
| Import Conventions | X/Y | Z | XX% |
| Type Conventions | X/Y | Z | XX% |
| Architectural Patterns | X/Y | Z | XX% |
| Dependency Rules | X/Y | Z | XX% |
| Code Patterns | X/Y | Z | XX% |
| Code Metrics | X/Y | Z | XX% |
| Documentation | X/Y | Z | XX% |
| **TOTAL** | **X/Y** | **Z** | **XX%** |

**Overall Compliance:** [Excellent (>90%) | Good (75-90%) | Needs Work (50-75%) | Critical (<50%)]

---

## Standards Files Analysis

### CLAUDE.md
- **Found:** Yes/No
- **Rules extracted:** [number]
- **Key rules:**
  1. [Rule summary]
  2. [Rule summary]
  ...

### AGENTS.md
- **Found:** Yes/No
- **Rules extracted:** [number]
- **Key rules:**
  1. [Rule summary]
  2. [Rule summary]
  ...

### Other Standards Files
[List any other standards files found and their key rules]

---

## Detailed Findings

### Category: Folder Structure

#### ✅ Compliant
| Rule | Location | Status |
|------|----------|--------|
| Components in `src/components/` | `src/components/` | Correct |

#### ❌ Violations

##### VIOLATION-001: [Descriptive title]

**Rule violated:** [Exact text from CLAUDE.md/AGENTS.md]
**Severity:** [Critical | High | Medium | Low]

**Location:**
- Path: `src/wrong-location/Component.tsx`

**Problem:**
[Clear description of what's wrong]

**Expected (per standards):**
```
src/components/Component.tsx
```

**Actual:**
```
src/wrong-location/Component.tsx
```

**Remediation:**
[Steps to fix]

---

### Category: Naming Conventions

#### ❌ Violations

##### VIOLATION-002: File naming convention not followed

**Rule violated:** "All TypeScript files should use kebab-case"
**Severity:** Medium

**Files affected:**
| Current Name | Expected Name |
|--------------|---------------|
| `UserService.ts` | `user-service.ts` |
| `APIClient.ts` | `api-client.ts` |

**Remediation:**
Rename files to follow kebab-case convention.

---

### Category: Import Conventions

[Similar structure for each category]

---

### Category: Type Conventions

#### ❌ Violations

##### VIOLATION-003: Usage of `any` type

**Rule violated:** "Never use `any`, use `unknown` for truly unknown types"
**Severity:** High

**Locations:**
| File | Line | Code |
|------|------|------|
| `src/utils/api.ts` | 45 | `function parse(data: any)` |
| `src/hooks/useData.ts` | 12 | `const result = data as any` |

**Remediation:**
```typescript
// Instead of:
function parse(data: any)
// Use:
function parse(data: unknown)
// Or with proper typing:
function parse(data: ApiResponse)
```

---

### Category: Architectural Patterns

#### ❌ Violations

##### VIOLATION-004: Layer boundary violation

**Rule violated:** "Services should not import from components"
**Severity:** Critical

**Location:**
- File: `src/services/userService.ts`
- Line: 5

**Problematic code:**
```typescript
import { UserCard } from '../components/UserCard' // ❌ Wrong!
```

**Impact:**
Breaks separation of concerns and creates circular dependency risk.

**Remediation:**
Services should only import from other services, utils, or types. Never from UI components.

---

### Category: Code Patterns

[Pattern compliance findings]

---

### Category: Documentation

#### ❌ Violations

##### VIOLATION-005: Missing JSDoc on public API

**Rule violated:** "All exported functions must have JSDoc documentation"
**Severity:** Low

**Files missing documentation:**
| File | Function | Status |
|------|----------|--------|
| `src/utils/helpers.ts` | `formatDate()` | No JSDoc |
| `src/services/api.ts` | `fetchData()` | No JSDoc |

---

## Architecture Diagram (Current State)

```
┌─────────────────────────────────────────────────────────┐
│                     PRESENTATION                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │  Components │  │   Pages     │  │   Hooks     │     │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘     │
└─────────┼────────────────┼────────────────┼─────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────┐
│                      BUSINESS                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │  Services   │  │  Use Cases  │  │   Domain    │     │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘     │
└─────────┼────────────────┼────────────────┼─────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────┐
│                        DATA                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │ Repositories│  │    API      │  │   Storage   │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
└─────────────────────────────────────────────────────────┘

[✅ = Compliant | ❌ = Violations found | ⚠️ = Partial compliance]
```

---

## Remediation Priority

### Critical (Immediate)
1. [Architectural violation] — Impact: breaks system integrity
2. [Type safety issue] — Impact: runtime errors

### High (This week)
1. [Naming convention] — Impact: maintainability
2. [Import pattern] — Impact: build issues

### Medium (This month)
1. [Documentation missing] — Impact: onboarding
2. [Minor pattern deviation] — Impact: consistency

### Low (Backlog)
1. [Style preference] — Impact: cosmetic

---

## Recommendations

### If CLAUDE.md/AGENTS.md are missing:
1. Create CLAUDE.md with project-specific rules
2. Document architectural decisions
3. Define naming conventions
4. Establish code patterns

### If standards exist but are incomplete:
1. Add missing rules for: [areas]
2. Clarify ambiguous rules: [rules]
3. Add examples for: [patterns]

---

## Good Practices Found

[Positive patterns that align with standards - recognize what's done well]

- Consistent use of TypeScript strict mode
- Well-organized folder structure
- Clear separation between components and business logic
- ...

---

## Code Metrics Summary

### Complexity

| Metric | Average | Max | Files Exceeding Threshold |
|--------|---------|-----|---------------------------|
| Cyclomatic Complexity | X | Y | [list] |
| Cognitive Complexity | X | Y | [list] |
| Lines per File | X | Y | [list] |
| Nesting Depth | X | Y | [list] |

### Coupling & Cohesion

| Module | Fan-in | Fan-out | Instability | Assessment |
|--------|--------|---------|-------------|------------|
| `src/services/` | X | Y | 0.X | [Stable/Unstable] |
| `src/components/` | X | Y | 0.X | [Stable/Unstable] |

### Potential Over-Engineering

| Issue | Location | Recommendation |
|-------|----------|----------------|
| Single-use abstraction | `src/interfaces/IUserRepo.ts` | Consider inlining |
| Too many layers | User CRUD passes 8 files | Simplify flow |

---

## Dependency Analysis

### Layer Violations

| From | To | Files | Severity |
|------|----|-------|----------|
| services | components | 3 files | Critical |
| repositories | services | 1 file | High |

### Circular Dependencies

```
[If found, show the cycle]
moduleA.ts → moduleB.ts → moduleC.ts → moduleA.ts
```

### Module Boundary Violations

| Module | Violation | Impact |
|--------|-----------|--------|
| `auth` imports from `payments` | Direct coupling | Should use events/API |

---

## Quality Metrics

| Metric | Value | Target (if defined) | Status |
|--------|-------|---------------------|--------|
| Test coverage | XX% | 80% | ✅/❌ |
| Type coverage | XX% | 100% | ✅/❌ |
| Documentation coverage | XX% | 90% | ✅/❌ |
| Lint errors | X | 0 | ✅/❌ |
| Cyclomatic complexity (avg) | X | < 10 | ✅/❌ |
| Files > 300 lines | X | 0 | ✅/❌ |
| Circular dependencies | X | 0 | ✅/❌ |
| Layer violations | X | 0 | ✅/❌ |
```

---

## External Tools Integration (Recommendations)

If available in the project, leverage these tools for deeper analysis:

| Tool | Purpose | Check Command |
|------|---------|---------------|
| **eslint-plugin-boundaries** | Layer/module boundary enforcement | `npx eslint --rule 'boundaries/*'` |
| **dependency-cruiser** | Dependency validation & visualization | `npx depcruise --validate src/` |
| **ArchUnitTS** | Architecture tests for TypeScript | Check `*.arch.test.ts` files |
| **madge** | Circular dependency detection | `npx madge --circular src/` |
| **ts-prune** | Find unused exports | `npx ts-prune` |
| **knip** | Find unused files, deps, exports | `npx knip` |

If these tools are configured, include their output in the report.

---

## References & Inspiration

This command is inspired by and incorporates patterns from:

- **[ArchUnitTS](https://github.com/LukasNiessen/ArchUnitTS)**: Code metrics and architecture testing
- **[dependency-cruiser](https://github.com/sverweij/dependency-cruiser)**: Dependency rule validation
- **[eslint-plugin-boundaries](https://github.com/javierbrea/eslint-plugin-boundaries)**: Module boundary enforcement
- **[ClaudeCodeAgents](https://github.com/darcyegb/ClaudeCodeAgents)**: QA agents patterns
- **[awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code)**: Community best practices
- **AGENTS.md Standard**: OpenAI's universal agent configuration format

---

## Operating Rules

1. **Read-only**: Don't modify any files, only analyze and report
2. **Standards first**: Always read CLAUDE.md/AGENTS.md before analyzing code
3. **Multi-tool awareness**: Check for .cursorrules, copilot-instructions, .continuerules, etc.
4. **Be specific**: Quote exact rules from standards files when reporting violations
5. **Detect the language**: Adapt analysis to the project's language/framework
6. **Interpret intelligently**: Search for files related to what the user requests
7. **Confirm if ambiguous**: If there are multiple matches, ask
8. **Prioritize correctly**: Architectural violations > dependency rules > naming > style
9. **Include remediation**: Each violation must have its solution
10. **Be fair**: If no standards file exists, use industry best practices but note this clearly
11. **Consider context**: Test files may have different rules than production code
12. **Recognize the good**: Mention what's done well, not just violations
13. **Quantify everything**: Provide percentages, metrics, and concrete numbers
14. **Detect over-engineering**: Flag unnecessary abstractions and complexity
15. **Offer to generate**: If CLAUDE.md/AGENTS.md missing, offer starter template
16. **Be actionable**: Remediation steps should be clear and concrete
17. **Suggest tooling**: Recommend external tools that could help enforce standards
18. **Hierarchical standards**: Nested standards files take precedence (per AGENTS.md spec)
