---
name: dinnovos-performance-audit
description: Detecta problemas de rendimiento - queries lentas, memory leaks, bundle size, lazy loading, algoritmos ineficientes. Solo lectura.
model: opus
allowed-tools: ["Bash(read-only)", "Read", "Grep", "Glob"]
---

# Auditoría de Rendimiento

Analiza el código en busca de problemas de rendimiento y oportunidades de optimización. **Solo lectura, no modifica nada.**

## Entrada del Usuario

El usuario puede especificar qué auditar de varias formas:

**Ruta exacta:**
- `/dinnovos-performance-audit src/`
- `/dinnovos-performance-audit src/services/dataService.ts`

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
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" \) | grep -v node_modules | grep -v dist | grep -v .git

# Buscar por nombre relacionado
find . -type f -iname "*<término>*" | grep -v node_modules
find . -type d -iname "*<término>*" | grep -v node_modules

# Buscar contenido relacionado
grep -ril "<término>" --include="*.ts" --include="*.tsx" --include="*.js" | grep -v node_modules | head -30
```

**Confirma con el usuario** si encuentras múltiples coincidencias.

### Si no se especificó nada:
Auditar todo el proyecto:

```bash
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.java" -o -name "*.go" \) \
  ! -path "*/node_modules/*" \
  ! -path "*/.git/*" \
  ! -path "*/dist/*" \
  ! -path "*/build/*"
```

**Límite:** Máximo 100 archivos. Si hay más, pide acotar o prioriza por riesgo.

---

## Paso 2: Contexto del Proyecto

```bash
# Estándares y guías
cat CLAUDE.md 2>/dev/null
cat AGENTS.md 2>/dev/null

# Configuración de build y bundle
cat package.json 2>/dev/null
cat webpack.config.js 2>/dev/null
cat vite.config.ts 2>/dev/null
cat next.config.js 2>/dev/null
cat tsconfig.json 2>/dev/null

# Dependencias (para detectar librerías pesadas)
cat package-lock.json 2>/dev/null | head -500
cat yarn.lock 2>/dev/null | head -500
```

---

## Paso 3: Leer y Analizar

```bash
cat [archivo]
wc -l [archivo]
```

Lee cada archivo y realiza el análisis de rendimiento completo.

---

## Paso 4: Categorías de Análisis

### 🔴 P0 - CRÍTICO: Bloqueos y Crashes

Problemas que causan degradación severa:

- **Loops infinitos** o condiciones de salida incorrectas
- **Operaciones síncronas bloqueantes** en código async
- **Memory leaks** evidentes (listeners sin remover, closures que retienen referencias)
- **Recursión sin límite** o caso base incorrecto
- **Deadlocks** en código concurrente

---

### 🟠 P1 - ALTO: Algoritmos Ineficientes

Problemas de complejidad algorítmica:

#### Operaciones O(n²) o peores
```javascript
// ❌ Malo: O(n²)
array1.forEach(item1 => {
  array2.forEach(item2 => {
    if (item1.id === item2.id) { ... }
  });
});

// ✅ Mejor: O(n)
const map = new Map(array2.map(item => [item.id, item]));
array1.forEach(item1 => {
  const item2 = map.get(item1.id);
});
```

#### Búsquedas repetidas en arrays
```javascript
// ❌ Malo: includes/find en cada iteración
items.filter(item => selectedIds.includes(item.id));

// ✅ Mejor: usar Set
const selectedSet = new Set(selectedIds);
items.filter(item => selectedSet.has(item.id));
```

#### Ordenamientos innecesarios
```javascript
// ❌ Malo: ordenar en cada render
{items.sort((a, b) => a.date - b.date).map(...)}

// ✅ Mejor: memoizar
const sortedItems = useMemo(() => 
  [...items].sort((a, b) => a.date - b.date), [items]);
```

---

### 🟠 P1 - ALTO: Problemas de Base de Datos

#### Queries N+1
```javascript
// ❌ Malo: N+1 queries
const users = await User.findAll();
for (const user of users) {
  user.orders = await Order.findByUserId(user.id); // N queries
}

// ✅ Mejor: eager loading o join
const users = await User.findAll({ include: Order });
```

#### Falta de índices (detectar por patrones)
```javascript
// Buscar: WHERE/find por campos que deberían tener índice
findBy({ email: ... })  // email debería tener índice
findBy({ status: ..., createdAt: ... })  // índice compuesto
```

#### Queries sin límite
```javascript
// ❌ Malo: sin paginación
const allUsers = await User.findAll();

// ✅ Mejor: paginado
const users = await User.findAll({ limit: 50, offset: page * 50 });
```

#### SELECT * cuando solo se necesitan campos específicos
```javascript
// ❌ Malo: trae todo
const users = await db.query('SELECT * FROM users');

// ✅ Mejor: solo lo necesario
const users = await db.query('SELECT id, name FROM users');
```

---

### 🟠 P1 - ALTO: Memory Leaks

#### Event listeners sin cleanup
```javascript
// ❌ Malo: nunca se remueve
useEffect(() => {
  window.addEventListener('resize', handleResize);
}, []);

// ✅ Mejor: cleanup
useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

#### Subscripciones sin unsubscribe
```javascript
// ❌ Malo: subscription leak
useEffect(() => {
  const sub = observable.subscribe(handler);
}, []);

// ✅ Mejor: cleanup
useEffect(() => {
  const sub = observable.subscribe(handler);
  return () => sub.unsubscribe();
}, []);
```

#### Timers sin clear
```javascript
// ❌ Malo: interval nunca se limpia
useEffect(() => {
  setInterval(poll, 5000);
}, []);

// ✅ Mejor: cleanup
useEffect(() => {
  const id = setInterval(poll, 5000);
  return () => clearInterval(id);
}, []);
```

#### Closures que retienen referencias grandes
```javascript
// ❌ Malo: retiene todo heavyData
function createHandler(heavyData) {
  return () => console.log(heavyData.length);
}

// ✅ Mejor: solo lo necesario
function createHandler(dataLength) {
  return () => console.log(dataLength);
}
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

#### Imágenes sin optimizar
```javascript
// ❌ Malo: imagen original
<img src="/photo.jpg" />

// ✅ Mejor: responsive y lazy
<img 
  src="/photo.jpg" 
  srcSet="/photo-400.jpg 400w, /photo-800.jpg 800w"
  loading="lazy"
/>
```

#### Falta de lazy loading en rutas/componentes
```javascript
// ❌ Malo: importa todo upfront
import HeavyComponent from './HeavyComponent';

// ✅ Mejor: lazy load
const HeavyComponent = lazy(() => import('./HeavyComponent'));
```

#### Bundle size excesivo
```javascript
// ❌ Malo: importa toda la librería
import _ from 'lodash';

// ✅ Mejor: importa solo lo necesario
import debounce from 'lodash/debounce';
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

#### Operaciones síncronas costosas en hot paths
```javascript
// ❌ Malo: JSON.parse en cada request
app.get('/data', (req, res) => {
  const data = JSON.parse(fs.readFileSync('large-file.json'));
});

// ✅ Mejor: cargar una vez
const data = JSON.parse(fs.readFileSync('large-file.json'));
app.get('/data', (req, res) => res.json(data));
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

#### Procesamiento en request que debería ser async
```javascript
// ❌ Malo: bloquea el request
app.post('/report', async (req, res) => {
  const report = await generateHeavyReport(req.body); // 30 segundos
  res.json(report);
});

// ✅ Mejor: job queue
app.post('/report', async (req, res) => {
  const jobId = await queue.add('generateReport', req.body);
  res.json({ jobId, status: 'processing' });
});
```

---

### 🟡 P2 - MEDIO: Cálculos Repetidos

#### Sin memoización
```javascript
// ❌ Malo: recalcula siempre
function Component({ items }) {
  const total = items.reduce((sum, item) => sum + item.price, 0);
  const sorted = items.sort((a, b) => a.name.localeCompare(b.name));
}

// ✅ Mejor: memoizar
function Component({ items }) {
  const total = useMemo(() => 
    items.reduce((sum, item) => sum + item.price, 0), [items]);
  const sorted = useMemo(() => 
    [...items].sort((a, b) => a.name.localeCompare(b.name)), [items]);
}
```

#### Regex compilados en cada llamada
```javascript
// ❌ Malo: compila regex cada vez
function validate(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

// ✅ Mejor: compilar una vez
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
function validate(email) {
  return EMAIL_REGEX.test(email);
}
```

---

### 🔵 P3 - BAJO: Optimizaciones Menores

- **Console.log en producción**: Afecta rendimiento en loops
- **Spread innecesario**: `{...obj}` cuando no se necesita copia
- **Async/await en operaciones síncronas**: Overhead innecesario
- **Concatenación de strings en loops**: Usar array.join()
- **Múltiples accesos a DOM**: Cachear referencias

---

## Paso 5: Generar Informe

**Responde directamente en el chat:**

```markdown
# ⚡ INFORME DE RENDIMIENTO DINNOVOS

**Fecha:** [fecha actual]
**Alcance:** `[ruta, descripción o "proyecto completo"]`
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

### Imports Optimizables
```javascript
// ❌ Actual
import _ from 'lodash';

// ✅ Sugerido
import debounce from 'lodash/debounce';
```

---

## 🎯 Métricas Clave a Monitorear

| Métrica | Estado Actual | Objetivo |
|---------|---------------|----------|
| Queries por request | [Desconocido/Alto/OK] | < 10 |
| Bundle size (JS) | [Desconocido/Grande/OK] | < 200KB |
| Memory leaks | [X detectados] | 0 |
| Componentes sin memo | [X detectados] | Minimizar |

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

Para validar y monitorear:
- **Frontend:** React DevTools Profiler, Lighthouse, Bundle Analyzer
- **Backend:** APM (DataDog, New Relic), Query analyzers
- **General:** Chrome DevTools Performance tab
```

---

## Reglas de Operación

1. **Solo lectura**: No modificar ningún archivo, solo analizar y reportar
2. **Interpreta inteligentemente**: Buscar archivos relacionados con lo que pida el usuario
3. **Confirma si hay ambigüedad**: Si hay múltiples coincidencias, pregunta
4. **Sé específico**: Indica archivos, líneas y código exacto
5. **Cuantifica cuando sea posible**: "O(n²) en array de 10K items = ~100M operaciones"
6. **Prioriza por impacto**: Bloqueos > Algoritmos > Memory > UI
7. **Propón soluciones**: Cada problema debe tener código corregido
8. **Evita falsos positivos**: No todo loop anidado es malo
9. **Considera el contexto**: Un O(n²) con n=10 no es problema
10. **Respeta estándares del proyecto**: Usa CLAUDE.md/AGENTS.md como referencia
11. **Sugiere herramientas**: Para validar las mejoras
12. **Reconoce lo bueno**: Menciona optimizaciones ya implementadas
