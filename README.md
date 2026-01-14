# 🤙 Bro Code Tools - Claude Code Plugin Marketplace

Hey bro! Code analysis tools suite for Claude Code.

## ✨ Features

- 🌐 **Polyglot**: Supports JavaScript/TypeScript, Python, Go, Rust, PHP, Ruby, Java, C#
- 🔍 **Auto-detection**: Identifies the language and adapts the analysis
- 🛡️ **Complete analysis**: Code review, security, performance, refactoring
- 💡 **Creative ideas**: Generate ideas and solutions with web search

---

## 📦 Available Commands

| Command | Description |
|---------|-------------|
| `/bro-commit-this` | Create automatic commits with Conventional Commits |
| `/bro-review-before-i-screw-up` | Audit changes before commit (don't screw up!) |
| `/bro-review-this` | Complete code quality review |
| `/bro-is-this-safe` | Security scan (OWASP Top 10) |
| `/bro-this-is-slow` | Detect performance issues |
| `/bro-refactor-this` | Find refactoring opportunities |
| `/bro-explain-this` | Explain project architecture and flows |
| `/bro-inspire-me` | Creative ideas and problem solutions |

---

## 🚀 Installation

### Step 1: Add the Marketplace

```bash
/plugin marketplace add dinnovos/dinnovos-marketplace
```

### Step 2: Install the Plugin

```bash
/plugin install bro-code-tools
```

### Verify Installation

```bash
/plugin list
```

---

## 🎯 Usage

### 📝 `/bro-commit-this`
Create automatic commits with Conventional Commits format.

```bash
/bro-commit-this
```

Analyzes your changes and generates descriptive messages like:
- `feat(auth): add Google OAuth login`
- `fix(api): resolve timeout on large file uploads`
- `refactor(db)!: migrate from MySQL to PostgreSQL`

---

### 🙏 `/bro-review-before-i-screw-up`
Complete audit of pending changes before commit. Don't screw up!

```bash
/bro-review-before-i-screw-up
```

Detects bugs, security issues, performance problems, and quality issues in your staged changes before you commit them.

---

### 🔍 `/bro-review-this`
Complete code quality review.

```bash
# Review a specific folder
/bro-review-this src/services/

# Review a specific file
/bro-review-this src/components/Button.tsx

# Review the entire project
/bro-review-this

# Natural language
/bro-review-this review the authentication module
/bro-review-this analyze the payment services
```

---

### 🔐 `/bro-is-this-safe`
Security scan based on OWASP Top 10.

```bash
# Scan a specific module
/bro-is-this-safe src/api/

# Scan authentication code
/bro-is-this-safe src/auth/

# Scan entire project
/bro-is-this-safe

# Natural language
/bro-is-this-safe check security in user registration
/bro-is-this-safe look for secrets in config files
```

Detects SQL injection, XSS, hardcoded credentials, insecure configurations, and more.

---

### 🐢 `/bro-this-is-slow`
Detect performance problems.

```bash
# Audit a specific service
/bro-this-is-slow src/services/dataService.ts

# Audit entire project
/bro-this-is-slow

# Natural language
/bro-this-is-slow why is the dashboard slow
/bro-this-is-slow find memory leaks in the chat module
/bro-this-is-slow analyze database queries
```

Finds N+1 queries, O(n²) algorithms, memory leaks, missing memoization, heavy imports, and more.

---

### ♻️ `/bro-refactor-this`
Find refactoring opportunities and duplicated code.

```bash
# Analyze components
/bro-refactor-this src/components/

# Analyze entire project
/bro-refactor-this

# Natural language
/bro-refactor-this find duplicates in services
/bro-refactor-this analyze the utils folder
```

Identifies duplicate code, similar logic, repeated functions, and naming inconsistencies.

---

### 🧭 `/bro-explain-this`
Explain project architecture and flows.

```bash
# Explain entire project
/bro-explain-this

# Explain specific module
/bro-explain-this src/modules/payments/

# Natural language
/bro-explain-this how does the authentication work
/bro-explain-this explain the API architecture
/bro-explain-this what does the order service do
```

Perfect for onboarding or understanding new codebases.

---

### 💡 `/bro-inspire-me`
Generate creative ideas or solve technical problems.

```bash
# Creative ideas
/bro-inspire-me new ways to monetize the app
/bro-inspire-me ideas to improve user onboarding
/bro-inspire-me features to differentiate from competition

# Problem solving
/bro-inspire-me the build takes too long
/bro-inspire-me I have memory leaks in production
/bro-inspire-me the database gets slow with many records
/bro-inspire-me tests are flaky and fail randomly
```

Uses web search for inspiration and generates 5-10 ideas ordered from conservative to revolutionary.

---

## 📁 Plugin Structure

```
bro-code-tools/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
└── commands/                     # Slash commands
    ├── bro-commit-this.md
    ├── bro-review-before-i-screw-up.md
    ├── bro-review-this.md
    ├── bro-is-this-safe.md
    ├── bro-this-is-slow.md
    ├── bro-refactor-this.md
    ├── bro-explain-this.md
    └── bro-inspire-me.md
```

---

## 🌐 Supported Languages

| Language | Detection |
|----------|-----------|
| JavaScript/TypeScript | `package.json` |
| Python | `pyproject.toml` |
| Go | `go.mod` |
| Rust | `Cargo.toml` |
| PHP | `composer.json` |
| Ruby | `Gemfile` |
| Java | `pom.xml` |
| C# | `*.csproj` |

---

## 🔄 Update

```bash
/plugin marketplace update dinnovos/dinnovos-marketplace
/plugin update bro-code-tools
```

---

## 🗑️ Uninstall

```bash
/plugin remove bro-code-tools
/plugin marketplace remove dinnovos/dinnovos-marketplace
```

---

## 📄 License

MIT License - Use these tools freely.

---

## 🤝 Contributions

Ideas for new commands? Open an Issue or PR!

---

Created by Dinnovos | Compatible with Claude Code 2.0+
