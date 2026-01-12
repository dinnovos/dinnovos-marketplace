# 🛠️ Dinnovos Code Tools - Claude Code Plugin Marketplace

Suite completa de herramientas de análisis de código para Claude Code. Incluye **commands**, **agents**, y **skills**.

## ✨ Características

- 🌐 **Políglota**: Soporta JavaScript/TypeScript, Python, Go, Rust, PHP, Ruby, Java, C#
- 🔍 **Detección automática**: Identifica el lenguaje y adapta el análisis
- 🤖 **Agentes especializados**: Arquitectura, seguridad, testing
- 📚 **Skills integradas**: Se activan automáticamente cuando son relevantes

---

## 📦 Contenido del Plugin

### Slash Commands (`/comando`)

| Comando | Descripción |
|---------|-------------|
| `/dinnovos-pre-commit-review` | Auditoría de cambios antes del commit |
| `/dinnovos-code-review` | Revisión completa de calidad de código |
| `/dinnovos-security-scan` | Escaneo de seguridad (OWASP Top 10) |
| `/dinnovos-performance-audit` | Detectar problemas de rendimiento |
| `/dinnovos-refactor-analysis` | Encontrar oportunidades de refactorización |
| `/dinnovos-codebase-explain` | Explicar arquitectura y flujos del proyecto |

### Agents (`@agente`)

| Agente | Descripción |
|--------|-------------|
| `@code-architect` | Experto en arquitectura, patrones de diseño, SOLID, Clean Architecture |
| `@security-expert` | Especialista en AppSec, OWASP, vulnerabilidades |
| `@test-specialist` | Experto en testing, TDD, cobertura, estrategias de QA |

### Skills (Auto-invocadas)

| Skill | Se activa cuando... |
|-------|---------------------|
| `code-standards` | Claude detecta necesidad de aplicar estándares de código |
| `git-workflow` | Se trabaja con Git, commits, branches, o conflictos |

---

## 🚀 Instalación

### Paso 1: Agregar el Marketplace

```bash
/plugin marketplace add TU_USUARIO/dinnovos-marketplace
```

### Paso 2: Instalar el Plugin

```bash
/plugin install dinnovos-code-tools
```

### Verificar Instalación

```bash
/plugin list
```

---

## 🎯 Uso

### Commands (Invocación Manual)

```bash
# Revisar cambios antes de commit
/dinnovos-pre-commit-review

# Revisar código específico
/dinnovos-code-review src/services/

# Escanear seguridad
/dinnovos-security-scan src/api/

# Auditar rendimiento
/dinnovos-performance-audit

# Buscar oportunidades de refactor
/dinnovos-refactor-analysis

# Explicar arquitectura
/dinnovos-codebase-explain
```

### Agents (Invocación con @)

```bash
# Pedir ayuda al arquitecto
@code-architect analiza la arquitectura de src/

# Consultar al experto en seguridad
@security-expert revisa este endpoint de autenticación

# Pedir al especialista de testing
@test-specialist genera tests para src/services/userService.ts
```

### Skills (Automáticas)

Las skills se activan automáticamente. Por ejemplo:
- Al escribir commits, `git-workflow` sugiere formato Conventional Commits
- Al escribir código, `code-standards` aplica convenciones del lenguaje

---

## 📁 Estructura del Plugin

```
dinnovos-code-tools/
├── .claude-plugin/
│   └── plugin.json           # Manifest del plugin
├── commands/                  # Slash commands
│   ├── dinnovos-pre-commit-review.md
│   ├── dinnovos-code-review.md
│   ├── dinnovos-refactor-analysis.md
│   ├── dinnovos-security-scan.md
│   ├── dinnovos-codebase-explain.md
│   └── dinnovos-performance-audit.md
├── agents/                    # Subagentes
│   ├── code-architect.md
│   ├── security-expert.md
│   └── test-specialist.md
└── skills/                    # Skills auto-invocadas
    ├── code-standards/
    │   └── SKILL.md
    └── git-workflow/
        └── SKILL.md
```

---

## 🌐 Lenguajes Soportados

| Lenguaje | Detección | Commands | Agents | Skills |
|----------|-----------|----------|--------|--------|
| JavaScript/TypeScript | `package.json` | ✅ | ✅ | ✅ |
| Python | `pyproject.toml` | ✅ | ✅ | ✅ |
| Go | `go.mod` | ✅ | ✅ | ✅ |
| Rust | `Cargo.toml` | ✅ | ✅ | ✅ |
| PHP | `composer.json` | ✅ | ✅ | ✅ |
| Ruby | `Gemfile` | ✅ | ✅ | ✅ |
| Java | `pom.xml` | ✅ | ✅ | ✅ |
| C# | `*.csproj` | ✅ | ✅ | ✅ |

---

## 🔄 Actualización

```bash
/plugin marketplace update TU_USUARIO/dinnovos-marketplace
/plugin update dinnovos-code-tools
```

---

## 🗑️ Desinstalación

```bash
/plugin remove dinnovos-code-tools
/plugin marketplace remove TU_USUARIO/dinnovos-marketplace
```

---

## 📋 Diferencias: Commands vs Agents vs Skills

| Aspecto | Commands | Agents | Skills |
|---------|----------|--------|--------|
| Invocación | Manual (`/comando`) | Manual (`@agente`) | Automática |
| Propósito | Tarea específica | Expertise especializado | Conocimiento contextual |
| Contexto | Limitado a la tarea | Contexto propio | Contexto principal |
| Ejemplo | `/security-scan` | `@security-expert` | Convenciones de código |

---

## 📄 Licencia

MIT License - Usa estas herramientas libremente.

---

## 🤝 Contribuciones

¿Ideas para nuevos commands, agents, o skills? ¡Abre un Issue o PR!

---

Creado por Dinnovos | Compatible con Claude Code 2.0+
