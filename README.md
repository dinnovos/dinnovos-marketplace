# 🛠️ Dinnovos Code Tools - Claude Code Plugin Marketplace

Suite completa de herramientas de análisis de código para Claude Code.

## ✨ Características

- 🌐 **Políglota**: Soporta JavaScript/TypeScript, Python, Go, Rust, PHP, Ruby, Java, C#
- 🔍 **Detección automática**: Identifica el lenguaje y adapta el análisis
- 🛡️ **Análisis completo**: Code review, seguridad, rendimiento, refactoring

---

## 📦 Contenido del Plugin

### Slash Commands (`/comando`)

| Comando | Descripción |
|---------|-------------|
| `/dinnovos-auto-commit` | Crea commits automáticos con Conventional Commits |
| `/dinnovos-pre-commit-review` | Auditoría de cambios antes del commit |
| `/dinnovos-code-review` | Revisión completa de calidad de código |
| `/dinnovos-security-scan` | Escaneo de seguridad (OWASP Top 10) |
| `/dinnovos-performance-audit` | Detectar problemas de rendimiento |
| `/dinnovos-refactor-analysis` | Encontrar oportunidades de refactorización |
| `/dinnovos-codebase-explain` | Explicar arquitectura y flujos del proyecto |

---

## 🚀 Instalación

### Paso 1: Agregar el Marketplace

```bash
/plugin marketplace add dinnovos/dinnovos-marketplace
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

```bash
# Crear commit automático con mensaje descriptivo
/dinnovos-auto-commit

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

---

## 📁 Estructura del Plugin

```
dinnovos-code-tools/
├── .claude-plugin/
│   └── plugin.json           # Manifest del plugin
└── commands/                  # Slash commands
    ├── dinnovos-auto-commit.md
    ├── dinnovos-pre-commit-review.md
    ├── dinnovos-code-review.md
    ├── dinnovos-refactor-analysis.md
    ├── dinnovos-security-scan.md
    ├── dinnovos-codebase-explain.md
    └── dinnovos-performance-audit.md
```

---

## 🌐 Lenguajes Soportados

| Lenguaje | Detección |
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

## 🔄 Actualización

```bash
/plugin marketplace update dinnovos/dinnovos-marketplace
/plugin update dinnovos-code-tools
```

---

## 🗑️ Desinstalación

```bash
/plugin remove dinnovos-code-tools
/plugin marketplace remove dinnovos/dinnovos-marketplace
```

---

## 📄 Licencia

MIT License - Usa estas herramientas libremente.

---

## 🤝 Contribuciones

¿Ideas para nuevos commands? ¡Abre un Issue o PR!

---

Creado por Dinnovos | Compatible con Claude Code 2.0+
