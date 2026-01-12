---
name: dinnovos-pre-commit-review
description: Auditoría completa de cambios pendientes antes del commit - detecta bugs, problemas de calidad, seguridad y oportunidades de mejora
model: opus
allowed-tools: ["Bash", "Read", "Grep"]
---

# Revisión Pre-Commit Completa

Analiza exhaustivamente todos los cambios pendientes antes de hacer commit. Detecta problemas críticos, evalúa calidad del código y sugiere mejoras concretas.

## Paso 1: Identificar Cambios Pendientes

Ejecuta estos comandos para obtener el estado actual:

```bash
# Archivos modificados (tracked)
git diff --name-only HEAD

# Archivos nuevos (untracked)
git ls-files --others --exclude-standard

# Archivos staged
git diff --cached --name-only
```

### Si no hay cambios

Si todos los comandos devuelven vacío, detente y responde:

```
✅ Directorio de trabajo limpio

No hay cambios pendientes para revisar. Nada que auditar.
```

**No continúes si no hay archivos que revisar.**

## Paso 2: Obtener Contexto Completo

```bash
# Diff completo de cambios no staged
git diff HEAD

# Diff de cambios staged (si existen)
git diff --cached

# Ver estructura del proyecto para contexto
ls -la
```

Si existe `CLAUDE.md`, `AGENTS.md`, `README.md` o archivos de configuración del proyecto (`.eslintrc`, `tsconfig.json`, etc.), léelos para entender los estándares del proyecto.

## Paso 3: Análisis por Categorías

Revisa cada archivo modificado buscando problemas en estas categorías, ordenadas por prioridad:

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

---

### 🔴 P0 - CRÍTICO: Seguridad

Vulnerabilidades que exponen el sistema:

- **Credenciales hardcodeadas**: API keys, passwords, tokens, secrets
- **Inyección**: SQL injection, command injection, XSS
- **Inputs no validados**: datos de usuario usados sin sanitizar
- **Exposición de datos sensibles**: logs con información privada
- **Configuraciones inseguras**: CORS permisivo, HTTPS deshabilitado
- **Dependencias vulnerables**: si se modificó package.json/requirements.txt

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

---

### 🟠 P1 - ALTO: Errores de Tipado y Contratos

Problemas que causarán bugs sutiles:

- **Any implícitos** o casteos forzados sin validación
- **Tipos opcionales** usados sin verificar existencia
- **Interfaces incompletas** que no reflejan la realidad
- **Parámetros con tipos incorrectos** en llamadas a funciones
- **Return types inconsistentes** con lo que realmente retorna

---

### 🟡 P2 - MEDIO: Calidad y Mantenibilidad

Código que dificultará el trabajo futuro:

- **Código duplicado**: bloques repetidos que deberían extraerse
- **Funciones demasiado largas** (>50 líneas): difíciles de entender y testear
- **Anidamiento excesivo** (>3 niveles): complejidad cognitiva alta
- **Ternarios anidados**: preferir switch/if-else para claridad
- **Nombres poco descriptivos**: variables de una letra, abreviaciones crípticas
- **Magic numbers/strings**: valores sin explicación ni constantes
- **Comentarios desactualizados**: peor que no tener comentarios
- **Acoplamiento alto**: dependencias circulares, módulos que saben demasiado

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

Ruido que debería eliminarse antes del commit:

- **Console.log/print/debugger**: código de debugging olvidado
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

## Paso 4: Generar Informe

Usa este formato exacto:

```markdown
# 📋 INFORME DE REVISIÓN DINNOVOS PRE-COMMIT

**Fecha:** [fecha actual]
**Branch:** [nombre del branch]
**Archivos analizados:** [número]
**Líneas modificadas:** ~[número aproximado]

---

## Resumen Ejecutivo

| Severidad | Cantidad | Descripción |
|-----------|----------|-------------|
| 🔴 P0 Crítico | X | Bugs y seguridad - BLOQUEAN el commit |
| 🟠 P1 Alto | X | Rendimiento y tipos - Deberían corregirse |
| 🟡 P2 Medio | X | Calidad - Recomendado corregir |
| 🔵 P3 Bajo | X | Limpieza - Opcional |

**Veredicto:** [✅ APROBADO | ⚠️ CON OBSERVACIONES | ❌ RECHAZADO]

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

## Checklist Pre-Commit

- [ ] Todos los P0 (críticos) están resueltos
- [ ] Los P1 (altos) están resueltos o tienen justificación
- [ ] No hay credenciales o datos sensibles
- [ ] No hay código de debugging (console.log, debugger)
- [ ] Los tests pasan (si aplica)
- [ ] El código compila sin errores

---

## Recomendaciones Finales

[Lista breve de acciones prioritarias antes de hacer commit]
```

---

## Reglas de Operación

1. **Sé específico**: Indica líneas exactas, muestra código concreto, no generalices
2. **Prioriza correctamente**: Un bug crítico importa más que 10 mejoras de estilo
3. **Explica el impacto real**: No digas "puede causar problemas", describe el escenario exacto
4. **Propón soluciones**: Cada problema debe tener una corrección sugerida
5. **Evita falsos positivos**: Si no estás seguro, márcalo como "posible problema a verificar"
6. **Contexto importa**: Código de tests tiene reglas diferentes a producción
7. **Preserva funcionalidad**: Las sugerencias de mejora nunca deben cambiar el comportamiento
8. **Claridad sobre brevedad**: Código explícito es mejor que one-liners crípticos
9. **Respeta los estándares del proyecto**: Si existe CLAUDE.md, AGENTS.md o linting config, síguelo
10. **Sé pragmático**: No todo necesita ser perfecto, enfócate en lo que realmente importa
