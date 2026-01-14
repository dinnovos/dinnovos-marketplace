---
name: bro-is-this-safe
description: Scans code for security vulnerabilities - generates detailed report without modifying files. Multi-language support. Read-only.
model: opus
allowed-tools: ["Bash(read-only)", "Read", "Grep", "Glob"]
---

# Security Scan

Analyze code for security vulnerabilities. **Read-only, doesn't modify anything. Multi-language support.**

## User Input

The user can specify what to scan in various ways:

**Exact path:**
- `/bro-is-this-safe src/api/`
- `/bro-is-this-safe src/auth/authService.ts`
- `/bro-is-this-safe app/auth/`

**Natural language (illustrative examples):**
- `/bro-is-this-safe scan the <area> module`
- `/bro-is-this-safe check security in <feature>`
- `/bro-is-this-safe analyze vulnerabilities in <topic> services`
- `/bro-is-this-safe look for secrets in <module>`

**No arguments:**
- `/bro-is-this-safe` → scans the entire project

> **Note:** Terms like "authentication", "payments", "API" are just examples. Interpret what the user requests and search for the corresponding files in the project.

---

## Step 1: Interpret the Request

### If it's an exact path:
Use directly.

### If it's natural language:
Search for files matching the description:

```bash
# Explore project structure (includes config files)
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.go" -o -name "*.rs" -o -name "*.php" -o -name "*.rb" -o -name "*.java" -o -name "*.env*" -o -name "*.yml" -o -name "*.yaml" -o -name "Dockerfile*" \) \
  ! -path "*/node_modules/*" ! -path "*/vendor/*" ! -path "*/target/*" ! -path "*/.git/*" ! -path "*/dist/*"

# Search by related name
find . -type f -iname "*<term>*" | grep -v node_modules
find . -type d -iname "*<term>*" | grep -v node_modules

# Search related content
grep -ril "<term>" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.py" --include="*.go" | grep -v node_modules | head -30
```

**Confirm with the user** if you find multiple matches.

**Limit:** Maximum 100 files. If there are more, ask to narrow down or prioritize by risk (auth, api, config first).

---

## Step 2: Project Context

Search and read configuration and standards files:

```bash
# Project standards and guides
cat CLAUDE.md 2>/dev/null
cat AGENTS.md 2>/dev/null
cat .cursor/rules.md 2>/dev/null

# Security configuration
cat .env.example 2>/dev/null
cat .gitignore 2>/dev/null

# Detect stack
cat package.json pyproject.toml go.mod Cargo.toml composer.json 2>/dev/null
cat docker-compose.yml 2>/dev/null
```

Use this information to understand the project's architecture and security configuration.

---

## Step 3: Read and Analyze

```bash
cat [file]
wc -l [file]
```

Read each file and perform the complete security analysis.

---

## Step 4: OWASP Top 10 Analysis

### A01: Broken Access Control
- Endpoints without permission verification
- Direct object access (IDOR)
- Privilege escalation
- Access control bypass

### A02: Cryptographic Failures
- Sensitive data unencrypted
- Weak algorithms (MD5, SHA1, DES)
- Hardcoded keys
- Self-signed certificates in production

### A03: Injection

#### SQL Injection by language:

**JavaScript/TypeScript:**
```javascript
// ❌ Vulnerable
`SELECT * FROM users WHERE id = ${id}`
db.query(`SELECT * FROM users WHERE email = '${email}'`)
// ✅ Safe
db.query('SELECT * FROM users WHERE id = ?', [id])
```

**Python:**
```python
# ❌ Vulnerable
f"SELECT * FROM users WHERE id = {user_id}"
cursor.execute(f"SELECT * FROM users WHERE email = '{email}'")
# ✅ Safe
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
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

#### Code Injection:

| Language | ❌ Avoid |
|----------|----------|
| JS | `eval()`, `Function()`, `setTimeout(string)` |
| Python | `eval()`, `exec()`, `pickle.loads()` |
| PHP | `eval()`, `create_function()`, `preg_replace /e` |
| Ruby | `eval()`, `instance_eval` with input |

### A04: Insecure Design
- Missing rate limiting
- No business validation
- Weak authentication flows

### A05: Security Misconfiguration
- Debug enabled in production
- Missing security headers
- Overly permissive CORS: `Access-Control-Allow-Origin: *`
- Excessive permissions
- Default configurations

### A06: Vulnerable Components
- Dependencies with known CVEs
- Outdated packages
- Abandoned libraries

### A07: Authentication Failures
- Weak passwords allowed
- No brute force protection
- Predictable tokens
- Sessions that don't expire
- Weak JWT secrets

### A08: Data Integrity Failures
- Insecure deserialization
- No integrity verification
- Automatic updates without signing

### A09: Logging Failures
- Sensitive data in logs
- No logging of critical events
- Publicly accessible logs

### A10: SSRF
- User-controlled URLs without validation
- Manipulable internal requests

---

## Step 5: Credentials and Secrets

### Patterns to detect:
```regex
password\s*=\s*["'][^"']+["']
api[_-]?key\s*=\s*["'][^"']+["']
secret\s*=\s*["'][^"']+["']
token\s*=\s*["'][^"']+["']
AWS_ACCESS_KEY
PRIVATE[_-]?KEY
-----BEGIN.*PRIVATE KEY-----
```

### Examples by language:

**JavaScript/TypeScript:**
```javascript
// ❌ Hardcoded
const API_KEY = "sk-1234567890"
const password = "admin123"
```

**Python:**
```python
# ❌ Hardcoded
API_KEY = "sk-1234567890"
DB_PASSWORD = "secret"
```

**Go:**
```go
// ❌ Hardcoded
const apiKey = "sk-1234567890"
var dbPassword = "secret"
```

---

## Step 6: Sensitive Files

Verify that `.gitignore` includes:
- `.env`, `.env.*`
- `*.pem`, `*.key`
- `*credentials*`
- `*.log`

---

## Step 7: Dependencies

```bash
# JavaScript
cat package.json | grep -A 100 '"dependencies"'
# Python
cat requirements.txt pyproject.toml
# Go
cat go.mod
# Rust
cat Cargo.toml
# PHP
cat composer.json
# Ruby
cat Gemfile
```

Identify potentially vulnerable or very outdated dependencies.

---

## Step 8: Generate Report

**Respond directly in the chat:**

```markdown
# BRO SECURITY REPORT

**Date:** [current date]
**Scope:** `[path, description or "entire project"]`
**Language(s):** [detected]
**Files analyzed:** [number]
**Vulnerabilities found:** [number]

---

## Executive Summary

| Severity | Count | Required Action |
|----------|-------|-----------------|
| Critical | X | Immediate (24-48h) |
| High | X | This week |
| Medium | X | This month |
| Low | X | Backlog |
| Info | X | Consider |

**Overall project risk:** [Critical | High | Medium | Low]

---

## Critical Vulnerabilities

### VULN-001: [Descriptive title]

**Category:** [OWASP A0X | Secrets | Injection | etc.]
**Severity:** Critical
**CVSS Score:** [if applicable]

**Location:**
- File: `path/to/file.ts`
- Line(s): XX-XX

**Vulnerable code:**
```[lang]
[problematic snippet]
```

**Description:**
[Explanation of what's wrong and why it's dangerous]

**Impact:**
[What an attacker could do by exploiting this vulnerability]

**Proof of concept:**
```
[How it could be exploited - without being malicious]
```

**Remediation:**
```[lang]
[corrected code]
```

**References:**
- [OWASP - Name](https://owasp.org/...)
- [CWE-XXX](https://cwe.mitre.org/...)

---

[Repeat for each vulnerability, grouped by severity]

---

## High Vulnerabilities

### VULN-002: ...

---

## Medium Vulnerabilities

### VULN-003: ...

---

## Low Vulnerabilities

### VULN-004: ...

---

## Information and Recommendations

### INFO-001: [Recommendation]

**Description:** [Suggestion that would improve security posture]

---

## Dependency Analysis

| Package | Current Version | Vulnerabilities | Action |
|---------|-----------------|-----------------|--------|
| [name] | X.X.X | X known CVEs | Update to X.X.X+ |

---

## Security Checklist

### Authentication
- [ ] Passwords hashed with bcrypt/argon2
- [ ] Rate limiting on login
- [ ] MFA available
- [ ] Tokens with expiration

### Authorization
- [ ] RBAC implemented
- [ ] Verification on each endpoint
- [ ] Principle of least privilege

### Data
- [ ] Sensitive data encrypted
- [ ] PII protected
- [ ] Encrypted backups

### Infrastructure
- [ ] HTTPS enforced
- [ ] Security headers
- [ ] CORS configured correctly

---

## Remediation Plan

### Immediate (24-48h)
1. [Critical vulnerability] — File: X
2. [Critical vulnerability] — File: Y

### This week
1. [High vulnerability] — File: X
2. [High vulnerability] — File: Y

### This month
1. [Medium vulnerability] — File: X

### Backlog
1. [Security improvement]

---

## Good Security Practices Found

[Positive patterns identified: correct use of prepared statements, appropriate hashing, input validation, configured headers, etc.]
```

---

## Operating Rules

1. **Read-only**: Don't modify any files, only analyze and report
2. **Don't execute exploits**: Only identify vulnerabilities, don't exploit them
3. **Detect the language**: Adapt vulnerability patterns to the project's language
4. **Interpret intelligently**: Search for files related to what the user requests
5. **Confirm if ambiguous**: If there are multiple matches, ask
6. **Be exhaustive**: Review all files in scope
7. **Prioritize correctly**: Critical first, always
8. **Include remediation**: Each vulnerability must have its solution with corrected code
9. **Be specific**: Exact files, lines and code
10. **Avoid false positives**: Don't alarm unnecessarily
11. **Consider context**: Development code vs production
12. **Respect project standards**: Use CLAUDE.md/AGENTS.md as reference
13. **Recognize the good**: Mention well-implemented security practices
14. **References**: Include OWASP, CWE when applicable
