# 🤙 Bro Code Tools - Claude Code Plugin Marketplace

Hey bro! Suite de herramientas de análisis de código para Claude Code.

## ✨ Características

- 🌐 **Políglota**: Soporta JavaScript/TypeScript, Python, Go, Rust, PHP, Ruby, Java, C#
- 🔍 **Detección automática**: Identifica el lenguaje y adapta el análisis
- 🛡️ **Análisis completo**: Code review, seguridad, rendimiento, refactoring
- 💡 **Ideas creativas**: Genera ideas y soluciones con búsqueda web

---

## 📦 Comandos Disponibles

| Comando | Descripción |
|---------|-------------|
| `/bro-commit-this` | Crea commits automáticos con Conventional Commits |
| `/bro-review-before-i-screw-up` | Auditoría de cambios antes del commit (no la cagues!) |
| `/bro-review-this` | Revisión completa de calidad de código |
| `/bro-is-this-safe` | Escaneo de seguridad (OWASP Top 10) |
| `/bro-this-is-slow` | Detectar problemas de rendimiento |
| `/bro-refactor-this` | Encontrar oportunidades de refactorización |
| `/bro-explain-this` | Explicar arquitectura y flujos del proyecto |
| `/bro-inspire-me` | Ideas creativas y soluciones a problemas |

---

## 🚀 Instalación

### Paso 1: Agregar el Marketplace

```bash
/plugin marketplace add dinnovos/dinnovos-marketplace
```

### Paso 2: Instalar el Plugin

```bash
/plugin install bro-code-tools
```

### Verificar Instalación

```bash
/plugin list
```

---

## 🎯 Uso

```bash
# Crear commit automático con mensaje descriptivo
/bro-commit-this

# Revisar cambios antes de commit (no la cagues!)
/bro-review-before-i-screw-up

# Revisar código específico
/bro-review-this src/services/

# Escanear seguridad
/bro-is-this-safe src/api/

# Auditar rendimiento
/bro-this-is-slow

# Buscar oportunidades de refactor
/bro-refactor-this

# Explicar arquitectura
/bro-explain-this

# Generar ideas creativas o resolver problemas
/bro-inspire-me mejorar el onboarding
/bro-inspire-me el build tarda demasiado
```

---

## 📁 Estructura del Plugin

```
bro-code-tools/
├── .claude-plugin/
│   └── plugin.json              # Manifest del plugin
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
/plugin update bro-code-tools
```

---

## 🗑️ Desinstalación

```bash
/plugin remove bro-code-tools
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
