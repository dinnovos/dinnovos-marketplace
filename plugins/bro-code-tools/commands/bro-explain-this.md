---
name: bro-explain-this
description: Explains the architecture, structure and flows of a project or module. Ideal for onboarding or understanding new code. Multi-language support. Read-only.
model: opus
allowed-tools: ["Bash(read-only)", "Read", "Grep", "Glob"]
---

# Codebase Explanation

Analyze and explain the architecture, structure and flows of a project or module. **Read-only, doesn't modify anything. Multi-language support.**

## User Input

The user can specify what to explain in various ways:

**Exact path:**
- `/bro-explain-this src/`
- `/bro-explain-this src/modules/payments/`
- `/bro-explain-this app/`

**Natural language (illustrative examples):**
- `/bro-explain-this explain the project`
- `/bro-explain-this how does the <area> module work`
- `/bro-explain-this explain the architecture of <feature>`
- `/bro-explain-this what does the <topic> service do`
- `/bro-explain-this how do the <module> components connect`

**No arguments:**
- `/bro-explain-this` → explains the entire project

> **Note:** Terms like "authentication", "payments", "users" are just examples. Interpret what the user requests and search for the corresponding files in the project.

---

## Step 1: Detect Language and Stack

```bash
# General structure
find . -type d -maxdepth 3 | grep -v node_modules | grep -v vendor | grep -v target | grep -v __pycache__ | sort

# Detect stack
ls -la
```

### Detection by files:

| File | Stack |
|------|-------|
| `package.json` | Node.js / JavaScript / TypeScript |
| `tsconfig.json` | TypeScript |
| `pyproject.toml` / `requirements.txt` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `composer.json` | PHP |
| `Gemfile` | Ruby |
| `pom.xml` / `build.gradle` | Java |
| `*.csproj` | C# / .NET |

### Common frameworks:

| Language | Indicator Files | Framework |
|----------|-----------------|-----------|
| JS/TS | `next.config.js` | Next.js |
| JS/TS | `vite.config.ts` | Vite |
| JS/TS | `angular.json` | Angular |
| Python | `manage.py` | Django |
| Python | `app.py` + Flask imports | Flask |
| Python | `main.py` + FastAPI imports | FastAPI |
| Go | `go.mod` + chi/gin/echo | Chi/Gin/Echo |
| Rust | `Cargo.toml` + actix/axum | Actix/Axum |
| PHP | `artisan` | Laravel |
| Ruby | `config.ru` + Rails | Rails |

---

## Step 2: Read Documentation

```bash
cat README.md 2>/dev/null
cat CLAUDE.md 2>/dev/null
cat AGENTS.md 2>/dev/null
cat ARCHITECTURE.md 2>/dev/null
cat docs/*.md 2>/dev/null
```

---

## Step 3: Identify Entry Points

### By language/framework:

| Stack | Entry Points |
|-------|--------------|
| **Node.js** | `src/index.ts`, `src/main.ts`, `src/app.ts` |
| **React** | `src/App.tsx`, `src/main.tsx`, `pages/_app.tsx` |
| **Next.js** | `pages/`, `app/`, `next.config.js` |
| **Python** | `main.py`, `app.py`, `__main__.py` |
| **Django** | `manage.py`, `urls.py`, `settings.py` |
| **FastAPI** | `main.py`, `app/main.py` |
| **Go** | `main.go`, `cmd/*/main.go` |
| **Rust** | `src/main.rs`, `src/lib.rs` |
| **PHP/Laravel** | `public/index.php`, `routes/web.php` |
| **Ruby/Rails** | `config.ru`, `config/routes.rb` |
| **Java/Spring** | `*Application.java`, `pom.xml` |
| **C#/.NET** | `Program.cs`, `Startup.cs` |

```bash
# Find entry points
cat src/index.ts src/main.ts src/app.ts 2>/dev/null          # Node
cat src/App.tsx pages/_app.tsx app/layout.tsx 2>/dev/null    # React/Next
cat main.py app.py manage.py 2>/dev/null                      # Python
cat cmd/*/main.go main.go 2>/dev/null                         # Go
cat src/main.rs src/lib.rs 2>/dev/null                        # Rust
```

---

## Step 4: Map Structure

### Common patterns by stack:

#### JavaScript/TypeScript (Node/React):
```
src/
├── components/     # UI components
├── pages/          # Routes/Pages
├── hooks/          # Custom hooks
├── services/       # API/Business logic
├── utils/          # Helpers
├── types/          # TypeScript types
└── lib/            # Shared libraries
```

#### Python (Django/FastAPI):
```
app/
├── api/            # Endpoints
├── models/         # Database models
├── schemas/        # Pydantic schemas
├── services/       # Business logic
├── repositories/   # Data access
└── utils/          # Helpers

# Django specific
project/
├── apps/
│   └── myapp/
│       ├── models.py
│       ├── views.py
│       ├── urls.py
│       └── serializers.py
└── settings.py
```

#### Go:
```
cmd/
├── api/            # Entry points
└── worker/
internal/
├── handlers/       # HTTP handlers
├── services/       # Business logic
├── repositories/   # Data access
├── models/         # Domain models
└── pkg/            # Shared packages
pkg/                # Public packages
```

#### Rust:
```
src/
├── main.rs         # Entry point
├── lib.rs          # Library root
├── handlers/       # Request handlers
├── services/       # Business logic
├── models/         # Domain types
└── db/             # Database layer
```

#### PHP (Laravel):
```
app/
├── Http/
│   ├── Controllers/
│   └── Middleware/
├── Models/
├── Services/
└── Repositories/
routes/
├── web.php
└── api.php
```

#### Ruby (Rails):
```
app/
├── controllers/
├── models/
├── views/
├── services/
└── jobs/
config/
└── routes.rb
```

---

## Step 5: Identify Architectural Patterns

| Pattern | Indicators |
|---------|------------|
| **MVC** | controllers/, models/, views/ |
| **Clean Architecture** | domain/, usecases/, infrastructure/ |
| **Hexagonal** | ports/, adapters/, core/ |
| **DDD** | domain/, application/, infrastructure/ |
| **Microservices** | services/, multiple go.mod/package.json |
| **Monorepo** | packages/, apps/, libs/ |

---

## Step 6: Map Data Flows

### Identify:
- Routes/Endpoints
- Controllers/Handlers
- Services/Use Cases
- Repositories/DAOs
- Models/Entities
- External APIs

---

## Step 7: Generate Report

**Respond directly in the chat:**

```markdown
# BRO CODEBASE EXPLANATION

**Date:** [current date]
**Scope:** `[path, description or "entire project"]`
**Files analyzed:** [number]
**Lines of code:** ~[estimated number]

---

## Executive Summary

**Project type:** [Web App | API | CLI | Library | Monorepo | etc.]
**Tech stack:** [TypeScript + React + Node | Python + FastAPI | Go + Chi | etc.]
**Architectural pattern:** [MVC | Clean Architecture | Hexagonal | etc.]
**Overall state:** [Well structured | Needs refactor | Legacy | etc.]

### In one sentence:
[Brief description of what the project does and how it's organized]

---

## General Architecture

### Folder Structure
```
project/
├── src/
│   ├── [folder]/     # [Purpose]
│   ├── [folder]/     # [Purpose]
│   └── [folder]/     # [Purpose]
├── [file]            # [Purpose]
└── [file]            # [Purpose]
```

### Architectural Pattern
[Explanation of the pattern used and how it's implemented]

### Identified Layers
| Layer | Location | Responsibility |
|-------|----------|----------------|
| [Presentation] | `src/components/` | [description] |
| [Business] | `src/services/` | [description] |
| [Data] | `src/repositories/` | [description] |

---

## Entry Points

### Main Entry Point
- **File:** `[path]`
- **Function:** [what it does]

### Initialization
```[lang]
// Summarized initialization flow
[key snippet]
```

---

## Main Flows

### Flow 1: [Flow name - e.g.: User Authentication]

```
[Request] → [Handler] → [Service] → [Repository] → [DB]
                ↓
           [Response]
```

**Files involved:**
1. `src/[file1].ts` → [what it does]
2. `src/[file2].ts` → [what it does]
3. `src/[file3].ts` → [what it does]

**Description:**
[Step by step explanation of the flow]

---

### Flow 2: [Flow name]
[Repeat structure]

---

## Main Modules

| Module | Location | Responsibility | Depends on |
|--------|----------|----------------|------------|
| [Auth] | `src/auth/` | [description] | [Database, Utils] |
| [Users] | `src/users/` | [description] | [Auth, Database] |

### Dependency Diagram
```
┌─────────┐     ┌─────────┐
│  Auth   │────▶│  Users  │
└────┬────┘     └────┬────┘
     │               │
     ▼               ▼
┌─────────────────────────┐
│       Database          │
└─────────────────────────┘
```

---

## Data Layer

**Database:** [PostgreSQL | MongoDB | etc.]
**ORM/Query Builder:** [Prisma | TypeORM | Mongoose | GORM | SQLAlchemy | etc.]

### Main Models
| Model | File | Key Fields |
|-------|------|------------|
| [User] | `src/models/user.ts` | id, email, password, role |
| [Order] | `src/models/order.ts` | id, userId, items, total |

### Relationships
```
User (1) ←──────→ (N) Order
Order (1) ←──────→ (N) OrderItem
```

---

## Presentation Layer (if applicable)

**Framework:** [React | Vue | Angular | etc.]
**State management:** [Redux | Zustand | Context | etc.]
**Routing:** [React Router | Next.js | etc.]

### Main Components
| Component | Location | Purpose |
|-----------|----------|---------|
| [Layout] | `src/components/Layout/` | [description] |
| [Dashboard] | `src/pages/Dashboard/` | [description] |

---

## External Integrations

| Service | Purpose | Files |
|---------|---------|-------|
| [Stripe] | Payments | `src/services/stripe.ts` |
| [SendGrid] | Emails | `src/services/email.ts` |

---

## Configuration

### Environment Variables
| Variable | Purpose | Required |
|----------|---------|----------|
| `DATABASE_URL` | DB connection | Yes |
| `JWT_SECRET` | Token signing | Yes |
| `API_KEY` | External API | No |

### Configuration Files
| File | Purpose |
|------|---------|
| `tsconfig.json` | TypeScript configuration |
| `.env.example` | Variables template |

---

## How to Navigate the Code

### To understand [feature X]:
1. Start at `src/[file]`
2. Follow to `src/[file2]`
3. Main logic is in `src/[file3]`

### To add [new feature]:
1. Create model in `src/models/`
2. Create service in `src/services/`
3. Add route in `src/routes/`
4. Connect in `src/index.ts`

---

## Points of Attention

### High Complexity
- `src/[file].ts` — [reason for complexity]

### Visible Technical Debt
- [Description of identified technical debt]

### Undocumented Areas
- [Modules or functions that need documentation]

---

## Good Practices Identified

[Positive patterns found: clear separation, consistent naming, well-organized tests, etc.]

---

## Additional Resources

- README.md: [what it contains]
- CONTRIBUTING.md: [if exists]
- /docs: [if exists]
```

---

## Operating Rules

1. **Read-only**: Don't modify any files, only analyze and explain
2. **Detect the stack**: Adapt explanation to the project's language/framework
3. **Interpret intelligently**: Search for files related to what the user requests
4. **Confirm if ambiguous**: If it's unclear what to explain, ask
5. **Be didactic**: Explain for someone who doesn't know the project
6. **Use diagrams**: ASCII art for flows and relationships when helpful
7. **Prioritize what's important**: Critical flows first, details later
8. **Be specific**: Mention specific files and lines
9. **Respect project standards**: Use CLAUDE.md/AGENTS.md as reference
10. **Identify patterns**: Name the architectural pattern if recognized
11. **Flag complexity**: Indicate areas that are hard to understand
12. **Suggest navigation**: Guide on where to start reading
13. **Recognize the good**: Mention positive practices found
