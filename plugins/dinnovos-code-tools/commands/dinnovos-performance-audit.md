---
name: dinnovos-performance-audit
description: Detecta problemas de rendimiento - queries lentas, memory leaks, bundle size, lazy loading, algoritmos ineficientes. Soporta múltiples lenguajes. Solo lectura.
model: opus
allowed-tools: ["Bash(read-only)", "Read", "Grep", "Glob"]
---

# Auditoría de Rendimiento

Analiza el código en busca de problemas de rendimiento y oportunidades de optimización. **Solo lectura, no modifica nada. Soporta múltiples lenguajes.**

## Entrada del Usuario

El usuario puede especificar qué auditar de varias formas:

**Ruta exacta:**
- `/dinnovos-performance-audit src/`
- `/dinnovos-performance-audit src/services/dataService.ts`
- `/dinnovos-performance-audit app/handlers/`

**Lenguaje natural (ejemplos ilustrativos):**
- `/dinnovos-performance-audit analiza rendimiento de <módulo>`
- `/dinnovos-performance-audit busca memory leaks en <área>`
- `/dinnovos-performance-audit revisa queries en <servicio>`
- `/dinnovos-performance-audit optimizaciones para <componente>`
- `/dinnovos-performance-audit por qué es lento <funcionalidad>`

**Sin argumentos:**
- `/dinnovos-performance-audit` → audita todo el proyecto

> **Nota:** Los términos como "API", "dashboard", "reportes" son solo ejemplos. Interpreta lo que el usuario solicite y busca los archivos correspondientes en el proyecto.

---

## Paso 1: Interpretar la Solicitud

### Si es ruta exacta:
Usar directamente.

### Si es lenguaje natural:
Buscar archivos que coincidan con la descripción:

```bash
# Explorar estructura del proyecto
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.go" -o -name "*.rs" -o -name "*.java" \) \
  ! -path "*/node_modules/*" ! -path "*/.git/*" ! -path "*/dist/*" ! -path "*/build/*" ! -path "*/target/*"

# Buscar por nombre relacionado
find . -type f -iname "*<término>*" | grep -v node_modules
find . -type d -iname "*<término>*" | grep -v node_modules

# Buscar contenido relacionado
grep -ril "<término>" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.py" --include="*.go" | grep -v node_modules | head -30
```

**Confirma con el usuario** si encuentras múltiples coincidencias.

**Límite:** Máximo 100 archivos. Si hay más, pide acotar o prioriza por riesgo.

---

## Paso 2: Contexto del Proyecto

```bash
# Estándares y guías
cat CLAUDE.md 2>/dev/null
cat AGENTS.md 2>/dev/null

# Detectar stack
cat package.json pyproject.toml go.mod Cargo.toml composer.json 2>/dev/null

# Configuración de build y bundle
cat webpack.config.js 2>/dev/null
cat vite.config.ts 2>/dev/null
cat next.config.js 2>/dev/null
cat tsconfig.json 2>/dev/null
```

---

## Paso 3: Leer y Analizar

```bash
cat [archivo]
wc -l [archivo]
```

Lee cada archivo y realiza el análisis de rendimiento completo.

---

## Paso 4: Análisis por Categorías

### 🔴 P0 - CRÍTICO: Bloqueos y Crashes

Problemas que causan degradación severa:

- **Loops infinitos** o condiciones de salida incorrectas
- **Operaciones síncronas bloqueantes** en código async
- **Memory leaks** evidentes (listeners sin remover, closures que retienen referencias)
- **Recursión sin límite** o caso base incorrecto
- **Deadlocks** en código concurrente

#### Ejemplos por lenguaje:

**JavaScript/TypeScript:**
```javascript
// ❌ Operación síncrona bloqueante
const data = fs.readFileSync('huge-file.json')

// ❌ Loop infinito potencial
while (condition) { /* sin break */ }
```

**Python:**
```python
# ❌ Carga todo en memoria
data = file.read()  # archivo de 10GB

# ❌ Recursión sin límite
def recursive(n):
    return recursive(n)  # sin caso base
```

**Go:**
```go
// ❌ Goroutine leak
go func() {
    for { /* sin salida */ }
}()

// ❌ Deadlock
mu.Lock()
mu.Lock()  // mismo mutex
```

**Rust:**
```rust
// ❌ Loop sin salida
loop { /* sin break */ }
```

---

### 🟠 P1 - ALTO: Algoritmos Ineficientes

Problemas de complejidad algorítmica.

#### O(n²) → O(n) por lenguaje:

**JavaScript/TypeScript:**
```javascript
// ❌ O(n²)
arr1.forEach(a => arr2.find(b => b.id === a.id))
items.filter(i => ids.includes(i.id))

// ✅ O(n)
const map = new Map(arr2.map(b => [b.id, b]))
const idSet = new Set(ids)
items.filter(i => idSet.has(i.id))
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
    for _, b := range slice2 {
        if a.ID == b.ID { ... }
    }
}

// ✅ O(n)
m := make(map[string]Item, len(slice2))
for _, b := range slice2 { m[b.ID] = b }
```

**Rust:**
```rust
// ❌ O(n²)
for a in &vec1 {
    if vec2.contains(a) { ... }
}

// ✅ O(n)
let set: HashSet<_> = vec2.iter().collect();
```

---

### 🟠 P1 - ALTO: Queries N+1

**JavaScript/TypeScript:**
```javascript
// ❌ N+1
const users = await User.findAll()
for (const user of users) {
    user.orders = await Order.findByUser(user.id)
}

// ✅ Eager loading
const users = await User.findAll({ include: Order })
```

**Python:**
```python
# ❌ N+1
users = User.query.all()
for user in users:
    orders = Order.query.filter_by(user_id=user.id).all()

# ✅ Eager loading
users = User.query.options(joinedload(User.orders)).all()
```

**Go:**
```go
// ❌ N+1
for _, user := range users {
    orders, _ := db.Query("SELECT * FROM orders WHERE user_id = ?", user.ID)
}

// ✅ Batch
db.Query("SELECT * FROM orders WHERE user_id IN (?)", userIDs)
```

---

### 🟠 P1 - ALTO: Memory Leaks

**JavaScript/TypeScript:**
```javascript
// ❌ Event listener leak
useEffect(() => {
    window.addEventListener('resize', handler)
}, [])

// ✅ Con cleanup
useEffect(() => {
    window.addEventListener('resize', handler)
    return () => window.removeEventListener('resize', handler)
}, [])
```

**Python:**
```python
# ❌ Conexión no cerrada
conn = psycopg2.connect(...)
cursor = conn.cursor()

# ✅ Context manager
with psycopg2.connect(...) as conn:
    with conn.cursor() as cursor:
        ...
```

**Go:**
```go
// ❌ Goroutine leak
go func() {
    for { <-ch }  // ch nunca cierra
}()

// ✅ Con context
go func(ctx context.Context) {
    for {
        select {
        case <-ctx.Done(): return
        case <-ch: ...
        }
    }
}(ctx)
```

---

### 🟡 P2 - MEDIO: String Concatenation

| Lenguaje | ❌ Malo (en loop) | ✅ Bueno |
|----------|------------------|---------|
| JS/TS | `result += str` | `parts.join('')` |
| Python | `result += s` | `''.join(strings)` |
| Go | `result += s` | `strings.Builder` |
| Rust | múltiples `push_str` | `String::with_capacity` |
| Java | `result += s` | `StringBuilder` |

---

### 🟡 P2 - MEDIO: Falta de Memoización

**JavaScript/TypeScript:**
```javascript
// ❌ Recalcula cada render
const sorted = items.sort(...)

// ✅ Memoizado
const sorted = useMemo(() => [...items].sort(...), [items])
```

**Python:**
```python
# ❌ Recalcula siempre
def expensive(n): return sum(range(n))

# ✅ Con cache
@lru_cache(maxsize=128)
def expensive(n): return sum(range(n))
```

---

### 🟡 P2 - MEDIO: Imports Pesados

```javascript
// ❌ Import completo
import _ from 'lodash'        // ~70KB
import moment from 'moment'   // ~300KB

// ✅ Import específico
import debounce from 'lodash/debounce'
import { format } from 'date-fns'
```

---

### 🟡 P2 - MEDIO: Problemas de Frontend

#### Re-renders innecesarios
```javascript
// ❌ Malo: nuevo objeto en cada render
<Component style={{ margin: 10 }} />
<Component onClick={() => handleClick(id)} />

// ✅ Mejor: memoizar
const style = useMemo(() => ({ margin: 10 }), []);
const handleClickMemo = useCallback(() => handleClick(id), [id]);
```

#### Falta de virtualización en listas largas
```javascript
// ❌ Malo: renderiza 10,000 items
{items.map(item => <Row key={item.id} {...item} />)}

// ✅ Mejor: virtualizar
<VirtualList items={items} renderItem={item => <Row {...item} />} />
```

#### Falta de lazy loading
```javascript
// ❌ Malo: importa todo upfront
import HeavyComponent from './HeavyComponent';

// ✅ Mejor: lazy load
const HeavyComponent = lazy(() => import('./HeavyComponent'));
```

---

### 🟡 P2 - MEDIO: Problemas de Backend

#### Falta de caching
```javascript
// ❌ Malo: siempre calcula/fetch
async function getConfig() {
  return await db.query('SELECT * FROM config');
}

// ✅ Mejor: cachear
let configCache = null;
async function getConfig() {
  if (!configCache) {
    configCache = await db.query('SELECT * FROM config');
  }
  return configCache;
}
```

#### Falta de connection pooling
```javascript
// ❌ Malo: nueva conexión por request
async function query(sql) {
  const conn = await mysql.createConnection(config);
  const result = await conn.query(sql);
  conn.close();
  return result;
}

// ✅ Mejor: pool
const pool = mysql.createPool(config);
async function query(sql) {
  return pool.query(sql);
}
```

---

### 🔵 P3 - BAJO: Micro-optimizaciones

- Console/print en loops
- Regex compilado en cada llamada
- Spread/clone innecesario
- Async/await en operaciones síncronas

| Lenguaje | Debugging a remover |
|----------|---------------------|
| JS/TS | `console.log` en loops |
| Python | `print()` en loops |
| Go | `fmt.Println` debug |
| Rust | `println!`, `dbg!` |

---

## Paso 5: Generar Informe

**Responde directamente en el chat:**

```markdown
# ⚡ INFORME DE RENDIMIENTO DINNOVOS

**Fecha:** [fecha actual]
**Alcance:** `[ruta, descripción o "proyecto completo"]`
**Lenguaje(s):** [detectados]
**Archivos analizados:** [número]
**Líneas de código:** ~[número]

---

## Resumen Ejecutivo

| Severidad | Cantidad | Impacto Estimado |
|-----------|----------|------------------|
| 🔴 P0 Crítico | X | Bloqueos/Crashes |
| 🟠 P1 Alto | X | Degradación severa |
| 🟡 P2 Medio | X | Lentitud notable |
| 🔵 P3 Bajo | X | Micro-optimizaciones |

**Estado de rendimiento:** [🔴 Crítico | 🟠 Necesita trabajo | 🟡 Aceptable | 🟢 Optimizado]

### Áreas Más Afectadas
1. [Área] — [cantidad] problemas
2. [Área] — [cantidad] problemas

---

## 🔴 Problemas Críticos (P0)

### PERF-001: [Título descriptivo]

**Categoría:** [Memory Leak | Loop Infinito | Bloqueo | etc.]
**Severidad:** 🔴 Crítica
**Impacto estimado:** [Descripción del impacto]

**Ubicación:**
- Archivo: `path/to/file.ts`
- Línea(s): XX-XX
- Función: `[nombre]`

**Código actual:**
```[lang]
[fragmento problemático]
```

**Problema:**
[Explicación de por qué es un problema de rendimiento]

**Solución sugerida:**
```[lang]
[código optimizado]
```

**Mejora esperada:** [Descripción cuantitativa si es posible]

---

[Repetir para cada problema, agrupados por severidad]

---

## 🟠 Problemas Altos (P1)

### PERF-002: ...

---

## 🟡 Problemas Medios (P2)

### PERF-003: ...

---

## 🔵 Optimizaciones Menores (P3)

### PERF-004: ...

---

## 📊 Análisis por Categoría

### 🗄️ Base de Datos
| Problema | Ubicación | Severidad |
|----------|-----------|-----------|
| [N+1 Query] | `src/services/user.ts:45` | 🟠 |
| [Sin índice] | `src/models/order.ts:23` | 🟡 |

### 🧠 Memoria
| Problema | Ubicación | Severidad |
|----------|-----------|-----------|
| [Event listener leak] | `src/components/Chat.tsx:34` | 🔴 |

### 🖥️ Frontend
| Problema | Ubicación | Severidad |
|----------|-----------|-----------|
| [Re-renders] | `src/pages/Dashboard.tsx:67` | 🟡 |

### ⚙️ Backend
| Problema | Ubicación | Severidad |
|----------|-----------|-----------|
| [Sin caching] | `src/api/config.ts:12` | 🟡 |

---

## 📦 Análisis de Bundle (si aplica)

### Dependencias Pesadas Detectadas
| Paquete | Tamaño Est. | Uso | Alternativa |
|---------|-------------|-----|-------------|
| `moment` | ~300KB | Formateo fechas | `date-fns` (~30KB) |
| `lodash` | ~70KB | 2 funciones | Import específico |

---

## 📋 Plan de Optimización

### 🔴 Inmediato (esta semana)
1. [Problema crítico] — Archivo: X — Impacto: [alto]
2. [Problema crítico] — Archivo: Y — Impacto: [alto]

### 🟠 Corto plazo (este mes)
1. [Problema alto] — Archivo: X
2. [Problema alto] — Archivo: Y

### 🟡 Mediano plazo
1. [Problema medio] — Archivo: X

### 🔵 Backlog
1. [Optimización menor]

---

## ✨ Buenas Prácticas de Rendimiento Encontradas

[Patrones positivos: uso correcto de memoización, lazy loading implementado, queries optimizadas, caching apropiado, etc.]

---

## 🛠️ Herramientas Recomendadas

| Lenguaje | Herramienta |
|----------|-------------|
| JS/TS | Lighthouse, Bundle Analyzer, React DevTools Profiler |
| Python | cProfile, py-spy |
| Go | pprof |
| Rust | cargo flamegraph |
```

---

## Reglas de Operación

1. **Solo lectura**: No modificar ningún archivo, solo analizar y reportar
2. **Detecta el lenguaje**: Adapta patrones de análisis al lenguaje del proyecto
3. **Interpreta inteligentemente**: Buscar archivos relacionados con lo que pida el usuario
4. **Confirma si hay ambigüedad**: Si hay múltiples coincidencias, pregunta
5. **Sé específico**: Indica archivos, líneas y código exacto
6. **Cuantifica cuando sea posible**: "O(n²) en array de 10K items = ~100M operaciones"
7. **Prioriza por impacto**: Bloqueos > Algoritmos > Memory > UI
8. **Propón soluciones idiomáticas**: Cada problema debe tener código corregido
9. **Evita falsos positivos**: No todo loop anidado es malo
10. **Considera el contexto**: Un O(n²) con n=10 no es problema
11. **Respeta estándares del proyecto**: Usa CLAUDE.md/AGENTS.md como referencia
12. **Sugiere herramientas**: Para validar las mejoras
13. **Reconoce lo bueno**: Menciona optimizaciones ya implementadas
