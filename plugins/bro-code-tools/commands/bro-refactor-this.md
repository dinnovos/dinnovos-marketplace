---
name: bro-refactor-this
description: Analiza código en busca de duplicaciones, lógica similar y oportunidades de refactorización. Soporta múltiples lenguajes. Solo lectura.
model: opus
allowed-tools: ["Bash(read-only)", "Read", "Grep", "Glob"]
---

# Análisis de Refactorización

Analiza el código en busca de duplicaciones y oportunidades de refactorización. **Solo lectura, no modifica nada. Soporta múltiples lenguajes.**

## Entrada del Usuario

El usuario puede especificar qué analizar de varias formas:

**Ruta exacta:**
- `/bro-refactor-this src/components/`
- `/bro-refactor-this src/services/userService.ts`
- `/bro-refactor-this app/services/`

**Lenguaje natural (ejemplos ilustrativos):**
- `/bro-refactor-this analiza los componentes de <área>`
- `/bro-refactor-this busca duplicados en <módulo>`
- `/bro-refactor-this revisa oportunidades en los servicios de <funcionalidad>`
- `/bro-refactor-this analiza todo lo relacionado con <tema>`

**Sin argumentos:**
- `/bro-refactor-this` → analiza todo el proyecto

> **Nota:** Los términos como "UI", "autenticación", "pagos" son solo ejemplos. Interpreta lo que el usuario solicite y busca los archivos correspondientes en el proyecto.

---

## Paso 1: Interpretar la Solicitud

### Si es ruta exacta:
Usar directamente.

### Si es lenguaje natural:
Buscar archivos que coincidan con la descripción:

```bash
# Explorar estructura del proyecto
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.go" -o -name "*.rs" -o -name "*.php" -o -name "*.rb" -o -name "*.java" -o -name "*.cs" \) \
  ! -path "*/node_modules/*" ! -path "*/vendor/*" ! -path "*/target/*" ! -path "*/__pycache__/*" ! -path "*/dist/*" ! -path "*/.git/*"

# Buscar por nombre relacionado (reemplaza <término> con lo que pidió el usuario)
find . -type f -iname "*<término>*" | grep -v node_modules
find . -type d -iname "*<término>*" | grep -v node_modules

# Buscar contenido relacionado
grep -ril "<término>" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.py" --include="*.go" | grep -v node_modules | head -30
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

**Límite:** Máximo 100 archivos. Si hay más, pide acotar o prioriza por tamaño.

---

## Paso 2: Contexto del Proyecto

Busca y lee archivos de configuración y estándares:

```bash
# Estándares y guías del proyecto
cat CLAUDE.md 2>/dev/null
cat AGENTS.md 2>/dev/null
cat .cursor/rules.md 2>/dev/null

# Detectar stack
cat package.json pyproject.toml go.mod Cargo.toml composer.json Gemfile 2>/dev/null

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

#### Ejemplos por lenguaje:

**JavaScript/TypeScript:**
```typescript
// ❌ Duplicado en UserCard.tsx y AdminCard.tsx
const formatName = (user) => `${user.first} ${user.last}`
// ✅ Extraer a utils/formatters.ts
export const formatName = (user: User) => `${user.first} ${user.last}`
```

**Python:**
```python
# ❌ Duplicado en user_service.py y admin_service.py
def format_name(user):
    return f"{user.first} {user.last}"
# ✅ Extraer a utils/formatters.py
```

**Go:**
```go
// ❌ Duplicado en handlers/
func formatName(u User) string {
    return u.First + " " + u.Last
}
// ✅ Extraer a pkg/formatters/
```

---

### 🧩 2. Lógica Similar

Funciones o bloques que hacen cosas parecidas con pequeñas variaciones.

**Buscar:**
- Mismo patrón con diferentes datos
- Validaciones similares
- Transformaciones de datos análogas
- Handlers con estructura repetida

#### Ejemplos por lenguaje:

**JavaScript/TypeScript:**
```typescript
// ❌ Similar
function validateUser(d) {
  if (!d.email) return {error: 'Email required'}
  if (!d.pass) return {error: 'Pass required'}
}
function validateAdmin(d) {
  if (!d.email) return {error: 'Email required'}
  if (!d.pass) return {error: 'Pass required'}
  if (!d.role) return {error: 'Role required'}
}
// ✅ Unificado
function validate(data, fields) {
  for (const f of fields) {
    if (!data[f]) return {error: `${f} required`}
  }
}
```

**Python:**
```python
# ❌ Similar
def get_user_by_email(email): return db.query(User).filter_by(email=email).first()
def get_user_by_id(id): return db.query(User).filter_by(id=id).first()
# ✅ Unificado
def get_user_by(**kwargs): return db.query(User).filter_by(**kwargs).first()
```

**Go:**
```go
// ❌ Similar handlers
func GetUserHandler(w http.ResponseWriter, r *http.Request) { /*...*/ }
func GetProductHandler(w http.ResponseWriter, r *http.Request) { /*...*/ }
// ✅ Generic handler (Go 1.18+)
func MakeGetHandler[T any](svc Service[T]) http.HandlerFunc { /*...*/ }
```

---

### 📦 3. Funciones Repetidas

Funciones con el mismo propósito en diferentes archivos.

**Buscar:**
- Utilidades duplicadas (formatDate, capitalize, slugify, etc.)
- Helpers repetidos
- Funciones de validación similares

| Utilidad | Buscar en |
|----------|-----------|
| formatDate | Múltiples archivos |
| capitalize | utils/, helpers/ |
| slugify | varios servicios |
| validateEmail | formularios |

---

### 🏗️ 4. Clases/Componentes Similares

Clases o componentes con estructura o comportamiento parecido.

#### Ejemplos por lenguaje:

**React:**
```tsx
// ❌ Componentes similares
const UserCard = ({user}) => <Card><Avatar/><Name/></Card>
const AdminCard = ({admin}) => <Card><Avatar/><Name/><Badge/></Card>
// ✅ Componente base
const PersonCard = ({person, badge}) => <Card><Avatar/><Name/>{badge}</Card>
```

**Python:**
```python
# ❌ Repositorios duplicados
class UserRepo:
    def find_all(self): return db.query(User).all()
class ProductRepo:
    def find_all(self): return db.query(Product).all()
# ✅ Base genérica
class BaseRepo(Generic[T]):
    def find_all(self) -> List[T]: return db.query(self.model).all()
```

**Go:**
```go
// ❌ Services similares
type UserService struct { db *DB }
func (s *UserService) GetAll() []User { /*...*/ }
type ProductService struct { db *DB }
func (s *ProductService) GetAll() []Product { /*...*/ }
// ✅ Interface común
type Repository[T any] interface {
    GetAll() []T
}
```

---

### 🔢 5. Constantes y Magic Numbers

Valores hardcodeados repetidos.

**Buscar:**
- Números mágicos repetidos (timeouts, límites, etc.)
- Strings duplicados (URLs, mensajes, keys)
- Configuraciones dispersas

| Tipo | JS/TS | Python | Go | Rust |
|------|-------|--------|-----|------|
| URL | `const API = ''` | `API = ''` | `const API = ""` | `const API: &str` |
| Timeout | `TIMEOUT = 30000` | `TIMEOUT = 30` | `Timeout = 30*time.Second` | `TIMEOUT: u64 = 30` |

---

### 📝 6. Inconsistencias de Naming

Variables que representan lo mismo con nombres diferentes.

**Buscar:**
- `user` vs `currentUser` vs `loggedUser` para lo mismo
- `isLoading` vs `loading` vs `isLoad`
- Inconsistencias en convenciones (camelCase vs snake_case)

| Aspecto | JS/TS | Python | Go | Rust |
|---------|-------|--------|-----|------|
| Variables | camelCase | snake_case | camelCase | snake_case |
| Funciones | camelCase | snake_case | PascalCase | snake_case |
| Constantes | UPPER_SNAKE | UPPER_SNAKE | PascalCase | UPPER_SNAKE |

---

### 🎯 7. Patrones Repetidos

Estructuras de código que se repiten con el mismo propósito.

#### Ejemplos por lenguaje:

**JavaScript/TypeScript:**
```typescript
// ❌ Repetido: fetch + loading + error
const [loading, setLoading] = useState(false)
const [data, setData] = useState(null)
useEffect(() => { fetch()... }, [])
// ✅ Custom hook o React Query
const { data, loading } = useFetch(url)
```

**Python:**
```python
# ❌ Repetido: try + log + raise
try: result = operation()
except Exception as e:
    logger.error(e)
    raise
# ✅ Decorator
@log_errors
def operation(): ...
```

**Go:**
```go
// ❌ Repetido: error wrapping
if err != nil {
    log.Printf("error: %v", err)
    return fmt.Errorf("failed: %w", err)
}
// ✅ Helper
if err != nil {
    return errors.Wrap(err, "context")
}
```

---

## Paso 5: Generar Informe

**Responde directamente en el chat:**

```markdown
# 📊 INFORME DE ANÁLISIS BRO REFACTOR

**Fecha:** [fecha actual]
**Alcance:** `[ruta, descripción o "proyecto completo"]`
**Lenguaje(s):** [detectados]
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
2. **Detecta el lenguaje**: Adapta ejemplos y sugerencias al lenguaje del proyecto
3. **Interpreta inteligentemente**: Buscar archivos relacionados con lo que pida el usuario
4. **Confirma si hay ambigüedad**: Si hay múltiples coincidencias, pregunta
5. **Sé exhaustivo**: Lee todos los archivos del alcance
6. **Sé específico**: Indica archivos y líneas exactas
7. **Prioriza por impacto**: Lo que más se repite primero
8. **Sugiere soluciones concretas**: No solo señales problemas, propón extracciones
9. **Ignora falsos positivos**: Código similar por necesidad (tests, migrations) no cuenta
10. **Considera el contexto**: A veces la duplicación es intencional
11. **Respeta estándares del proyecto**: Usa CLAUDE.md/AGENTS.md como referencia
12. **Reconoce lo bueno**: Menciona abstracciones bien hechas
13. **Cuantifica el impacto**: "X líneas reducibles" ayuda a priorizar
