---
name: code-standards
description: Guía de estándares de código y convenciones. Se activa automáticamente cuando Claude detecta necesidad de aplicar estándares consistentes. Soporta múltiples lenguajes.
---

# Code Standards Skill

Esta skill proporciona guías de estándares y convenciones de código para múltiples lenguajes.

## Cuándo Usar

Claude debe activar esta skill cuando:
- El usuario pide revisar estándares de código
- Se está escribiendo código nuevo
- Se detectan inconsistencias en naming o formato
- El usuario pregunta sobre convenciones

## Estándares por Lenguaje

### JavaScript/TypeScript

**Naming:**
- Variables y funciones: `camelCase`
- Clases y tipos: `PascalCase`
- Constantes: `UPPER_SNAKE_CASE`
- Archivos de componentes: `PascalCase.tsx`
- Archivos utilitarios: `camelCase.ts`

**Estructura:**
```typescript
// 1. Imports externos
import React from 'react';

// 2. Imports internos
import { Button } from '@/components';

// 3. Types
interface Props { ... }

// 4. Constantes
const DEFAULT_VALUE = 10;

// 5. Componente/Función principal
export function Component() { ... }

// 6. Funciones auxiliares
function helper() { ... }
```

### Python

**Naming (PEP 8):**
- Variables y funciones: `snake_case`
- Clases: `PascalCase`
- Constantes: `UPPER_SNAKE_CASE`
- Módulos: `snake_case.py`
- Privados: `_prefijo_underscore`

**Estructura:**
```python
"""Docstring del módulo."""

# 1. Imports estándar
import os

# 2. Imports terceros
import requests

# 3. Imports locales
from .utils import helper

# 4. Constantes
DEFAULT_TIMEOUT = 30

# 5. Clases
class MyClass:
    """Docstring de clase."""
    pass

# 6. Funciones
def my_function():
    """Docstring de función."""
    pass
```

### Go

**Naming:**
- Exportados: `PascalCase`
- No exportados: `camelCase`
- Interfaces: `NombreVerbo` (ej: `Reader`, `Writer`)
- Archivos: `snake_case.go`

**Estructura:**
```go
package main

import (
    // Estándar
    "fmt"
    
    // Terceros
    "github.com/pkg/errors"
    
    // Internos
    "myapp/internal/config"
)

const DefaultTimeout = 30

type Service struct { ... }

func NewService() *Service { ... }

func (s *Service) Method() { ... }
```

### Rust

**Naming:**
- Variables y funciones: `snake_case`
- Tipos y traits: `PascalCase`
- Constantes: `UPPER_SNAKE_CASE`
- Módulos: `snake_case`

**Estructura:**
```rust
//! Documentación del módulo

use std::io;
use external_crate::Thing;
use crate::internal::helper;

const MAX_SIZE: usize = 100;

pub struct MyStruct { ... }

impl MyStruct {
    pub fn new() -> Self { ... }
}

fn private_helper() { ... }
```

## Reglas Universales

1. **Consistencia**: Seguir el estilo existente del proyecto
2. **Claridad**: Nombres descriptivos > abreviaciones
3. **Documentación**: Funciones públicas documentadas
4. **Imports**: Ordenados y agrupados
5. **Líneas**: Máximo 80-120 caracteres
