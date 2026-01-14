---
name: bro-review-this
description: Auditoría de código - acepta rutas o descripciones naturales. Soporta múltiples lenguajes. Solo lectura.
model: opus
allowed-tools: ["Bash(read-only)", "Read", "Grep", "Glob"]
---

# Revisión de Código

Analiza el código especificado y genera un informe detallado. **Solo lectura, no modifica nada. Soporta múltiples lenguajes.**

## Entrada del Usuario

El usuario puede especificar qué revisar de varias formas:

**Ruta exacta:**
- `/bro-review-this src/components/Button.tsx`
- `/bro-review-this src/hooks/`
- `/bro-review-this internal/handlers/`

**Lenguaje natural (ejemplos ilustrativos):**
- `/bro-review-this revisa el componente <nombre>`
- `/bro-review-this revisa los servicios de <funcionalidad>`
- `/bro-review-this analiza el módulo de <feature>`
- `/bro-review-this revisa todo lo relacionado con <tema>`

**Sin argumentos:**
- `/bro-review-this` → analiza todo el proyecto

> **Nota:** Los términos como "login", "usuarios", "pagos" son solo ejemplos. Interpreta lo que el usuario solicite y busca los archivos correspondientes en el proyecto.

---

## Paso 1: Interpretar la Solicitud

### Si es ruta exacta:
Usar directamente.

### Si es lenguaje natural:
Buscar archivos que coincidan con la descripción:

```bash
# Explorar estructura del proyecto (todas las extensiones)
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.go" -o -name "*.rs" -o -name "*.php" -o -name "*.rb" -o -name "*.java" -o -name "*.cs" -o -name "*.kt" \) | grep -v node_modules | grep -v vendor | grep -v target | grep -v __pycache__ | grep -v dist | grep -v .git

# Buscar por nombre relacionado (reemplaza <término> con lo que pidió el usuario)
find . -type f -iname "*<término>*" | grep -v node_modules
find . -type d -iname "*<término>*" | grep -v node_modules

# Buscar contenido relacionado
grep -ril "<término>" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.py" --include="*.go" --include="*.rs" --include="*.php" | grep -v node_modules | head -20
```

**Confirma con el usuario** si encuentras múltiples coincidencias:
```
Encontré estos archivos relacionados con "<término>":
1. src/components/[Archivo1].tsx
2. src/hooks/[Archivo2].ts
3. src/services/[Archivo3].ts

¿Reviso todos o alguno específico?
```

Si solo hay una coincidencia clara, procede directamente.

---

## Paso 2: Contexto del Proyecto

Busca y lee archivos de configuración y estándares:

```bash
# Estándares y guías del proyecto
cat CLAUDE.md 2>/dev/null
cat AGENTS.md 2>/dev/null
cat .cursor/rules.md 2>/dev/null

# Detectar stack y configuración
cat package.json pyproject.toml go.mod Cargo.toml composer.json Gemfile pom.xml 2>/dev/null

# Configuración de linting y tipos
cat .eslintrc* 2>/dev/null
cat tsconfig.json 2>/dev/null
cat biome.json 2>/dev/null
cat pyproject.toml 2>/dev/null
```

Usa esta información para evaluar el código según los estándares específicos del proyecto.

**Límite:** Máximo 50 archivos. Si hay más, pide acotar.

---

## Paso 3: Leer y Analizar

```bash
cat [archivo]
wc -l [archivo]
```

---

## Paso 4: Análisis por Categorías

Revisa cada archivo buscando problemas en estas categorías, ordenadas por prioridad:

---

### 🔴 P0 - CRÍTICO: Bugs y Errores Lógicos

Problemas que causarán fallos en producción:

- **Variables no inicializadas** o acceso a propiedades de `null`/`undefined`
- **Condiciones imposibles** o lógica invertida
- **Errores off-by-one** en iteraciones y límites
- **Race conditions** en código asíncrono
- **Excepciones no capturadas** que pueden crashear la aplicación
- **Memory leaks** o recursos no liberados (conexiones, listeners, timers)
- **Tipos incorrectos** que pasarán en runtime pero fallarán
- **Estados inconsistentes** que corrompen datos

#### Ejemplos por lenguaje:

| Problema | JS/TS | Python | Go | Rust | PHP | Ruby |
|----------|-------|--------|-----|------|-----|------|
| Null access | `?.` faltante | `None` check | `nil` check | `Option` | `??` | `&.` |
| Error handling | `.catch()` | `try/except` | `if err != nil` | `Result` | `try/catch` | `rescue` |
| Race conditions | ✓ | ✓ | goroutines | threads | - | threads |

**JavaScript/TypeScript:**
```javascript
// ❌ Null sin verificar
user.profile.name
// ✅ Con optional chaining
user?.profile?.name
```

**Python:**
```python
# ❌ None sin verificar
user.profile.name
# ✅ Con verificación
user.profile.name if user and user.profile else None
```

**Go:**
```go
// ❌ Error ignorado
result, _ := getData()
// ✅ Error manejado
result, err := getData()
if err != nil { return err }
```

**Rust:**
```rust
// ❌ Unwrap en producción
let value = result.unwrap()
// ✅ Con manejo
let value = result?
```

---

### 🔴 P0 - CRÍTICO: Seguridad

Vulnerabilidades que exponen el sistema:

- **Credenciales hardcodeadas**: API keys, passwords, tokens, secrets
- **Inyección**: SQL injection, command injection, XSS
- **Inputs no validados**: datos de usuario usados sin sanitizar
- **Exposición de datos sensibles**: logs con información privada
- **Configuraciones inseguras**: CORS permisivo, HTTPS deshabilitado
- **Dependencias vulnerables**: revisar package.json/requirements.txt si están en el alcance

#### SQL Injection por lenguaje:

**JavaScript/TypeScript:**
```javascript
// ❌ Vulnerable
`SELECT * FROM users WHERE id = ${id}`
// ✅ Seguro
db.query('SELECT * FROM users WHERE id = ?', [id])
```

**Python:**
```python
# ❌ Vulnerable
f"SELECT * FROM users WHERE id = {id}"
# ✅ Seguro
cursor.execute("SELECT * FROM users WHERE id = %s", (id,))
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

---

### 🟠 P1 - ALTO: Problemas de Rendimiento

Código que degradará la experiencia del usuario:

- **Operaciones O(n²) o peores** donde existe solución O(n)
- **Queries N+1**: múltiples llamadas a DB/API en loops
- **Operaciones bloqueantes** en código que debería ser async
- **Cálculos costosos** repetidos sin memoización
- **Re-renders innecesarios** en componentes React
- **Bundles inflados**: imports que traen librerías completas
- **Falta de paginación** en listas potencialmente grandes

#### O(n²) → O(n) por lenguaje:

**JavaScript/TypeScript:**
```javascript
// ❌ O(n²)
arr1.forEach(a => arr2.find(b => b.id === a.id))
// ✅ O(n)
const map = new Map(arr2.map(b => [b.id, b]))
arr1.forEach(a => map.get(a.id))
```

**Python:**
```python
# ❌ O(n²)
[x for x in list1 if x in list2]
# ✅ O(n)
set2 = set(list2)
[x for x in list1 if x in set2]
```

**Go:**
```go
// ❌ O(n²)
for _, a := range slice1 {
    for _, b := range slice2 { ... }
}
// ✅ O(n)
m := make(map[string]Item)
for _, b := range slice2 { m[b.ID] = b }
```

#### String concatenation:

| Lenguaje | ❌ Malo (en loop) | ✅ Bueno |
|----------|------------------|---------|
| JS/TS | `result += str` | `parts.join('')` |
| Python | `result += s` | `''.join(strings)` |
| Go | `result += s` | `strings.Builder` |
| Rust | múltiples `push_str` | `String::with_capacity` |
| Java | `result += s` | `StringBuilder` |

---

### 🟠 P1 - ALTO: Errores de Tipado y Contratos

Problemas que causarán bugs sutiles:

| Lenguaje | Problema | Ejemplo |
|----------|----------|---------|
| TypeScript | `any` implícito | `function fn(x)` sin tipo |
| Python | Type hint mal | `def fn(x: str) -> int: return x` |
| Go | `interface{}` | Usar generics si Go 1.18+ |
| Rust | Trait bounds | Bounds innecesariamente complejos |

- **Any implícitos** o casteos forzados sin validación
- **Tipos opcionales** usados sin verificar existencia
- **Interfaces incompletas** que no reflejan la realidad
- **Parámetros con tipos incorrectos** en llamadas a funciones
- **Return types inconsistentes** con lo que realmente retorna

---

### 🟡 P2 - MEDIO: Calidad y Mantenibilidad

Código que dificultará el trabajo futuro:

- **Código duplicado**: bloques repetidos que deberían extraerse (>5 líneas)
- **Funciones demasiado largas** (>50 líneas): difíciles de entender y testear
- **Anidamiento excesivo** (>3 niveles): complejidad cognitiva alta
- **Ternarios anidados**: preferir switch/if-else para claridad
- **Nombres poco descriptivos**: variables de una letra, abreviaciones crípticas
- **Magic numbers/strings**: valores sin explicación ni constantes
- **Comentarios desactualizados**: peor que no tener comentarios
- **Acoplamiento alto**: dependencias circulares, módulos que saben demasiado

### Estándares por lenguaje:

| Lenguaje | Estándar |
|----------|----------|
| JS/TS | ESLint, Prettier |
| Python | PEP 8, Black, Ruff |
| Go | gofmt, golint |
| Rust | rustfmt, clippy |
| PHP | PSR-12 |
| Ruby | RuboCop |

---

### 🟡 P2 - MEDIO: Violaciones de Estándares del Proyecto

Si existe CLAUDE.md, AGENTS.md o configuración de linting, verifica:

- **Imports desordenados** o sin extensiones (si el proyecto las requiere)
- **Arrow functions** donde se espera `function` keyword (o viceversa)
- **Falta de return types** explícitos en funciones públicas
- **Componentes React** sin Props types definidos
- **Patrones de error handling** inconsistentes con el resto del código
- **Convenciones de naming** no seguidas

---

### 🔵 P3 - BAJO: Código Basura y Limpieza

Ruido que debería eliminarse:

| Lenguaje | Debugging a remover |
|----------|---------------------|
| JS/TS | `console.log`, `debugger` |
| Python | `print()`, `breakpoint()` |
| Go | `fmt.Println` debug |
| Rust | `println!`, `dbg!` |
| PHP | `var_dump()`, `dd()` |
| Ruby | `puts`, `binding.pry` |

- **Código comentado**: si no sirve, se borra; Git guarda el historial
- **Variables declaradas sin usar**: dead code
- **Imports no utilizados**: inflan el bundle innecesariamente
- **TODOs obsoletos**: sin fecha ni owner, nunca se resuelven
- **Funciones muertas**: nunca llamadas desde ningún lugar
- **Archivos vacíos o placeholder**: si no tienen contenido útil

---

### 🔵 P3 - BAJO: Oportunidades de Simplificación

Mejoras opcionales que aumentan la elegancia:

- **Lógica que puede simplificarse** sin perder claridad
- **Abstracciones que pueden consolidarse**
- **Patrones modernos** disponibles (optional chaining, nullish coalescing)
- **Utilidades existentes** en el proyecto que podrían reutilizarse

---

## Paso 5: Generar Informe

**Responde directamente en el chat con este formato:**

```markdown
# 📋 INFORME DE REVISIÓN BRO CODE-REVIEW

**Fecha:** [fecha actual]
**Alcance:** `[ruta o descripción]`
**Lenguaje(s):** [detectados]
**Archivos analizados:** [número]
**Líneas totales:** ~[número aproximado]

---

## Resumen Ejecutivo

| Severidad | Cantidad | Descripción |
|-----------|----------|-------------|
| 🔴 P0 Crítico | X | Bugs y seguridad - Requieren atención inmediata |
| 🟠 P1 Alto | X | Rendimiento y tipos - Deberían corregirse |
| 🟡 P2 Medio | X | Calidad - Recomendado corregir |
| 🔵 P3 Bajo | X | Limpieza - Opcional |

**Calificación:** [⭐⭐⭐⭐⭐ Excelente | ⭐⭐⭐⭐ Bueno | ⭐⭐⭐ Aceptable | ⭐⭐ Necesita Trabajo | ⭐ Crítico]

---

## Hallazgos Detallados

### 📁 [ruta/al/archivo.ext]

#### 🔴 P0: [Título descriptivo del problema]

**Líneas:** XX-XX
**Categoría:** [Bug | Seguridad | Rendimiento | Tipos | Calidad | Limpieza]

**Problema:**
[Descripción clara de qué está mal]

**Código actual:**
```[lenguaje]
[fragmento problemático con contexto suficiente]
```

**Corrección sugerida:**
```[lenguaje]
[código corregido]
```

**Impacto si no se corrige:**
[Descripción concreta del escenario de fallo: qué pasará, bajo qué condiciones, qué consecuencias tendrá para usuarios/sistema]

---

[Repetir para cada hallazgo, agrupados por archivo]

---

## Plan de Acción

**Inmediato (P0):** [lista de acciones críticas]
**Corto plazo (P1):** [lista de mejoras importantes]
**Opcional (P2-P3):** [lista de mejoras menores]

---

## ✨ Aspectos Positivos

[Buenas prácticas encontradas en el código - no todo es crítica]

---

## Recomendaciones Finales

[Lista breve de acciones prioritarias para mejorar el código]
```

---

## Reglas de Operación

1. **Solo lectura**: No modificar ningún archivo, solo analizar y reportar
2. **Detecta el lenguaje**: Adapta análisis y ejemplos al lenguaje del proyecto
3. **Interpreta inteligentemente**: Buscar archivos relacionados con lo que pida el usuario
4. **Confirma si hay ambigüedad**: Si hay múltiples coincidencias, pregunta
5. **Sé específico**: Indica líneas exactas, muestra código concreto, no generalices
6. **Prioriza correctamente**: Un bug crítico importa más que 10 mejoras de estilo
7. **Explica el impacto real**: No digas "puede causar problemas", describe el escenario exacto
8. **Propón soluciones idiomáticas**: Patrones del lenguaje detectado
9. **Evita falsos positivos**: Si no estás seguro, márcalo como "posible problema a verificar"
10. **Contexto importa**: Código de tests tiene reglas diferentes a producción
11. **Respeta estándares del proyecto**: Usa CLAUDE.md/AGENTS.md como referencia
12. **Claridad sobre brevedad**: Código explícito es mejor que one-liners crípticos
13. **Reconoce lo bueno**: Menciona prácticas positivas encontradas, no solo problemas
14. **Sé pragmático**: No todo necesita ser perfecto, enfócate en lo que realmente importa
