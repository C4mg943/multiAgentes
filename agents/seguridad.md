# 🔒 Agente Seguridad — Buenas prácticas para todos los stacks

> Leé este archivo cuando vayas a desplegar un proyecto o cuando el profe pida consideraciones de seguridad

## Regla de oro
**Nunca confíes en el cliente.** Todo lo que llega del frontend puede ser manipulado.
Validá, sanitizá y autenticá siempre del lado del servidor.

---

## 1. Variables de entorno — lo más básico

```bash
# .env (NUNCA subir al repo)
DATABASE_URL=postgresql://user:pass@localhost:5432/dbname
JWT_SECRET=cadena_aleatoria_larga_minimo_32_caracteres
JWT_EXPIRES_IN=15m
REFRESH_TOKEN_EXPIRES_IN=7d
CORS_ORIGIN=https://tudominio.com

# .env.example (SÍ subir al repo, sin valores reales)
DATABASE_URL=
JWT_SECRET=
JWT_EXPIRES_IN=
CORS_ORIGIN=
```

```bash
# .gitignore — siempre incluir
.env
.env.local
.env.production
*.key
*.pem
```

**Generar un JWT_SECRET seguro:**
```bash
# En terminal
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
# o en Python
python -c "import secrets; print(secrets.token_hex(64))"
```

---

## 2. Autenticación JWT — implementación correcta

### Flujo correcto con access + refresh tokens
```
Login → access token (15min) + refresh token (7 días, httpOnly cookie)
       ↓
Requests → Authorization: Bearer <access_token>
       ↓
Access token expira → usar refresh token para obtener nuevo access token
       ↓
Logout → invalidar refresh token en DB
```

### Dónde guardar los tokens (frontend)
```typescript
// ✅ Refresh token: httpOnly cookie (no accesible desde JS → protege contra XSS)
// El backend lo setea así:
res.cookie('refreshToken', token, {
  httpOnly: true,
  secure: true,           // solo HTTPS
  sameSite: 'strict',     // protege contra CSRF
  maxAge: 7 * 24 * 60 * 60 * 1000  // 7 días en ms
})

// ✅ Access token: memoria (variable en el store de Zustand/Redux)
// Dura poco (15min), si se pierde al recargar se renueva con el refresh token

// ❌ NUNCA: localStorage para tokens (vulnerable a XSS)
// ❌ NUNCA: sessionStorage para refresh tokens
```

### Verificación JWT (Node.js)
```typescript
// middlewares/auth.middleware.ts
export const authenticate = async (req: AuthRequest, res: Response, next: NextFunction) => {
  const authHeader = req.headers.authorization
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ success: false, message: 'Token requerido' })
  }

  const token = authHeader.split(' ')[1]
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JwtPayload
    req.user = payload
    next()
  } catch (err) {
    if (err instanceof jwt.TokenExpiredError) {
      return res.status(401).json({ success: false, message: 'Token expirado' })
    }
    return res.status(401).json({ success: false, message: 'Token inválido' })
  }
}
```

### Verificación JWT (Python/FastAPI)
```python
# core/security.py
from jose import JWTError, jwt
from fastapi import HTTPException, status

def verify_token(token: str) -> dict:
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
        if payload.get("sub") is None:
            raise HTTPException(status_code=401, detail="Token inválido")
        return payload
    except JWTError:
        raise HTTPException(status_code=401, detail="Token inválido o expirado")
```

---

## 3. Contraseñas — hashing correcto

```typescript
// Node.js — bcrypt factor 12
import bcrypt from 'bcrypt'
const SALT_ROUNDS = 12

export const hashPassword = (plain: string) => bcrypt.hash(plain, SALT_ROUNDS)
export const verifyPassword = (plain: string, hash: string) => bcrypt.compare(plain, hash)

// ❌ NUNCA guardar contraseñas en texto plano
// ❌ NUNCA usar MD5 o SHA1 para contraseñas (son rápidos → fácil de crackear)
// ✅ bcrypt, argon2 o scrypt (son lentos a propósito)
```

```python
# Python — passlib con bcrypt
from passlib.context import CryptContext
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(plain: str) -> str:
    return pwd_context.hash(plain)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)
```

---

## 4. Validación de inputs — nunca confiar en el cliente

```typescript
// Node.js — Zod en todos los endpoints
const loginSchema = z.object({
  body: z.object({
    email: z.string().email('Email inválido'),
    password: z.string().min(1, 'Contraseña requerida'),
  })
})

// ❌ NUNCA hacer queries con input directo del usuario
const user = await db.query(`SELECT * FROM users WHERE email = '${req.body.email}'`) // SQL injection!

// ✅ Siempre usar ORM o queries parametrizadas
const user = await prisma.user.findUnique({ where: { email: req.body.email } })
```

```python
# Python — Pydantic valida automáticamente en FastAPI
class LoginRequest(BaseModel):
    email: EmailStr          # valida formato email automáticamente
    password: str = Field(min_length=1)
```

---

## 5. CORS — configuración correcta

```typescript
// Node.js/Express
import cors from 'cors'

app.use(cors({
  origin: process.env.CORS_ORIGIN,   // nunca '*' en producción
  credentials: true,                  // necesario para cookies
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}))
```

```python
# FastAPI
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[settings.CORS_ORIGIN],  # nunca ["*"] en producción
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE", "PATCH"],
    allow_headers=["Content-Type", "Authorization"],
)
```

---

## 6. Rate limiting — proteger contra brute force

```typescript
// Node.js — express-rate-limit
import rateLimit from 'express-rate-limit'

// Límite estricto para auth
export const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutos
  max: 10,                    // máximo 10 intentos
  message: { success: false, message: 'Demasiados intentos, esperá 15 minutos' },
  standardHeaders: true,
})

// Límite general para toda la API
export const apiLimiter = rateLimit({
  windowMs: 60 * 1000,  // 1 minuto
  max: 100,
})

// Aplicar en rutas
app.use('/api/auth', authLimiter)
app.use('/api', apiLimiter)
```

```python
# FastAPI — slowapi
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@router.post("/login")
@limiter.limit("10/15minutes")
async def login(request: Request, data: LoginRequest):
    ...
```

---

## 7. Headers de seguridad (Helmet en Node.js)

```typescript
// Express — helmet configura headers seguros automáticamente
import helmet from 'helmet'
app.use(helmet())

// Qué hace helmet:
// X-Content-Type-Options: nosniff
// X-Frame-Options: DENY  (protege contra clickjacking)
// X-XSS-Protection: 1
// Strict-Transport-Security (fuerza HTTPS)
// Content-Security-Policy
```

```python
# FastAPI — configurar manualmente o usar secure headers
from fastapi.middleware.trustedhost import TrustedHostMiddleware
app.add_middleware(TrustedHostMiddleware, allowed_hosts=["tudominio.com", "*.tudominio.com"])
```

---

## 8. Autorización — roles y permisos

```typescript
// Middleware de roles
export const requireRole = (...roles: Role[]) =>
  (req: AuthRequest, res: Response, next: NextFunction) => {
    if (!roles.includes(req.user!.role)) {
      return res.status(403).json({ success: false, message: 'Sin permisos suficientes' })
    }
    next()
  }

// Uso en rutas
router.delete('/users/:id', authenticate, requireRole('ADMIN'), deleteUser)
```

---

## 9. Checklist de seguridad antes de entregar / desplegar

### Crítico (falla = proyecto inseguro)
- [ ] Sin credenciales hardcodeadas en el código
- [ ] `.env` en `.gitignore` y nunca commiteado
- [ ] Contraseñas hasheadas con bcrypt/argon2
- [ ] JWT con expiración corta (≤1h para access token)
- [ ] Validación de inputs en todos los endpoints
- [ ] Sin SQL injection posible (usar ORM siempre)
- [ ] CORS restringido al dominio del frontend
- [ ] Rutas protegidas con middleware de autenticación

### Importante
- [ ] Rate limiting en rutas de autenticación
- [ ] Headers de seguridad (Helmet o equivalente)
- [ ] HTTPS en producción (Let's Encrypt)
- [ ] Roles y permisos implementados correctamente
- [ ] Errores no exponen stack traces ni info interna
- [ ] Logs de eventos de seguridad (intentos fallidos de login)

### Para proyectos universitarios con nota
- [ ] Mencionar en la documentación qué medidas de seguridad implementaste
- [ ] Diagrama de flujo de autenticación en el informe
- [ ] Explicar por qué bcrypt y no MD5 (demuestra que entendés el concepto)

---

## 10. Errores de seguridad más comunes en proyectos uni

| Error | Consecuencia | Solución |
|---|---|---|
| JWT_SECRET = "secret" | Cualquiera puede forjar tokens | Generar con crypto.randomBytes |
| Guardar token en localStorage | XSS roba la sesión | httpOnly cookie para refresh |
| Sin validar inputs | SQL injection, crashes | Zod / Pydantic en todos los endpoints |
| CORS con `*` en prod | Cualquier sitio puede hacer requests | Restringir al dominio exacto |
| Contraseña en texto plano | BD comprometida = usuarios comprometidos | bcrypt factor 12 mínimo |
| Sin rate limiting en login | Brute force ilimitado | express-rate-limit / slowapi |
| Exponer stack traces | Info del servidor filtrada | NODE_ENV=production |
