---
name: dinnovos-auto-commit
description: Analiza los cambios pendientes y crea un commit con mensaje descriptivo siguiendo Conventional Commits
allowed-tools: ["Bash", "Read", "Grep"]
model: haiku
---

# Auto Commit

Analiza los archivos pendientes de commit, genera un mensaje descriptivo y ejecuta el commit.

## Paso 1: Verificar estado del repositorio

```bash
git status --short
```

### Si no hay cambios

Si el comando no devuelve nada, responde:

```
✅ No hay cambios pendientes

El directorio de trabajo está limpio. No hay nada que commitear.
```

Detente aquí si no hay cambios.

## Paso 2: Preparar archivos

Verifica si hay archivos staged:

```bash
git diff --cached --name-only
```

Si no hay archivos staged, agrega todos los cambios:

```bash
git add -A
```

## Paso 3: Analizar los cambios

Obtén el diff completo de lo que se va a commitear:

```bash
git diff --cached --stat
git diff --cached
```

Lee y analiza:
- Qué archivos fueron modificados/agregados/eliminados
- Qué tipo de cambios son (feature, fix, refactor, docs, etc.)
- Cuál es el propósito principal del cambio

## Paso 4: Generar mensaje de commit

Genera un mensaje siguiendo **Conventional Commits**:

### Formato

```
<tipo>(<alcance>): <descripción corta>

<cuerpo opcional - qué y por qué>

<footer opcional - breaking changes, issues>
```

### Tipos disponibles

| Tipo | Cuándo usarlo |
|------|---------------|
| `feat` | Nueva funcionalidad |
| `fix` | Corrección de bug |
| `refactor` | Cambio de código que no agrega feature ni corrige bug |
| `docs` | Cambios en documentación |
| `style` | Formateo, punto y coma faltantes, etc. (no afecta lógica) |
| `test` | Agregar o modificar tests |
| `chore` | Tareas de mantenimiento, dependencias, config |
| `perf` | Mejoras de rendimiento |
| `ci` | Cambios en CI/CD |
| `build` | Cambios en build system o dependencias externas |
| `revert` | Revertir commit anterior |

### Reglas del mensaje

1. **Descripción corta**: máximo 50 caracteres, imperativo, sin punto final
2. **Alcance**: opcional, indica el módulo/componente afectado
3. **Cuerpo**: opcional, explica qué y por qué (no cómo)
4. **En español o inglés**: según el idioma predominante en el proyecto

### Ejemplos

```bash
# Simple
git commit -m "feat(auth): agregar login con Google"

# Con cuerpo
git commit -m "fix(api): corregir timeout en peticiones largas

El timeout de 30s era insuficiente para uploads grandes.
Aumentado a 120s para archivos hasta 100MB."

# Breaking change
git commit -m "refactor(db)!: migrar de MySQL a PostgreSQL

BREAKING CHANGE: requiere nueva configuración de conexión"
```

## Paso 5: Ejecutar el commit

Una vez analizado el diff y generado el mensaje apropiado:

```bash
git commit -m "<mensaje generado>"
```

## Paso 6: Confirmar

Muestra el resultado:

```
✅ Commit creado exitosamente

**Hash:** [hash corto]
**Mensaje:** [mensaje del commit]

**Archivos incluidos:**
- [lista de archivos]

**Siguiente paso:** `git push` para subir los cambios
```

## Reglas

1. **Analiza TODOS los cambios** antes de decidir el tipo
2. **Un commit = un propósito** — Si hay cambios muy diversos, sugiere dividirlos
3. **Mensaje claro** — Alguien debe entender qué se hizo sin ver el código
4. **No uses mensajes genéricos** — Evita "update", "fix", "changes"
5. **Detecta el idioma** — Usa el idioma predominante en commits anteriores
6. **Si hay muchos cambios diversos**, pregunta al usuario si quiere:
   - Un solo commit general
   - Dividir en múltiples commits
7. **NO agregues "Co-Authored-By"** — El mensaje debe ser limpio, sin líneas de co-autoría de IA

## Verificación previa al commit

Antes de ejecutar el commit, verifica:

```bash
# Ver si hay linter configurado
npm run lint 2>/dev/null || yarn lint 2>/dev/null || true
```

Si el linter falla, informa al usuario pero no detengas el commit (es su decisión).

## Casos especiales

### Si detectas archivos sensibles

Si ves archivos como `.env`, `*.key`, `credentials.*`, `*secret*`:

```
⚠️ ADVERTENCIA: Detecté archivos potencialmente sensibles:
- .env.local
- config/secrets.json

¿Estás seguro de que quieres incluirlos en el commit?
Estos archivos normalmente deberían estar en .gitignore
```

Espera confirmación antes de continuar.

### Si hay cambios muy grandes

Si hay más de 500 líneas cambiadas o más de 20 archivos:

```
ℹ️ Este commit incluye muchos cambios:
- X archivos modificados
- +Y líneas / -Z líneas

¿Prefieres dividirlo en commits más pequeños?
```
