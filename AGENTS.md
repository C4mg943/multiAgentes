# AGENTS.md — Sistema de Agentes

## Quién soy
Estudiante de ingeniería de sistemas, universidad. Equipos de 2-3 personas.
Nivel intermedio-avanzado. Necesito código limpio, completo y con buenas prácticas.

## Cómo responderme
- Código siempre funcional y completo, nunca fragmentos a medias
- Explicá el por qué de decisiones importantes
- Español en explicaciones, inglés en el código
- Si hay algo malo en mi enfoque, decímelo directamente

## Mi entorno
- OS: Windows (nativo)
- Editor: VSCode + OpenCode con MiniMax M2.5
- Git + GitHub, metodología varía: RUP/UML o Scrum según materia
- Docker ocasional cuando la materia lo exige

## Stack por proyecto
| Tipo | Stack |
|---|---|
| Frontend Web | React 18 + Next.js 14 + Tailwind CSS |
| Frontend Empresarial | Angular 17 + TypeScript |
| Backend JS | Node.js + Express + Prisma |
| Backend Python | FastAPI + SQLAlchemy |
| Backend Java | Spring Boot 3 |
| Base de datos | PostgreSQL (única para todo) |
| Despliegue | VPS + nginx + SSL + dominio propio |

## Reglas de código
1. Tipado siempre: TypeScript en JS/TS, type hints en Python
2. camelCase en JS/TS, snake_case en Python, PascalCase para clases
3. Sin console.log en producción, usar logger apropiado
4. Manejo de errores explícito, nunca swallow exceptions
5. Commits en inglés: type(scope): description
6. Nunca credenciales hardcodeadas, siempre .env
7. Patrón: Controller → Service → Repository → Database
8. Respuesta API: { success, data, meta } o { success, message, errors }

## Agentes especializados (leer cuando se necesite)
- `agents/frontend-react.md` → React 18 + Next.js 14 + Tailwind
- `agents/frontend-angular.md` → Angular 17 + TypeScript
- `agents/backend-node.md` → Node.js + Express + Prisma
- `agents/backend-python.md` → FastAPI + SQLAlchemy
- `agents/backend-java.md` → Spring Boot 3
- `agents/base-de-datos.md` → PostgreSQL
- `agents/metodologia-rup.md` → RUP / UML formal
- `agents/metodologia-scrum.md` → Scrum / Agile
- `agents/revisor.md` → Code review
- `agents/documentacion.md` → Documentar proyectos
- `agents/despliegue.md` → VPS + dominio + SSL

## Build / Lint / Test por stack

### Node.js
```bash
npm install && npm run dev
npm test                              # todos los tests
npm test -- --testNamePattern="name" # un solo test
npm run lint && npm run build
```

### Python
```bash
python -m venv venv && venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload
pytest / pytest -k "test_name"
ruff check . && mypy app/
```

### Java
```bash
./mvnw test
./mvnw test -Dtest=NombreClase
./mvnw clean package
```

### Frontend React/Next.js
```bash
npm run dev && npm run build && npm test && npm run lint
```

### Frontend Angular
```bash
ng serve && ng build
ng test --include='**/archivo.spec.ts'
```

## Convenciones de nombrado
| Elemento | Convención | Ejemplo |
|---|---|---|
| Variables/Funciones JS/TS | camelCase | `getUserById` |
| Variables/Funciones Python | snake_case | `get_user_by_id` |
| Clases (todos) | PascalCase | `UserService` |
| Componentes React | PascalCase.tsx | `UserCard.tsx` |
| Constantes | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |

## Estructura de carpetas
- **Node.js**: `src/{controllers,services,repositories,middlewares,routes,schemas,types,utils}`
- **Python**: `app/{api/v1/endpoints,core,models,schemas,services,repositories,main.py}`
- **React/Next.js**: `src/{app,components/ui|forms|[feature],hooks,lib,store,types,services}`

## Manejo de errores
- HTTP status codes correctos: 4xx cliente, 5xx servidor
- Nunca exponer stack traces en producción
- Global error handlers en todos los stacks
- Logging apropiado: Winston (Node), logging (Python)