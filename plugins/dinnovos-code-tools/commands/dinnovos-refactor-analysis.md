---
name: dinnovos-refactor-analysis
description: Analiza código en busca de duplicaciones, lógica similar y oportunidades de refactorización. Solo lectura.
model: opus
allowed-tools: ["Bash(read-only)", "Read", "Grep", "Glob"]
---

# Análisis de Refactorización

Analiza el código en busca de duplicaciones y oportunidades de refactorización. **Solo lectura, no modifica nada.**

## Entrada del Usuario

El usuario puede especificar qué analizar de varias formas:

**Ruta exacta:**
- `/dinnovos-refactor-analysis src/components/`
- `/dinnovos-refactor-analysis src/services/userService.ts`

**Lenguaje natural (ejemplos ilustrativos):**
- `/dinnovos-refactor-analysis analiza los componentes de <área>`
- `/dinnovos-refactor-analysis busca duplicados en <módulo>`
- `/dinnovos-refactor-analysis revisa oportunidades en los servicios de <funcionalidad>`
- `/dinnovos-refactor-analysis analiza todo lo relacionado con <tema>`

**Sin argumentos:**
- `/dinnovos-refactor-analysis` → analiza todo el proyecto

> **Nota:** Los términos como "UI", "autenticación", "pagos" son solo ejemplos. Interpreta lo que el usuario solicite y busca los archivos correspondientes en el proyecto.

---

## Paso 1: Interpretar la Solicitud

### Si es ruta exacta:
Usar directamente.

### Si es lenguaje natural:
Buscar archivos que coincidan con la descripción:

```bash
# Explorar estructura del proyecto
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" \) | grep -v node_modules | grep -v dist | grep -v .git

# Buscar por nombre relacionado (reemplaza <término> con lo que pidió el usuario)
find . -type f -iname "*<término>*" | grep -v node_modules
find . -type d -iname "*<término>*" | grep -v node_modules

# Buscar contenido relacionado
grep -ril "<término>" --include="*.ts" --include="*.tsx" --include="*.js" | grep -v node_modules | head -30
```

**Confirma con el usuario** si encuentras múltiples coincidencias:
```
Encontré estos archivos/carpetas relacionados con "<término>":
1. src/components/[Carpeta1]/
2. src/services/[Archivo1].ts
3. src/hooks/[Archivo2].ts

¿Analizo todos o alguno específico?
```

Si solo hay una coincidencia clara, procede directamente.

### Si no se especificó nada:
Analizar todo el proyecto:

```bash
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.java" -o -name "*.go" -o -name "*.php" -o -name "*.rb" -o -name "*.cs" -o -name "*.vue" -o -name "*.svelte" \) \
  ! -path "*/node_modules/*" \
  ! -path "*/.git/*" \
  ! -path "*/dist/*" \
  ! -path "*/build/*" \
  ! -path "*/__pycache__/*" \
  ! -path "*/vendor/*" \
  ! -path "*/.next/*"
```

**Límite:** Máximo 100 archivos. Si hay más, pide acotar o prioriza por tamaño.

---

## Paso 2: Contexto del Proyecto

Busca y lee archivos de configuración y estándares:

```bash
# Estándares y guías del proyecto
cat CLAUDE.md 2>/dev/null
cat AGENTS.md 2>/dev/null
cat .cursor/rules.md 2>/dev/null

# Configuración de linting y tipos
cat .eslintrc* 2>/dev/null
cat tsconfig.json 2>/dev/null
cat biome.json 2>/dev/null
```

Usa esta información para entender la estructura y convenciones del proyecto.

---

## Paso 3: Leer y Analizar

```bash
cat [archivo]
wc -l [archivo]
```

Lee cada archivo y realiza el análisis completo.

---

## Paso 4: Categorías de Análisis

### 🔄 1. Código Duplicado

Bloques de código idénticos o casi idénticos (>5 líneas) en múltiples lugares.

**Buscar:**
- Funciones con mismo cuerpo
- Bloques copy-paste
- Lógica repetida con diferentes nombres

**Reportar:**
- Archivos y líneas exactas
- Porcentaje de similitud
- Nombre sugerido para extracción

---

### 🧩 2. Lógica Similar

Funciones o bloques que hacen cosas parecidas con pequeñas variaciones.

**Buscar:**
- Mismo patrón con diferentes datos
- Validaciones similares
- Transformaciones de datos análogas
- Handlers con estructura repetida

**Reportar:**
- Qué hace cada uno
- En qué se diferencian
- Cómo unificarlos (parámetros, generics, etc.)

---

### 📦 3. Funciones Repetidas

Funciones con el mismo propósito en diferentes archivos.

**Buscar:**
- Utilidades duplicadas (formatDate, capitalize, slugify, etc.)
- Helpers repetidos
- Funciones de validación similares

**Reportar:**
- Nombre y ubicación de cada una
- Diferencias entre implementaciones
- Cuál es la mejor implementación

---

### 🏗️ 4. Clases/Componentes Similares

Clases o componentes con estructura o comportamiento parecido.

**Buscar:**
- Componentes UI con variaciones menores
- Clases con métodos casi idénticos
- Services/Controllers con lógica repetida

**Reportar:**
- Qué tienen en común
- Qué los diferencia
- Patrón de abstracción sugerido (herencia, composición, HOC, etc.)

---

### 🔢 5. Constantes y Magic Numbers

Valores hardcodeados repetidos.

**Buscar:**
- Números mágicos repetidos (timeouts, límites, etc.)
- Strings duplicados (URLs, mensajes, keys)
- Configuraciones dispersas

**Reportar:**
- Valor repetido
- Archivos y líneas donde aparece
- Nombre sugerido para la constante

---

### 📝 6. Inconsistencias de Naming

Variables que representan lo mismo con nombres diferentes.

**Buscar:**
- `user` vs `currentUser` vs `loggedUser` para lo mismo
- `isLoading` vs `loading` vs `isLoad`
- Inconsistencias en convenciones (camelCase vs snake_case)

**Reportar:**
- Las variaciones encontradas
- Nombre consistente sugerido

---

### 🎯 7. Patrones Repetidos

Estructuras de código que se repiten con el mismo propósito.

**Buscar:**
- Try/catch con mismo manejo de error
- Fetch/API calls con estructura idéntica
- useState + useEffect para data fetching
- Mapeos y transformaciones repetidas

**Reportar:**
- El patrón identificado
- Dónde se repite
- Abstracción sugerida (hook, utilidad, wrapper, etc.)

---

## Paso 5: Generar Informe

**Responde directamente en el chat:**

```markdown
# 📊 INFORME DE ANÁLISIS DINNOVOS REFACTOR

**Fecha:** [fecha actual]
**Alcance:** `[ruta, descripción o "proyecto completo"]`
**Archivos analizados:** [número]
**Líneas de código totales:** ~[número]

---

## Resumen Ejecutivo

| Categoría | Ocurrencias | Impacto |
|-----------|-------------|---------|
| 🔄 Código duplicado | X | Alto/Medio/Bajo |
| 🧩 Lógica similar | X | Alto/Medio/Bajo |
| 📦 Funciones repetidas | X | Alto/Medio/Bajo |
| 🏗️ Componentes similares | X | Alto/Medio/Bajo |
| 🔢 Constantes duplicadas | X | Alto/Medio/Bajo |
| 📝 Inconsistencias naming | X | Alto/Medio/Bajo |
| 🎯 Patrones repetidos | X | Alto/Medio/Bajo |

**Líneas potencialmente reducibles:** ~[número]
**Prioridad de refactorización:** [🔴 Alta | 🟠 Media | 🟢 Baja]

---

## 🔄 Código Duplicado

### Duplicación #1: [Nombre descriptivo]

**Similitud:** X%
**Líneas afectadas:** X

| Ubicación | Líneas |
|-----------|--------|
| `src/components/UserCard.tsx` | 45-67 |
| `src/components/AdminCard.tsx` | 32-54 |

**Código duplicado:**
```[lang]
// Fragmento representativo (máx 15 líneas)
```

**Sugerencia:** Extraer a `components/shared/ProfileCard.tsx` con props para variaciones.

---

[Repetir para cada duplicación encontrada]

---

## 🧩 Lógica Similar

### Caso #1: [Nombre descriptivo]

**Archivos involucrados:**
- `src/services/userService.ts` → función `validateUser()`
- `src/services/adminService.ts` → función `validateAdmin()`

**Qué hacen:**
[Descripción breve]

**Diferencias:**
[Lista de diferencias]

**Código actual:**
```[lang]
// Fragmento de la primera
```
```[lang]
// Fragmento de la segunda
```

**Sugerencia:** [Cómo unificar]

---

[Repetir para cada categoría con hallazgos]

---

## 📋 Plan de Acción Recomendado

### 🔴 Prioridad Alta (hacer primero)
1. [Acción específica] — Impacto: X líneas, X archivos
2. [Acción específica] — Impacto: X líneas, X archivos

### 🟠 Prioridad Media
1. [Acción específica]
2. [Acción específica]

### 🟢 Prioridad Baja (nice to have)
1. [Acción específica]
2. [Acción específica]

---

## 📈 Estadísticas Adicionales

| Métrica | Top 5 |
|---------|-------|
| Archivos con más duplicación | [lista] |
| Funciones más largas | [lista con líneas] |
| Archivos más grandes | [lista con líneas] |

---

## ✨ Buenas Prácticas Encontradas

[Patrones positivos, abstracciones bien hechas, código limpio identificado]
```

---

## Reglas de Operación

1. **Solo lectura**: No modificar ningún archivo, solo analizar y reportar
2. **Interpreta inteligentemente**: Buscar archivos relacionados con lo que pida el usuario
3. **Confirma si hay ambigüedad**: Si hay múltiples coincidencias, pregunta
4. **Sé exhaustivo**: Lee todos los archivos del alcance
5. **Sé específico**: Indica archivos y líneas exactas
6. **Prioriza por impacto**: Lo que más se repite primero
7. **Sugiere soluciones concretas**: No solo señales problemas, propón extracciones
8. **Ignora falsos positivos**: Código similar por necesidad (tests, migrations) no cuenta
9. **Considera el contexto**: A veces la duplicación es intencional
10. **Respeta estándares del proyecto**: Usa CLAUDE.md/AGENTS.md como referencia
11. **Reconoce lo bueno**: Menciona abstracciones bien hechas
12. **Cuantifica el impacto**: "X líneas reducibles" ayuda a priorizar
