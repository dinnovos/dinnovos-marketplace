---
name: bro-inspire-me
description: Generates creative ideas and solutions to technical problems - from conservative to revolutionary approaches. Uses web search for inspiration. Requires description. Multi-language support. Read-only.
model: opus
allowed-tools: ["Bash(read-only)", "Read", "Grep", "Glob", "WebSearch", "WebFetch"]
---

# Inspire Me - Creative Ideas and Problem Solving

Analyze the project and generate 5-10 ideas/solutions ordered from conservative to revolutionary. **Read-only, doesn't modify anything. Uses web search for inspiration. Multi-language support.**

## Two Operating Modes

### Creative Ideas Mode
To explore new features, improvements or directions for the project.

### Problem Solving Mode
To find creative solutions to technical problems the developer is facing.

---

## User Input

The user MUST specify what they need. **The parameter is REQUIRED.**

**Examples - Creative Ideas:**
- `/bro-inspire-me new ways to monetize`
- `/bro-inspire-me ideas to improve onboarding`
- `/bro-inspire-me how to make the dashboard more interactive`
- `/bro-inspire-me features to differentiate from competition`
- `/bro-inspire-me explore AI integrations`

**Examples - Problem Solving:**
- `/bro-inspire-me the build takes too long`
- `/bro-inspire-me I have memory leaks in production`
- `/bro-inspire-me the database gets slow with many records`
- `/bro-inspire-me tests are flaky and fail randomly`
- `/bro-inspire-me how to handle concurrency in this module`
- `/bro-inspire-me legacy code is hard to maintain`
- `/bro-inspire-me need to scale to thousands of simultaneous users`

**If the user does NOT provide a description:**
- Respond: "To inspire you I need to know what you need. Please run the command with a description, for example: `/bro-inspire-me improve onboarding` or `/bro-inspire-me the build takes too long`"

> **Note:** The user describes a creative area OR a technical problem. Without this description, the command CANNOT execute.

---

## Step 0: Detect Operating Mode

Analyze the user's request to determine the mode:

### It's CREATIVE IDEAS MODE if:
- Mentions "ideas", "features", "functionality", "improve", "add"
- Talks about opportunities, growth, differentiation, innovation
- Asks "what could I do", "how could I improve", "what would I add"
- Explores new directions for the product

### It's PROBLEM SOLVING MODE if:
- Describes a current problem: "takes too long", "fails", "doesn't work", "is slow"
- Mentions errors, bugs, memory leaks, performance issues
- Uses words like "problem", "issue", "error", "difficult", "complicated"
- Asks "how to solve", "how to fix", "how to resolve"
- Describes a current technical limitation

**Important:** Adapt all analysis and searches according to the detected mode.

---

## Step 1: Understand the Project

### Detect Stack and Architecture

```bash
# General structure
find . -type d -maxdepth 3 | grep -v node_modules | grep -v vendor | grep -v target | grep -v __pycache__ | sort

# View main files
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

### Read Documentation and Configuration

```bash
# Project configuration
cat package.json pyproject.toml go.mod Cargo.toml composer.json Gemfile pom.xml 2>/dev/null

# Documentation
cat README.md 2>/dev/null
cat CLAUDE.md 2>/dev/null
cat AGENTS.md 2>/dev/null
```

### Identify:
- **Purpose**: What problem does it solve?
- **Target users**: Who uses it?
- **Main features**: What does it currently do?
- **Business model**: How does it generate value? (if applicable)
- **Current state**: MVP, mature product, legacy?

---

## Step 2: Analyze the Exploration Area

Search for files and code related to the area the user specified:

```bash
# Search by filename
find . -type f -iname "*<term>*" | grep -v node_modules | grep -v vendor | head -20

# Search by content
grep -ril "<term>" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" --include="*.py" --include="*.go" --include="*.rs" --include="*.php" --include="*.rb" --include="*.java" --include="*.cs" | grep -v node_modules | head -20
```

Read the found files to understand:
- Current state of that area
- What functionality exists
- What limitations it has
- What technologies it uses
- What improvement opportunities exist

---

## Step 3: Research on the Web

**IMPORTANT:** Use WebSearch to find external information. This is REQUIRED.

---

### If it's CREATIVE IDEAS MODE:

```
# Domain trends
WebSearch: "[app type] innovative features 2024 2025"
WebSearch: "[industry/domain] tech trends"

# Similar successful projects
WebSearch: "best [app type] examples"
WebSearch: "[known competitor] features"
WebSearch: "alternatives to [similar product]"

# Based on user's specific area
WebSearch: "[specific area] best practices 2024"
WebSearch: "innovative [area] solutions"
WebSearch: "[area] UX patterns"
```

---

### If it's PROBLEM SOLVING MODE:

```
# Solutions to the specific problem
WebSearch: "[specific problem] solutions [stack]"
WebSearch: "how to fix [problem] in [technology]"
WebSearch: "[problem] best practices"

# Case studies and experiences
WebSearch: "[problem] case study"
WebSearch: "how [known company] solved [problem]"
WebSearch: "[problem] at scale"

# Tools and techniques
WebSearch: "[stack] [problem] tools"
WebSearch: "[problem] debugging techniques"
WebSearch: "[problem] profiling [technology]"

# Patterns and architectures
WebSearch: "[problem] architecture patterns"
WebSearch: "[problem] design patterns [stack]"
```

---

### If you find something interesting:

Use WebFetch to read the full content of the article or page.

### Record all sources:

Save URL, title and what insight you got from each source to include in the report.

---

## Step 4: Generate Ideas/Solutions

Generate between **5 and 10 ideas/solutions** ordered by creativity level:

### Creativity Scale (applies to both modes)

| Level | Type | Creative Ideas | Problem Solving |
|-------|------|----------------|-----------------|
| ⭐ (1-2) | **Conservative** | Incremental improvements | Direct and proven solution |
| ⭐⭐ (3-4) | **Moderate** | New achievable features | Smart optimization |
| ⭐⭐⭐ (5-6) | **Bold** | Approach changes | Partial system redesign |
| ⭐⭐⭐⭐ (7-8) | **Revolutionary** | New paradigms | Architecture change |
| ⭐⭐⭐⭐⭐ (9-10) | **Visionary** | Disruptive ideas | Total rethinking |

### Criteria for each idea/solution:

1. **Technical viability**: Is it possible with the current stack? What changes does it require?
2. **Impact**: How much does it improve the situation? Does it solve the root problem?
3. **Estimated effort**: Days, weeks, months?
4. **Risk**: What could go wrong? Is it reversible?
5. **Inspiration**: Where did the idea come from? (source if applicable)

---

### Examples - What makes a CREATIVE idea:

**NOT creative**: "Add Google login"
**Creative**: "Passwordless access system using magic links + device biometrics"

**NOT creative**: "Improve the dashboard"
**Creative**: "Self-adapting dashboard based on user role and behavior with draggable widgets"

**NOT creative**: "Add notifications"
**Creative**: "Smart 'nudge' system that predicts when users need to act before it becomes urgent"

---

### Examples - What makes a CREATIVE solution:

**Problem: "The build takes too long"**

**NOT creative**: "Use more RAM"
**Creative (⭐)**: "Implement compilation cache with esbuild/SWC"
**Creative (⭐⭐⭐)**: "Migrate to incremental builds with Turborepo + remote caching"
**Creative (⭐⭐⭐⭐⭐)**: "Micro-frontends architecture where each module compiles independently"

**Problem: "Database is slow with many records"**

**NOT creative**: "Add more indexes"
**Creative (⭐)**: "Analyze query plans and optimize the N slowest queries"
**Creative (⭐⭐⭐)**: "Implement read replicas + connection pooling with PgBouncer"
**Creative (⭐⭐⭐⭐⭐)**: "CQRS with Event Sourcing - completely separate reads/writes"

**Problem: "Memory leaks in production"**

**NOT creative**: "Restart the server every day"
**Creative (⭐)**: "Comparative heap snapshots + identify objects that aren't being released"
**Creative (⭐⭐⭐)**: "Implement circuit breakers + graceful degradation when memory > 80%"
**Creative (⭐⭐⭐⭐⭐)**: "Migrate to serverless architecture where each request is stateless"

---

## Step 5: Generate Report

**Respond directly in the chat using the template according to the mode:**

---

### CREATIVE IDEAS MODE TEMPLATE:

```markdown
# BRO CREATIVE IDEAS REPORT

**Date:** [current date]
**Project:** `[project name]`
**Area explored:** [specified area]
**Stack:** [detected technologies]
**Mode:** Creative Ideas

---

## Project Summary

**Type:** [Web App | API | CLI | Mobile | Library | etc.]
**Purpose:** [In one sentence, what problem it solves]
**Target users:** [Who it serves]

**Main features:**
- [Feature 1]
- [Feature 2]
- [Feature 3]

**Current state of explored area:**
[Brief description of how that area currently is. What exists, what's missing, what limitations it has]

---

## Inspiration Sources Consulted

| Source | Type | Key insight |
|--------|------|-------------|
| [name/URL] | [Article/Product/Trend] | [What we learned from this source] |
| [name/URL] | [Article/Product/Trend] | [What we learned from this source] |

---

## Generated Ideas
```

---

### PROBLEM SOLVING MODE TEMPLATE:

```markdown
# BRO CREATIVE SOLUTIONS REPORT

**Date:** [current date]
**Project:** `[project name]`
**Problem:** [user's problem description]
**Stack:** [detected technologies]
**Mode:** Problem Solving

---

## Problem Analysis

**Reported symptoms:**
[What the user described]

**Technical context:**
- **Stack:** [technologies]
- **Related files:** [found files]
- **Scale:** [users, data, requests, etc. if applicable]

**Initial diagnosis:**
[What could be causing the problem based on code analysis]

**Current impact:**
[How it affects the project/users/development]

---

## Research Conducted

| Source | Type | Key insight |
|--------|------|-------------|
| [name/URL] | [Article/StackOverflow/Docs] | [What we learned] |
| [name/URL] | [Case Study/Tool] | [What we learned] |

---

## Proposed Solutions
```

---

### Ideas/Solutions Content (same for both modes):

### Idea 1: [Catchy and Memorable Title]

**Creativity level:** ⭐ (1-2) — Conservative
**Estimated effort:** [X days/weeks] — [Low/Medium/High]

**The idea:**
[Clear explanation in 2-3 paragraphs. What it is, how it would work from the user's perspective, what problem it solves or what opportunity it leverages]

**Why it's technically viable:**
- [The current stack already has X that facilitates this]
- [There's a library/service Y that solves the complex part]
- [Similar pattern already implemented in module Z]

**Suggested technologies/approaches:**
- **[Technology 1]**: [what it would be used for]
- **[Technology 2]**: [what it would be used for]
- **[Pattern/approach]**: [how to apply it]

**Expected impact:**
[How it would improve user experience, metrics that could improve, business value]

**Inspiration:** [Where the idea came from - web source, competitor, trend, etc.]

---

### Idea 2: [Catchy Title]

**Creativity level:** ⭐⭐ (3-4) — Moderate
**Estimated effort:** [X weeks] — [Medium]

[Same structure...]

---

### Idea 3: [Catchy Title]

**Creativity level:** ⭐⭐⭐ (5-6) — Bold
**Estimated effort:** [X weeks/months] — [Medium/High]

[Same structure...]

---

### Idea 4: [Catchy Title]

**Creativity level:** ⭐⭐⭐⭐ (7-8) — Revolutionary
**Estimated effort:** [X months] — [High]

[Same structure...]

---

### Idea 5: [Catchy Title]

**Creativity level:** ⭐⭐⭐⭐⭐ (9-10) — Visionary
**Estimated effort:** [X months] — [High]

[Same structure...]

---

[Add more ideas if relevant, up to 10 maximum. Make sure to have variety across all creativity levels]

---

## Decision Matrix

| Idea | Creativity | Viability | Impact | Effort | Recommendation |
|------|------------|-----------|--------|--------|----------------|
| [Idea 1] | ⭐ | High | Medium | Low | Quick win |
| [Idea 2] | ⭐⭐ | High | High | Medium | Prioritize |
| [Idea 3] | ⭐⭐⭐ | Medium | High | Medium | Evaluate |
| [Idea 4] | ⭐⭐⭐⭐ | Medium | Very High | High | Plan |
| [Idea 5] | ⭐⭐⭐⭐⭐ | Low | Transformative | Very High | Future vision |

**Legend:**
- Quick win / Prioritize = Implement soon
- Evaluate = Evaluate in more detail
- Plan / Future vision = Keep on radar for the future

---

## Recommendation

**To start today (quick wins):**
1. [Concrete action based on conservative ideas]
2. [Another low effort high impact action]

**To plan this month:**
1. [Action based on moderate/bold ideas]
2. [Research or prototype]

**For long-term vision:**
1. [How revolutionary ideas could evolve the product]
2. [What to validate before investing in visionary ideas]

---

## Considerations

**Risks to evaluate:**
- [Risk 1 of some idea and how to mitigate it]
- [Risk 2]

**Dependencies:**
- [What would be needed to implement the most ambitious ideas]
- [Skills or resources that might be missing]

**Recommended validations:**
- [How to validate ideas before investing significant effort]
- [Metrics or feedback to collect]

---

## Final Reflection

[An inspiring paragraph about the project's potential and how these ideas could transform it. Invite the user to think beyond the obvious and consider what kind of product they want to build]
```

---

## Operating Rules

### General (both modes):

1. **Read-only**: Don't modify any files, only analyze and generate ideas/solutions
2. **Use web search**: Research on the internet — this is REQUIRED
3. **Be genuinely creative**: Proposals should surprise, not be obvious or generic
4. **Order by creativity**: ALWAYS from conservative (⭐) to visionary (⭐⭐⭐⭐⭐)
5. **Justify viability**: Each proposal must be technically possible, explain how
6. **Concrete technologies**: Don't say "optimize", say exactly WHAT and HOW
7. **Estimate effort**: Give a realistic idea of time/resources needed
8. **Consider context**: Proposals must make sense for THIS specific project
9. **Cite your sources**: Mention where the inspiration came from
10. **Balance distribution**: Include proposals at ALL creativity levels

### Creative Ideas Mode:

11. **Think of the end user**: How does this improve their experience?
12. **Avoid the generic**: "Add social login" isn't creative; "Gamified onboarding" is
13. **Challenge the status quo**: Include ideas that rethink the current approach
14. **Only mention emerging tech if relevant**: AI, blockchain, AR/VR only when they add real value

### Problem Solving Mode:

15. **Diagnose first**: Analyze the code to understand the root cause
16. **Offer progressive solutions**: From quick fixes to complete redesigns
17. **Consider trade-offs**: Every solution has pros and cons, mention them
18. **Include specific tools**: Mention concrete libraries, services, commands
19. **Think about prevention**: How to prevent the problem from happening again
20. **Consider production context**: Some solutions require downtime, plan for it
