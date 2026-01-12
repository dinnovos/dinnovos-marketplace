---
name: dinnovos-codebase-explain
description: Explica la arquitectura, estructura y flujos de un proyecto o módulo. Ideal para onboarding o entender código nuevo. Solo lectura.
model: opus
allowed-tools: ["Bash(read-only)", "Read", "Grep", "Glob"]
---

# Explicación de Codebase

Analiza y explica la arquitectura, estructura y flujos de un proyecto o módulo. **Solo lectura, no modifica nada.**

## Entrada del Usuario

El usuario puede especificar qué explicar de varias formas:

**Ruta exacta:**
- `/dinnovos-codebase-explain src/`
- `/dinnovos-codebase-explain src/modules/payments/`

**Lenguaje natural (ejemplos ilustrativos):**
- `/dinnovos-codebase-explain explica el proyecto`
- `/dinnovos-codebase-explain cómo funciona el módulo de <área>`
- `/dinnovos-codebase-explain explica la arquitectura de <funcionalidad>`
- `/dinnovos-codebase-explain qué hace el servicio de <tema>`
- `/dinnovos-codebase-explain cómo se conectan los componentes de <módulo>`

**Sin argumentos:**
- `/dinnovos-codebase-explain` → explica todo el proyecto

> **Nota:** Los términos como "autenticación", "pagos", "usuarios" son solo ejemplos. Interpreta lo que el usuario solicite y busca los archivos correspondientes en el proyecto.

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

**Confirma con el usuario** si hay ambigüedad sobre qué explicar.

### Si no se especificó nada:
Explorar todo el proyecto.

---

## Paso 2: Exploración Inicial

```bash
# Estructura de carpetas (2 niveles)
find . -type d -maxdepth 3 | grep -v node_modules | grep -v .git | grep -v dist | sort

# Archivos principales
ls -la
cat README.md 2>/dev/null
cat package.json 2>/dev/null
cat tsconfig.json 2>/dev/null

# Estándares del proyecto
cat CLAUDE.md 2>/dev/null
cat AGENTS.md 2>/dev/null
cat .cursor/rules.md 2>/dev/null
cat ARCHITECTURE.md 2>/dev/null
cat CONTRIBUTING.md 2>/dev/null
```

---

## Paso 3: Análisis Profundo

### 3.1 Puntos de Entrada
```bash
# Identificar entry points
cat src/index.ts 2>/dev/null
cat src/main.ts 2>/dev/null
cat src/app.ts 2>/dev/null
cat src/App.tsx 2>/dev/null
cat main.py 2>/dev/null
cat cmd/main.go 2>/dev/null
```

### 3.2 Rutas y Endpoints (si aplica)
```bash
# APIs y rutas
find . -type f -name "*route*" | grep -v node_modules
find . -type f -name "*controller*" | grep -v node_modules
find . -type f -name "*endpoint*" | grep -v node_modules
grep -ril "app.get\|app.post\|router\." --include="*.ts" --include="*.js" | grep -v node_modules
```

### 3.3 Modelos y Esquemas
```bash
# Modelos de datos
find . -type f -name "*model*" | grep -v node_modules
find . -type f -name "*schema*" | grep -v node_modules
find . -type f -name "*entity*" | grep -v node_modules
find . -type d -name "models" -o -name "entities" -o -name "schemas" | grep -v node_modules
```

### 3.4 Servicios y Lógica de Negocio
```bash
# Servicios
find . -type f -name "*service*" | grep -v node_modules
find . -type f -name "*usecase*" | grep -v node_modules
find . -type d -name "services" -o -name "usecases" | grep -v node_modules
```

### 3.5 Componentes UI (si aplica)
```bash
# Componentes React/Vue/etc
find . -type f \( -name "*.tsx" -o -name "*.vue" -o -name "*.svelte" \) | grep -v node_modules | head -30
find . -type d -name "components" -o -name "pages" -o -name "views" | grep -v node_modules
```

### 3.6 Configuración
```bash
# Archivos de configuración
find . -type f -name "*.config.*" | grep -v node_modules
find . -type f -name ".env*" | grep -v node_modules
cat .env.example 2>/dev/null
```

### 3.7 Dependencias Clave
```bash
# Dependencias principales
cat package.json 2>/dev/null | grep -A 50 '"dependencies"'
cat requirements.txt 2>/dev/null
cat go.mod 2>/dev/null
```

---

## Paso 4: Categorías de Análisis

### 🏗️ 1. Arquitectura General
- Patrón arquitectónico (MVC, Clean Architecture, Hexagonal, etc.)
- Estructura de carpetas y su propósito
- Separación de responsabilidades
- Capas identificadas

### 🚪 2. Puntos de Entrada
- Entry points de la aplicación
- Cómo se inicializa
- Configuración de bootstrap

### 🔀 3. Flujos Principales
- Flujos de datos más importantes
- Request/Response lifecycle
- Flujos de autenticación (si existe)
- Flujos de negocio críticos

### 📦 4. Módulos y Dependencias
- Módulos principales y su responsabilidad
- Cómo se relacionan entre sí
- Dependencias externas clave
- Dependencias internas (imports entre módulos)

### 💾 5. Capa de Datos
- Cómo se manejan los datos
- Base de datos utilizada
- ORMs o query builders
- Migraciones

### 🎨 6. Capa de Presentación (si aplica)
- Framework de UI
- Estructura de componentes
- Manejo de estado
- Routing

### 🔌 7. Integraciones Externas
- APIs externas consumidas
- Servicios de terceros
- Webhooks
- Colas de mensajes

### ⚙️ 8. Configuración y Entorno
- Variables de entorno
- Archivos de configuración
- Diferentes entornos (dev, staging, prod)

---

## Paso 5: Generar Informe

**Responde directamente en el chat:**

```markdown
# 🗺️ EXPLICACIÓN DE CODEBASE DINNOVOS

**Fecha:** [fecha actual]
**Alcance:** `[ruta, descripción o "proyecto completo"]`
**Archivos analizados:** [número]
**Líneas de código:** ~[número estimado]

---

## 📋 Resumen Ejecutivo

**Tipo de proyecto:** [Web App | API | CLI | Library | Monorepo | etc.]
**Stack tecnológico:** [TypeScript + React + Node | Python + FastAPI | etc.]
**Patrón arquitectónico:** [MVC | Clean Architecture | Hexagonal | etc.]
**Estado general:** [Bien estructurado | Necesita refactor | Legacy | etc.]

### En una oración:
[Descripción breve de qué hace el proyecto y cómo está organizado]

---

## 🏗️ Arquitectura General

### Estructura de Carpetas
```
proyecto/
├── src/
│   ├── [carpeta]/     # [Propósito]
│   ├── [carpeta]/     # [Propósito]
│   └── [carpeta]/     # [Propósito]
├── [archivo]          # [Propósito]
└── [archivo]          # [Propósito]
```

### Patrón Arquitectónico
[Explicación del patrón usado y cómo se implementa]

### Capas Identificadas
| Capa | Ubicación | Responsabilidad |
|------|-----------|-----------------|
| [Presentación] | `src/components/` | [descripción] |
| [Negocio] | `src/services/` | [descripción] |
| [Datos] | `src/repositories/` | [descripción] |

---

## 🚪 Puntos de Entrada

### Entry Point Principal
- **Archivo:** `[ruta]`
- **Función:** [qué hace]

### Inicialización
```[lang]
// Flujo de inicialización resumido
[fragmento clave]
```

---

## 🔀 Flujos Principales

### Flujo 1: [Nombre del flujo - ej: Autenticación de Usuario]

```
[Componente A] → [Componente B] → [Componente C]
     ↓                 ↓                ↓
[Acción]         [Acción]         [Acción]
```

**Archivos involucrados:**
1. `src/[archivo1].ts` → [qué hace]
2. `src/[archivo2].ts` → [qué hace]
3. `src/[archivo3].ts` → [qué hace]

**Descripción:**
[Explicación paso a paso del flujo]

---

### Flujo 2: [Nombre del flujo]
[Repetir estructura]

---

## 📦 Módulos Principales

| Módulo | Ubicación | Responsabilidad | Depende de |
|--------|-----------|-----------------|------------|
| [Auth] | `src/auth/` | [descripción] | [Database, Utils] |
| [Users] | `src/users/` | [descripción] | [Auth, Database] |

### Diagrama de Dependencias
```
┌─────────┐     ┌─────────┐
│  Auth   │────▶│  Users  │
└────┬────┘     └────┬────┘
     │               │
     ▼               ▼
┌─────────────────────────┐
│       Database          │
└─────────────────────────┘
```

---

## 💾 Capa de Datos

**Base de datos:** [PostgreSQL | MongoDB | etc.]
**ORM/Query Builder:** [Prisma | TypeORM | Mongoose | etc.]

### Modelos Principales
| Modelo | Archivo | Campos Clave |
|--------|---------|--------------|
| [User] | `src/models/user.ts` | id, email, password, role |
| [Order] | `src/models/order.ts` | id, userId, items, total |

### Relaciones
```
User (1) ←──────→ (N) Order
Order (1) ←──────→ (N) OrderItem
```

---

## 🎨 Capa de Presentación

**Framework:** [React | Vue | Angular | etc.]
**Manejo de estado:** [Redux | Zustand | Context | etc.]
**Routing:** [React Router | Next.js | etc.]

### Componentes Principales
| Componente | Ubicación | Propósito |
|------------|-----------|-----------|
| [Layout] | `src/components/Layout/` | [descripción] |
| [Dashboard] | `src/pages/Dashboard/` | [descripción] |

---

## 🔌 Integraciones Externas

| Servicio | Propósito | Archivos |
|----------|-----------|----------|
| [Stripe] | Pagos | `src/services/stripe.ts` |
| [SendGrid] | Emails | `src/services/email.ts` |

---

## ⚙️ Configuración

### Variables de Entorno
| Variable | Propósito | Requerida |
|----------|-----------|-----------|
| `DATABASE_URL` | Conexión a BD | ✅ |
| `JWT_SECRET` | Firma de tokens | ✅ |
| `API_KEY` | API externa | ❌ |

### Archivos de Configuración
| Archivo | Propósito |
|---------|-----------|
| `tsconfig.json` | Configuración TypeScript |
| `.env.example` | Template de variables |

---

## 🧭 Cómo Navegar el Código

### Para entender [funcionalidad X]:
1. Empieza en `src/[archivo]`
2. Sigue a `src/[archivo2]`
3. La lógica principal está en `src/[archivo3]`

### Para agregar [nueva feature]:
1. Crear modelo en `src/models/`
2. Crear servicio en `src/services/`
3. Agregar ruta en `src/routes/`
4. Conectar en `src/index.ts`

---

## ⚠️ Puntos de Atención

### Complejidad Alta
- `src/[archivo].ts` — [razón de la complejidad]

### Deuda Técnica Visible
- [Descripción de deuda técnica identificada]

### Áreas Sin Documentar
- [Módulos o funciones que necesitan documentación]

---

## ✨ Buenas Prácticas Identificadas

[Patrones positivos encontrados: separación clara, naming consistente, tests bien organizados, etc.]

---

## 📚 Recursos Adicionales

- README.md: [qué contiene]
- CONTRIBUTING.md: [si existe]
- /docs: [si existe]
```

---

## Reglas de Operación

1. **Solo lectura**: No modificar ningún archivo, solo analizar y explicar
2. **Interpreta inteligentemente**: Buscar archivos relacionados con lo que pida el usuario
3. **Confirma si hay ambigüedad**: Si no está claro qué explicar, pregunta
4. **Sé didáctico**: Explica para alguien que no conoce el proyecto
5. **Usa diagramas**: ASCII art para flujos y relaciones cuando ayude
6. **Prioriza lo importante**: Flujos críticos primero, detalles después
7. **Sé específico**: Menciona archivos y líneas concretas
8. **Respeta estándares del proyecto**: Usa CLAUDE.md/AGENTS.md como referencia
9. **Identifica patrones**: Nombra el patrón arquitectónico si lo reconoces
10. **Señala complejidad**: Indica áreas difíciles de entender
11. **Sugiere navegación**: Guía sobre por dónde empezar a leer
12. **Reconoce lo bueno**: Menciona prácticas positivas encontradas
