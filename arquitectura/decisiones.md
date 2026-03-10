# 🏗️ Architecture Decisions

> Decisiones de arquitectura para proyectos web.
> Diseñado para proyectos personales, universitarios y prototipos rápidos.
> Este documento ayuda a humanos y agentes de IA a entender **qué tecnologías usar y por qué**.

---

# Frontend

## ¿Por qué React + Next.js y no Angular para proyectos propios?

**Next.js ventajas**

* SSR y SSG integrados
* Mejor SEO
* Mejor performance inicial
* Deploy muy fácil en Vercel
* Gran ecosistema

**Angular ventajas**

* Arquitectura muy estructurada
* Mejor para equipos grandes
* Muy usado en entornos enterprise

**Regla**

```
Next.js → proyectos propios
Angular → cuando la cátedra o empresa lo exige
```

---

## Styling

### ¿Por qué TailwindCSS?

* Desarrollo rápido
* No se crean archivos CSS enormes
* Consistencia visual
* Muy buen soporte con React

Alternativas:

* **CSS Modules** → para estilos aislados
* **SCSS** → cuando el proyecto es grande

---

## Estado global

### ¿Por qué Zustand y no Redux?

Redux tiene mucho boilerplate:

* actions
* reducers
* dispatch
* selectors

Zustand:

* mucho más simple
* menos código
* ideal para proyectos medianos

**Regla**

```
Zustand → proyectos personales
Redux → cuando el proyecto lo exige
```

---

## Fetching de datos

### ¿Por qué React Query?

Ventajas:

* caché automático
* revalidación automática
* manejo de loading / error
* evita duplicar lógica de fetching

Alternativas:

* **SWR** → similar pero menos features
* **fetch manual** → solo para casos simples

---

# Backend

Stack principal esperado:

```
TypeScript
Node.js
NestJS
```

Razón: ecosistema consistente con el frontend.

---

## ¿Por qué NestJS?

Ventajas:

* arquitectura modular
* inspirado en Angular
* soporte nativo para TypeScript
* testing integrado
* estructura clara

Esto evita proyectos Node desorganizados.

---

## Patrones de capas

Arquitectura preferida:

```
Controller
Service
DTO
DAO / Repository
Model
```

### Flujo

```
Controller
   ↓
Service
   ↓
DAO / Repository
   ↓
Database
```

---

### DTO (Data Transfer Object)

Se usa para:

* validar datos de entrada
* tipado fuerte
* evitar exponer modelos internos

---

### DAO / Repository

Responsable de:

* queries a la base de datos
* encapsular acceso a datos
* desacoplar la lógica del ORM

---

# Base de Datos

## ¿Por qué PostgreSQL?

Ventajas:

* JSONB
* mejores queries complejas
* CTEs
* window functions
* muy estable

Alternativas:

**MySQL**

* perfectamente válido
* más simple

**MongoDB**

Solo usar cuando:

```
el schema cambia constantemente
```

No usar Mongo como excusa para **no diseñar el schema**.

---

## ORM

### ¿Por qué Prisma?

Ventajas:

* autocompletado real
* type safety
* migraciones simples
* excelente integración con TypeScript

Alternativas:

**TypeORM**

* más antiguo
* más bugs en relaciones complejas

**Sequelize**

* menos enfocado en TypeScript

---

# API Design

## REST vs GraphQL

### REST (default)

Usar REST cuando:

* proyecto universitario
* API simple
* fácil de documentar
* fácil de explicar

---

### GraphQL

Usar cuando:

* múltiples clientes
* apps complejas
* diferentes necesidades de datos

Ejemplo:

```
web
mobile
admin panel
```

---

# Arquitectura del sistema

## Arquitectura por capas (default)

La mayoría de proyectos usarán:

```
MVC + DTO + DAO
```

Ventajas:

* simple
* fácil de mantener
* fácil de explicar
* suficiente para la mayoría de proyectos

---

## Monolito modular

Usar cuando:

* proyecto pequeño o mediano
* un solo equipo
* despliegue simple

Estructura típica:

```
auth
users
products
orders
```

Cada módulo tiene:

```
controller
service
repository
dto
```

---

## Microservicios (solo cuando sea necesario)

Usar microservicios solo si:

```
sistema muy grande
equipos separados
alta escalabilidad
```

Problemas que introducen:

* complejidad operacional
* comunicación entre servicios
* observabilidad
* despliegue complejo

---

### Ejemplo de microservicios

```
API Gateway
Auth Service
User Service
Notification Service
Payment Service
```

Comunicación:

```
HTTP
Message Queue
```

---

# Autenticación

### JWT stateless

Ventajas:

* simple
* escalable
* no necesita sesiones en DB

---

### Configuración recomendada

```
Access token → 15m - 1h
Refresh token → 7 - 30 días
```

---

### Seguridad del refresh token

Guardar en:

```
httpOnly cookie
```

Nunca en:

```
localStorage
```

Razón: **XSS**

---

# Índices de Base de Datos

Crear índices en:

```
foreign keys
email
username
created_at
```

No crear índices en:

```
campos que cambian constantemente
```

Ejemplo:

```
balance
contador
```

---

# Migraciones

Reglas:

```
siempre usar el ORM
nunca modificar DB manualmente en dev
```

En producción:

```
no borrar columnas directamente
deprecar primero
migrar datos
eliminar después
```

---

# Despliegue

Para proyectos personales:

```
Frontend → Vercel
Backend → Vercel / Railway / VPS
Database → Supabase / Neon
```

Razones:

* planes gratuitos
* fácil integración
* deploy rápido

---

# Docker (opcional)

Docker se usa cuando:

```
proyecto necesita entorno reproducible
trabajo en equipo
CI/CD
```

Para proyectos simples **no es obligatorio**.

---

# Seguridad

Las políticas completas se encuentran en:

```
seguridad.md
```

Pero reglas básicas:

```
validar siempre input
usar HTTPS
rate limiting
no exponer errores internos
```

---

# Tradeoffs

Este documento prioriza:

```
simplicidad
mantenibilidad
aprendizaje
```

No siempre:

```
máxima escalabilidad
arquitecturas distribuidas
```

Las decisiones pueden cambiar dependiendo del proyecto.

---

# Project Boot Rules (para agentes de IA)

Reglas para iniciar proyectos automáticamente.

```
si el proyecto es web → usar Next.js
si hay API → usar NestJS
si hay base de datos → usar PostgreSQL + Prisma
si hay autenticación → usar JWT + refresh tokens
si hay estado global → usar Zustand
si hay fetching de datos → usar React Query
```

---

# Objetivo del documento

Este documento busca:

* reducir decisiones innecesarias
* mantener consistencia entre proyectos
* facilitar el trabajo con agentes de IA
* documentar por qué se eligió cada tecnología
