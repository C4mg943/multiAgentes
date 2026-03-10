# 🐍 Agente Backend — Python + FastAPI

> Leé este archivo cuando trabajes en APIs con Python

## Stack exacto
- **Python** 3.11+
- **FastAPI** 0.110+ (async nativo)
- **SQLAlchemy** 2.x (ORM, con async engine)
- **Alembic** para migraciones de DB
- **Pydantic** v2 para schemas y validación (ya viene con FastAPI)
- **PostgreSQL** via `asyncpg`
- **python-jose** para JWT
- **passlib** para hashing (bcrypt)
- **pytest + httpx** para tests

## Estructura de carpetas
```
app/
├── api/
│   ├── v1/
│   │   ├── endpoints/        # Routers por entidad
│   │   │   ├── auth.py
│   │   │   └── users.py
│   │   └── router.py         # Agrega todos los endpoints
├── core/
│   ├── config.py             # Settings con Pydantic BaseSettings
│   ├── security.py           # JWT, hashing
│   └── database.py           # Engine, SessionLocal, Base
├── models/                   # Modelos SQLAlchemy (tablas)
├── schemas/                  # Schemas Pydantic (request/response)
├── services/                 # Lógica de negocio
├── repositories/             # Queries a DB
└── main.py                   # Entry point, monta la app
```

## Modelo SQLAlchemy moderno
```python
# models/user.py
from sqlalchemy.orm import Mapped, mapped_column
from datetime import datetime
from app.core.database import Base

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(unique=True, index=True)
    hashed_password: Mapped[str]
    name: Mapped[str] = mapped_column(String(100))
    is_active: Mapped[bool] = mapped_column(default=True)
    created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow)
```

## Schema Pydantic v2
```python
# schemas/user.py
from pydantic import BaseModel, EmailStr, ConfigDict

class UserCreate(BaseModel):
    email: EmailStr
    password: str
    name: str

class UserResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True) # ORM mode

    id: int
    email: str
    name: str
    is_active: bool
```

## Endpoint con dependencias
```python
# api/v1/endpoints/users.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession

router = APIRouter(prefix="/users", tags=["users"])

@router.get("/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: int,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    user = await user_service.get_by_id(db, user_id)
    if not user:
        raise HTTPException(status_code=404, detail="Usuario no encontrado")
    return user
```

## Configuración con variables de entorno
```python
# core/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    DATABASE_URL: str
    SECRET_KEY: str
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30

    class Config:
        env_file = ".env"

settings = Settings()
```

## Manejo de errores personalizado
```python
# Siempre usar HTTPException con códigos correctos
raise HTTPException(
    status_code=status.HTTP_404_NOT_FOUND,
    detail="Recurso no encontrado"
)

# Para errores de negocio, crear excepciones propias y capturarlas en main.py
class BusinessError(Exception):
    def __init__(self, message: str, code: int = 400):
        self.message = message
        self.code = code
```

## Lo que SIEMPRE debo recordarme
- Todo async: `async def` en endpoints y servicios que toquen DB
- `await` en todas las operaciones de DB
- Cerrar sesión de DB con `finally` o usando `Depends` correctamente
- Nunca retornar el modelo SQLAlchemy directo, siempre transformar al schema Pydantic
- Variables de entorno con `pydantic-settings`, nunca hardcodeadas
- Documentación automática disponible en `/docs` (Swagger) y `/redoc`
