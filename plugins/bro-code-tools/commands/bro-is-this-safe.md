---
name: bro-is-this-safe
description: Escanea el código en busca de vulnerabilidades de seguridad - genera informe detallado sin modificar archivos. Soporta múltiples lenguajes. Solo lectura.
model: opus
allowed-tools: ["Bash(read-only)", "Read", "Grep", "Glob"]
---

# Escaneo de Seguridad

Analiza el código en busca de vulnerabilidades de seguridad. **Solo lectura, no modifica nada. Soporta múltiples lenguajes.**

## Entrada del Usuario

El usuario puede especificar qué escanear de varias formas:

**Ruta exacta:**
- `/bro-is-this-safe src/api/`
- `/bro-is-this-safe src/auth/authService.ts`
- `/bro-is-this-safe app/auth/`

**Lenguaje natural (ejemplos ilustrativos):**
- `/bro-is-this-safe escanea el módulo de <área>`
- `/bro-is-this-safe revisa seguridad en <funcionalidad>`
- `/bro-is-this-safe analiza vulnerabilidades en los servicios de <tema>`
- `/bro-is-this-safe busca secrets en <módulo>`

**Sin argumentos:**
- `/bro-is-this-safe` → escanea todo el proyecto

> **Nota:** Los términos como "autenticación", "pagos", "API" son solo ejemplos. Interpreta lo que el usuario solicite y busca los archivos correspondientes en el proyecto.

---

## Paso 1: Interpretar la Solicitud

### Si es ruta exacta:
Usar directamente.

### Si es lenguaje natural:
Buscar archivos que coincidan con la descripción:

```bash
# Explorar estructura del proyecto (incluye archivos de config)
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.go" -o -name "*.rs" -o -name "*.php" -o -name "*.rb" -o -name "*.java" -o -name "*.env*" -o -name "*.yml" -o -name "*.yaml" -o -name "Dockerfile*" \) \
  ! -path "*/node_modules/*" ! -path "*/vendor/*" ! -path "*/target/*" ! -path "*/.git/*" ! -path "*/dist/*"

# Buscar por nombre relacionado
find . -type f -iname "*<término>*" | grep -v node_modules
find . -type d -iname "*<término>*" | grep -v node_modules

# Buscar contenido relacionado
grep -ril "<término>" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.py" --include="*.go" | grep -v node_modules | head -30
```

**Confirma con el usuario** si encuentras múltiples coincidencias.

**Límite:** Máximo 100 archivos. Si hay más, pide acotar o prioriza por riesgo (auth, api, config primero).

---

## Paso 2: Contexto del Proyecto

Busca y lee archivos de configuración y estándares:

```bash
# Estándares y guías del proyecto
cat CLAUDE.md 2>/dev/null
cat AGENTS.md 2>/dev/null
cat .cursor/rules.md 2>/dev/null

# Configuración de seguridad
cat .env.example 2>/dev/null
cat .gitignore 2>/dev/null

# Detectar stack
cat package.json pyproject.toml go.mod Cargo.toml composer.json 2>/dev/null
cat docker-compose.yml 2>/dev/null
```

Usa esta información para entender la arquitectura y configuración de seguridad del proyecto.

---

## Paso 3: Leer y Analizar

```bash
cat [archivo]
wc -l [archivo]
```

Lee cada archivo y realiza el análisis de seguridad completo.

---

## Paso 4: Análisis OWASP Top 10

### A01: Broken Access Control
- Endpoints sin verificación de permisos
- Acceso directo a objetos (IDOR)
- Elevación de privilegios
- Bypass de controles de acceso

### A02: Cryptographic Failures
- Datos sensibles sin encriptar
- Algoritmos débiles (MD5, SHA1, DES)
- Keys hardcodeadas
- Certificados autofirmados en producción

### A03: Injection

#### SQL Injection por lenguaje:

**JavaScript/TypeScript:**
```javascript
// ❌ Vulnerable
`SELECT * FROM users WHERE id = ${id}`
db.query(`SELECT * FROM users WHERE email = '${email}'`)
// ✅ Seguro
db.query('SELECT * FROM users WHERE id = ?', [id])
```

**Python:**
```python
# ❌ Vulnerable
f"SELECT * FROM users WHERE id = {user_id}"
cursor.execute(f"SELECT * FROM users WHERE email = '{email}'")
# ✅ Seguro
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

**Go:**
```go
// ❌ Vulnerable
fmt.Sprintf("SELECT * FROM users WHERE id = %s", id)
// ✅ Seguro
db.Query("SELECT * FROM users WHERE id = $1", id)
```

**PHP:**
```php
// ❌ Vulnerable
"SELECT * FROM users WHERE id = " . $id
// ✅ Seguro
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
```

**Ruby:**
```ruby
# ❌ Vulnerable
"SELECT * FROM users WHERE id = #{id}"
# ✅ Seguro
User.where(id: id)
```

#### Command Injection:

| Lenguaje | ❌ Vulnerable | ✅ Seguro |
|----------|--------------|----------|
| JS | `exec(cmd)` | `execFile(cmd, args)` |
| Python | `os.system(f"ping {h}")` | `subprocess.run(['ping', h])` |
| Go | `exec.Command("sh", "-c", input)` | `exec.Command("ping", host)` |
| PHP | `system($cmd)` | `escapeshellarg()` |
| Ruby | `` `#{cmd}` `` | `system('cmd', arg)` |

#### XSS:

| Lenguaje | ❌ Vulnerable |
|----------|--------------|
| JS/React | `dangerouslySetInnerHTML`, `innerHTML` |
| PHP | `echo $input` sin escape |
| Ruby/Rails | `raw()`, `html_safe` mal usado |
| Go/templ | `template.HTML()` con input |

#### Code Injection:

| Lenguaje | ❌ Evitar |
|----------|----------|
| JS | `eval()`, `Function()`, `setTimeout(string)` |
| Python | `eval()`, `exec()`, `pickle.loads()` |
| PHP | `eval()`, `create_function()`, `preg_replace /e` |
| Ruby | `eval()`, `instance_eval` con input |

### A04: Insecure Design
- Falta de rate limiting
- Sin validación de negocio
- Flujos de autenticación débiles

### A05: Security Misconfiguration
- Debug habilitado en producción
- Headers de seguridad faltantes
- CORS demasiado permisivo: `Access-Control-Allow-Origin: *`
- Permisos excesivos
- Configuraciones por defecto

### A06: Vulnerable Components
- Dependencias con CVEs conocidos
- Paquetes desactualizados
- Librerías abandonadas

### A07: Authentication Failures
- Contraseñas débiles permitidas
- Sin protección contra brute force
- Tokens predecibles
- Sesiones que no expiran
- JWT secrets débiles

### A08: Data Integrity Failures
- Deserialización insegura
- Sin verificación de integridad
- Updates automáticos sin firma

### A09: Logging Failures
- Datos sensibles en logs
- Sin logging de eventos críticos
- Logs accesibles públicamente

### A10: SSRF
- URLs controladas por usuario sin validar
- Requests internos manipulables

---

## Paso 5: Credenciales y Secrets

### Patrones a detectar:
```regex
password\s*=\s*["'][^"']+["']
api[_-]?key\s*=\s*["'][^"']+["']
secret\s*=\s*["'][^"']+["']
token\s*=\s*["'][^"']+["']
AWS_ACCESS_KEY
PRIVATE[_-]?KEY
-----BEGIN.*PRIVATE KEY-----
```

### Ejemplos por lenguaje:

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

## Paso 6: Archivos Sensibles

Verificar que `.gitignore` incluye:
- `.env`, `.env.*`
- `*.pem`, `*.key`
- `*credentials*`
- `*.log`

---

## Paso 7: Dependencias

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

Identificar dependencias potencialmente vulnerables o muy desactualizadas.

---

## Paso 8: Generar Informe

**Responde directamente en el chat:**

```markdown
# 🔒 INFORME DE SEGURIDAD BRO

**Fecha:** [fecha actual]
**Alcance:** `[ruta, descripción o "proyecto completo"]`
**Lenguaje(s):** [detectados]
**Archivos analizados:** [número]
**Vulnerabilidades encontradas:** [número]

---

## Resumen Ejecutivo

| Severidad | Cantidad | Acción Requerida |
|-----------|----------|------------------|
| 🔴 Crítica | X | Inmediata (24-48h) |
| 🟠 Alta | X | Esta semana |
| 🟡 Media | X | Este mes |
| 🔵 Baja | X | Backlog |
| ℹ️ Info | X | Considerar |

**Riesgo general del proyecto:** [🔴 Crítico | 🟠 Alto | 🟡 Medio | 🟢 Bajo]

---

## 🔴 Vulnerabilidades Críticas

### VULN-001: [Título descriptivo]

**Categoría:** [OWASP A0X | Secrets | Injection | etc.]
**Severidad:** 🔴 Crítica
**CVSS Score:** [si aplica]

**Ubicación:**
- Archivo: `path/to/file.ts`
- Línea(s): XX-XX

**Código vulnerable:**
```[lang]
[fragmento problemático]
```

**Descripción:**
[Explicación de qué está mal y por qué es peligroso]

**Impacto:**
[Qué podría hacer un atacante explotando esta vulnerabilidad]

**Prueba de concepto:**
```
[Cómo se podría explotar - sin ser malicioso]
```

**Remediación:**
```[lang]
[código corregido]
```

**Referencias:**
- [OWASP - Nombre](https://owasp.org/...)
- [CWE-XXX](https://cwe.mitre.org/...)

---

[Repetir para cada vulnerabilidad, agrupadas por severidad]

---

## 🟠 Vulnerabilidades Altas

### VULN-002: ...

---

## 🟡 Vulnerabilidades Medias

### VULN-003: ...

---

## 🔵 Vulnerabilidades Bajas

### VULN-004: ...

---

## ℹ️ Información y Recomendaciones

### INFO-001: [Recomendación]

**Descripción:** [Sugerencia que mejoraría la postura de seguridad]

---

## 📦 Análisis de Dependencias

| Paquete | Versión Actual | Vulnerabilidades | Acción |
|---------|----------------|------------------|--------|
| [nombre] | X.X.X | X CVEs conocidos | Actualizar a X.X.X+ |

---

## ✅ Checklist de Seguridad

### Autenticación
- [ ] Passwords hasheados con bcrypt/argon2
- [ ] Rate limiting en login
- [ ] MFA disponible
- [ ] Tokens con expiración

### Autorización
- [ ] RBAC implementado
- [ ] Verificación en cada endpoint
- [ ] Principio de menor privilegio

### Datos
- [ ] Datos sensibles encriptados
- [ ] PII protegida
- [ ] Backups encriptados

### Infraestructura
- [ ] HTTPS forzado
- [ ] Headers de seguridad
- [ ] CORS configurado correctamente

---

## 📋 Plan de Remediación

### 🔴 Inmediato (24-48h)
1. [Vulnerabilidad crítica] — Archivo: X
2. [Vulnerabilidad crítica] — Archivo: Y

### 🟠 Esta semana
1. [Vulnerabilidad alta] — Archivo: X
2. [Vulnerabilidad alta] — Archivo: Y

### 🟡 Este mes
1. [Vulnerabilidad media] — Archivo: X

### 🔵 Backlog
1. [Mejora de seguridad]

---

## ✨ Buenas Prácticas de Seguridad Encontradas

[Patrones positivos identificados: uso correcto de prepared statements, hashing apropiado, validación de inputs, headers configurados, etc.]
```

---

## Reglas de Operación

1. **Solo lectura**: No modificar ningún archivo, solo analizar y reportar
2. **No ejecutes exploits**: Solo identifica vulnerabilidades, no las explotes
3. **Detecta el lenguaje**: Adapta patrones de vulnerabilidad al lenguaje del proyecto
4. **Interpreta inteligentemente**: Buscar archivos relacionados con lo que pida el usuario
5. **Confirma si hay ambigüedad**: Si hay múltiples coincidencias, pregunta
6. **Sé exhaustivo**: Revisa todos los archivos del alcance
7. **Prioriza correctamente**: Críticas primero, siempre
8. **Incluye remediación**: Cada vulnerabilidad debe tener su solución con código corregido
9. **Sé específico**: Archivos, líneas y código exacto
10. **Evita falsos positivos**: No alarmes innecesariamente
11. **Considera el contexto**: Código de desarrollo vs producción
12. **Respeta estándares del proyecto**: Usa CLAUDE.md/AGENTS.md como referencia
13. **Reconoce lo bueno**: Menciona prácticas de seguridad bien implementadas
14. **Referencias**: Incluye OWASP, CWE cuando aplique
