# ⚙️ Agente Backend — Node.js + Express

> Leé este archivo cuando trabajes en APIs REST con Node.js

## Stack exacto
- **Node.js** 20 LTS
- **Express** 4.x con TypeScript
- **Prisma** ORM (no Sequelize, no TypeORM — Prisma tiene mejor DX)
- **PostgreSQL** como base de datos principal
- **JWT** (jsonwebtoken) para autenticación
- **Zod** para validación de schemas
- **bcrypt** para hashing de contraseñas
- **Winston** para logging
- **Jest + Supertest** para tests

## Estructura de carpetas
```
src/
├── config/               # Variables de entorno, configuración DB
├── controllers/          # Reciben req/res, delegan a services
├── services/             # Lógica de negocio pura
├── repositories/         # Acceso a DB via Prisma
├── middlewares/          # Auth, validación, error handler
├── routes/               # Definición de rutas
├── schemas/              # Schemas Zod para validación
├── types/                # Interfaces y tipos propios
├── utils/                # Helpers reutilizables
└── server.ts             # Entry point
prisma/
├── schema.prisma         # Modelo de datos
└── migrations/
```

## Patrón Controller → Service → Repository
```typescript
// controllers/user.controller.ts
export const getUser = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const user = await userService.findById(req.params.id)
    res.json({ success: true, data: user })
  } catch (error) {
    next(error) // siempre pasar errores al error handler global
  }
}

// services/user.service.ts
export const findById = async (id: string): Promise<User> => {
  const user = await userRepository.findById(id)
  if (!user) throw new AppError('Usuario no encontrado', 404)
  return user
}

// repositories/user.repository.ts
export const findById = (id: string) =>
  prisma.user.findUnique({ where: { id } })
```

## Middleware de autenticación JWT
```typescript
// middlewares/auth.middleware.ts
export const authenticate = async (req: AuthRequest, res: Response, next: NextFunction) => {
  const token = req.headers.authorization?.split(' ')[1]
  if (!token) return res.status(401).json({ message: 'Token requerido' })

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JwtPayload
    req.user = payload
    next()
  } catch {
    res.status(401).json({ message: 'Token inválido' })
  }
}
```

## Validación con Zod
```typescript
// schemas/user.schema.ts
export const createUserSchema = z.object({
  body: z.object({
    email: z.string().email('Email inválido'),
    password: z.string().min(8, 'Mínimo 8 caracteres'),
    name: z.string().min(2).max(50),
  })
})

// middleware que valida automáticamente
export const validate = (schema: ZodSchema) =>
  (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse({ body: req.body, params: req.params })
    if (!result.success) return res.status(400).json({ errors: result.error.flatten() })
    next()
  }
```

## Error handler global (siempre al final del app)
```typescript
// middlewares/errorHandler.ts
export const errorHandler = (err: Error, req: Request, res: Response, next: NextFunction) => {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({ success: false, message: err.message })
  }
  logger.error(err)
  res.status(500).json({ success: false, message: 'Error interno del servidor' })
}
```

## Estructura de respuesta API (siempre consistente)
```json
// Éxito
{ "success": true, "data": { ... }, "meta": { "total": 100, "page": 1 } }

// Error
{ "success": false, "message": "Descripción del error", "errors": [ ... ] }
```

## Variables de entorno (.env)
```env
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://user:pass@localhost:5432/dbname
JWT_SECRET=supersecreto_cambiame_en_produccion
JWT_EXPIRES_IN=7d
```

## Lo que SIEMPRE debo recordarme
- NUNCA lógica de negocio en controllers
- NUNCA acceso a DB directo en controllers o services (usar repositories)
- Siempre `try/catch` o pasar a `next(error)` en controllers async
- Hashear contraseñas ANTES de guardar, NUNCA guardar en texto plano
- No exponer stack traces en producción
- Rate limiting en rutas de auth para evitar brute force
