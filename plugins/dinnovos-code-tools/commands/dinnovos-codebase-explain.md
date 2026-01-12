---
name: dinnovos-codebase-explain
description: Explica la arquitectura, estructura y flujos de un proyecto o módulo. Ideal para onboarding o entender código nuevo. Soporta múltiples lenguajes. Solo lectura.
model: opus
allowed-tools: ["Bash(read-only)", "Read", "Grep", "Glob"]
---

# Explicación de Codebase

Analiza y explica la arquitectura, estructura y flujos de un proyecto o módulo. **Solo lectura, no modifica nada. Soporta múltiples lenguajes.**

## Entrada del Usuario

El usuario puede especificar qué explicar de varias formas:

**Ruta exacta:**
- `/dinnovos-codebase-explain src/`
- `/dinnovos-codebase-explain src/modules/payments/`
- `/dinnovos-codebase-explain app/`

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

## Paso 1: Detectar Lenguaje y Stack

```bash
# Estructura general
find . -type d -maxdepth 3 | grep -v node_modules | grep -v vendor | grep -v target | grep -v __pycache__ | sort

# Detectar stack
ls -la
```

### Detección por archivos:

| Archivo | Stack |
|---------|-------|
| `package.json` | Node.js / JavaScript / TypeScript |
| `tsconfig.json` | TypeScript |
| `pyproject.toml` / `requirements.txt` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `composer.json` | PHP |
| `Gemfile` | Ruby |
| `pom.xml` / `build.gradle` | Java |
| `*.csproj` | C# / .NET |

### Frameworks comunes:

| Lenguaje | Archivos indicadores | Framework |
|----------|---------------------|-----------|
| JS/TS | `next.config.js` | Next.js |
| JS/TS | `vite.config.ts` | Vite |
| JS/TS | `angular.json` | Angular |
| Python | `manage.py` | Django |
| Python | `app.py` + Flask imports | Flask |
| Python | `main.py` + FastAPI imports | FastAPI |
| Go | `go.mod` + chi/gin/echo | Chi/Gin/Echo |
| Rust | `Cargo.toml` + actix/axum | Actix/Axum |
| PHP | `artisan` | Laravel |
| Ruby | `config.ru` + Rails | Rails |

---

## Paso 2: Leer Documentación

```bash
cat README.md 2>/dev/null
cat CLAUDE.md 2>/dev/null
cat AGENTS.md 2>/dev/null
cat ARCHITECTURE.md 2>/dev/null
cat docs/*.md 2>/dev/null
```

---

## Paso 3: Identificar Entry Points

### Por lenguaje/framework:

| Stack | Entry Points |
|-------|--------------|
| **Node.js** | `src/index.ts`, `src/main.ts`, `src/app.ts` |
| **React** | `src/App.tsx`, `src/main.tsx`, `pages/_app.tsx` |
| **Next.js** | `pages/`, `app/`, `next.config.js` |
| **Python** | `main.py`, `app.py`, `__main__.py` |
| **Django** | `manage.py`, `urls.py`, `settings.py` |
| **FastAPI** | `main.py`, `app/main.py` |
| **Go** | `main.go`, `cmd/*/main.go` |
| **Rust** | `src/main.rs`, `src/lib.rs` |
| **PHP/Laravel** | `public/index.php`, `routes/web.php` |
| **Ruby/Rails** | `config.ru`, `config/routes.rb` |
| **Java/Spring** | `*Application.java`, `pom.xml` |
| **C#/.NET** | `Program.cs`, `Startup.cs` |

```bash
# Buscar entry points
cat src/index.ts src/main.ts src/app.ts 2>/dev/null          # Node
cat src/App.tsx pages/_app.tsx app/layout.tsx 2>/dev/null    # React/Next
cat main.py app.py manage.py 2>/dev/null                      # Python
cat cmd/*/main.go main.go 2>/dev/null                         # Go
cat src/main.rs src/lib.rs 2>/dev/null                        # Rust
```

---

## Paso 4: Mapear Estructura

### Patrones comunes por stack:

#### JavaScript/TypeScript (Node/React):
```
src/
├── components/     # UI components
├── pages/          # Routes/Pages
├── hooks/          # Custom hooks
├── services/       # API/Business logic
├── utils/          # Helpers
├── types/          # TypeScript types
└── lib/            # Shared libraries
```

#### Python (Django/FastAPI):
```
app/
├── api/            # Endpoints
├── models/         # Database models
├── schemas/        # Pydantic schemas
├── services/       # Business logic
├── repositories/   # Data access
└── utils/          # Helpers

# Django específico
project/
├── apps/
│   └── myapp/
│       ├── models.py
│       ├── views.py
│       ├── urls.py
│       └── serializers.py
└── settings.py
```

#### Go:
```
cmd/
├── api/            # Entry points
└── worker/
internal/
├── handlers/       # HTTP handlers
├── services/       # Business logic
├── repositories/   # Data access
├── models/         # Domain models
└── pkg/            # Shared packages
pkg/                # Public packages
```

#### Rust:
```
src/
├── main.rs         # Entry point
├── lib.rs          # Library root
├── handlers/       # Request handlers
├── services/       # Business logic
├── models/         # Domain types
└── db/             # Database layer
```

#### PHP (Laravel):
```
app/
├── Http/
│   ├── Controllers/
│   └── Middleware/
├── Models/
├── Services/
└── Repositories/
routes/
├── web.php
└── api.php
```

#### Ruby (Rails):
```
app/
├── controllers/
├── models/
├── views/
├── services/
└── jobs/
config/
└── routes.rb
```

---

## Paso 5: Identificar Patrones Arquitectónicos

| Patrón | Indicadores |
|--------|-------------|
| **MVC** | controllers/, models/, views/ |
| **Clean Architecture** | domain/, usecases/, infrastructure/ |
| **Hexagonal** | ports/, adapters/, core/ |
| **DDD** | domain/, application/, infrastructure/ |
| **Microservices** | services/, múltiples go.mod/package.json |
| **Monorepo** | packages/, apps/, libs/ |

---

## Paso 6: Mapear Flujos de Datos

### Identificar:
- Routes/Endpoints
- Controllers/Handlers
- Services/Use Cases
- Repositories/DAOs
- Models/Entities
- External APIs

---

## Paso 7: Generar Informe

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
**Stack tecnológico:** [TypeScript + React + Node | Python + FastAPI | Go + Chi | etc.]
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

## 🚪 Entry Points

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
[Request] → [Handler] → [Service] → [Repository] → [DB]
                ↓
           [Response]
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
**ORM/Query Builder:** [Prisma | TypeORM | Mongoose | GORM | SQLAlchemy | etc.]

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

## 🎨 Capa de Presentación (si aplica)

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
2. **Detecta el stack**: Adapta explicación al lenguaje/framework del proyecto
3. **Interpreta inteligentemente**: Buscar archivos relacionados con lo que pida el usuario
4. **Confirma si hay ambigüedad**: Si no está claro qué explicar, pregunta
5. **Sé didáctico**: Explica para alguien que no conoce el proyecto
6. **Usa diagramas**: ASCII art para flujos y relaciones cuando ayude
7. **Prioriza lo importante**: Flujos críticos primero, detalles después
8. **Sé específico**: Menciona archivos y líneas concretas
9. **Respeta estándares del proyecto**: Usa CLAUDE.md/AGENTS.md como referencia
10. **Identifica patrones**: Nombra el patrón arquitectónico si lo reconoces
11. **Señala complejidad**: Indica áreas difíciles de entender
12. **Sugiere navegación**: Guía sobre por dónde empezar a leer
13. **Reconoce lo bueno**: Menciona prácticas positivas encontradas
