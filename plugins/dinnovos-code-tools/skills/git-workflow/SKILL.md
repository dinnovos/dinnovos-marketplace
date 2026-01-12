---
name: git-workflow
description: Guía de flujos de trabajo con Git. Se activa cuando Claude detecta operaciones de Git, commits, branches, o merge conflicts. Incluye convenciones de commits y estrategias de branching.
---

# Git Workflow Skill

Esta skill proporciona guías para flujos de trabajo con Git, convenciones de commits, y estrategias de branching.

## Cuándo Usar

Claude debe activar esta skill cuando:
- El usuario trabaja con Git (commits, branches, merges)
- Se necesita escribir mensajes de commit
- Hay conflictos de merge
- Se pregunta sobre estrategias de branching

## Conventional Commits

### Formato

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types

| Type | Descripción | Ejemplo |
|------|-------------|---------|
| `feat` | Nueva funcionalidad | `feat(auth): add login with Google` |
| `fix` | Corrección de bug | `fix(api): handle null response` |
| `docs` | Documentación | `docs(readme): update install instructions` |
| `style` | Formato (no afecta lógica) | `style: format with prettier` |
| `refactor` | Refactorización | `refactor(user): extract validation logic` |
| `perf` | Mejora de rendimiento | `perf(query): add index to users table` |
| `test` | Tests | `test(auth): add login unit tests` |
| `chore` | Tareas de mantenimiento | `chore(deps): update dependencies` |
| `ci` | CI/CD | `ci: add GitHub Actions workflow` |

### Reglas

1. **Imperativo**: "add" no "added" ni "adds"
2. **Minúsculas**: type y description en minúsculas
3. **Sin punto final**: en la primera línea
4. **Máximo 72 caracteres**: en la primera línea
5. **Body opcional**: para explicar el "qué" y "por qué"

### Breaking Changes

```
feat(api)!: change response format

BREAKING CHANGE: response.data is now response.result
```

## Estrategias de Branching

### Git Flow

```
main ─────────────────────────────────────►
       │                           │
       └── develop ────────────────┤
              │         │          │
              └─ feature/login ────┤
              │                    │
              └─ release/1.0 ──────┘
```

- `main`: Producción
- `develop`: Desarrollo
- `feature/*`: Nuevas funcionalidades
- `release/*`: Preparación de releases
- `hotfix/*`: Fixes urgentes en producción

### GitHub Flow (Simplificado)

```
main ─────────────────────────────────────►
       │              │
       └── feature ───┘ (PR + merge)
```

- `main`: Siempre deployable
- Feature branches cortas
- PR para todo cambio
- Deploy después de merge

### Trunk-Based

```
main ─────────────────────────────────────►
  │ │ │ │ │  (commits pequeños y frecuentes)
```

- Commits directos a main
- Feature flags para WIP
- CI/CD robusto requerido

## Resolución de Conflictos

### Proceso

1. **Identificar**: `git status`
2. **Abrir archivo**: Buscar marcadores `<<<<<<<`
3. **Resolver**: Elegir cambios o combinar
4. **Marcar resuelto**: `git add <archivo>`
5. **Continuar**: `git merge --continue` o `git rebase --continue`

### Marcadores

```
<<<<<<< HEAD
Tu código actual
=======
Código entrante
>>>>>>> branch-name
```

## Comandos Útiles

```bash
# Ver historial visual
git log --oneline --graph --all

# Deshacer último commit (mantener cambios)
git reset --soft HEAD~1

# Stash con nombre
git stash push -m "descripción"

# Cherry-pick
git cherry-pick <commit-hash>

# Rebase interactivo
git rebase -i HEAD~3

# Buscar en historial
git log -S "texto" --oneline
```

## Mejores Prácticas

1. **Commits atómicos**: Un cambio lógico por commit
2. **Branches cortas**: Merge frecuente, evitar divergencia
3. **Pull antes de push**: `git pull --rebase origin main`
4. **No reescribir historial público**: Solo en branches personales
5. **Revisar antes de commit**: `git diff --staged`
