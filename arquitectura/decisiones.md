# 🏗️ Decisiones de Arquitectura

> Por qué elegí cada tecnología — útil para defensas y para no olvidarlo

## Frontend

### ¿Por qué React + Next.js y no Angular para proyectos propios?
- Next.js tiene SSR/SSG integrado (mejor SEO, mejor performance inicial)
- Ecosistema más amplio, más recursos de aprendizaje
- Angular es mejor para equipos grandes con contratos enterprise (es lo que piden en Java-land)
- **Regla:** Angular cuando la cátedra lo exige o el equipo ya lo usa; Next.js para proyectos propios

### ¿Por qué Zustand y no Redux?
- Redux tiene demasiado boilerplate para proyectos universitarios (actions, reducers, selectors...)
- Zustand hace lo mismo en 10% del código
- Si en el trabajo piden Redux, el concepto es el mismo, solo más verboso

### ¿Por qué React Query y no SWR o fetch manual?
- Caché automático, revalidación, estados de loading/error out of the box
- Evita duplicar lógica de fetching en múltiples componentes
- SWR es similar pero React Query tiene mejor soporte para mutaciones

## Backend

### ¿Por qué Prisma y no TypeORM o Sequelize?
- Prisma tiene auto-completado real y type safety total
- Migraciones más simples con `prisma migrate dev`
- TypeORM tiene bugs históricos con relaciones complejas
- Sequelize es más legacy, menos TypeScript-friendly

### ¿Por qué FastAPI y no Flask o Django?
- FastAPI es async nativo (mejor performance con I/O)
- Documentación automática con Swagger (útil para defender proyectos)
- Validación integrada con Pydantic
- Flask: bueno para proyectos muy simples, le falta estructura para proyectos medianos
- Django: excelente para proyectos muy grandes o con admin panel, overkill para APIs puras

### ¿Por qué PostgreSQL y no MySQL o MongoDB?
- PostgreSQL tiene JSONB (puedo guardar docs cuando necesito)
- Mejor soporte para queries complejas (window functions, CTEs)
- MongoDB: usar solo cuando el schema es genuinamente variable (no como excuse para no diseñar)
- MySQL: perfectamente válido, pero PostgreSQL tiene más features

## Patrones de arquitectura

### REST vs GraphQL
- **REST** para proyectos universitarios (más simple, más fácil de explicar)
- **GraphQL** cuando hay múltiples clientes con necesidades muy distintas de datos

### Monolito vs Microservicios
- **Monolito modular** para todo proyecto universitario
- Microservicios solo si la materia lo exige explícitamente
- Razón: los microservicios tienen overhead operacional enorme (Docker, service discovery, distributed tracing...)

### Autenticación
- **JWT stateless** para APIs (fácil de implementar, no necesita DB para validar)
- Tokens de acceso: 15min-1h
- Refresh tokens: 7-30 días
- Guardar refresh token en httpOnly cookie (no localStorage — XSS)

## Decisiones de base de datos

### Cuándo usar índices
- Siempre en FKs (relaciones)
- En campos que se buscan frecuentemente (email, username)
- En campos por los que se ordena (created_at)
- NO en campos que cambian muy seguido (balance, contador)

### Migraciones
- Siempre usar el ORM para migraciones, nunca SQL manual en dev
- Nunca borrar columnas directamente en prod — deprecar primero
