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

```bash
# Create automatic commit with descriptive message
/bro-commit-this

# Review changes before commit (don't screw up!)
/bro-review-before-i-screw-up

# Review specific code
/bro-review-this src/services/

# Security scan
/bro-is-this-safe src/api/

# Performance audit
/bro-this-is-slow

# Find refactoring opportunities
/bro-refactor-this

# Explain architecture
/bro-explain-this

# Generate creative ideas or solve problems
/bro-inspire-me improve the onboarding
/bro-inspire-me the build takes too long
```

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
