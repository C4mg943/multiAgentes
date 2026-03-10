# 🏃 Agente Metodología — Scrum / Agile

> Usá este agente cuando la materia pida Scrum, sprints o metodología ágil

## Scrum en equipos universitarios de 2-3 personas

Con equipo chico los roles se comprimen:
| Rol Scrum real | En la uni (2-3 personas) |
|---|---|
| Product Owner | El que habla con el profe / "cliente" |
| Scrum Master | Rotativo o el más organizado |
| Dev Team | Todos |

---

## Artefactos Scrum que generalmente piden

### 1. Product Backlog
Lista priorizada de todo lo que el sistema debe hacer.

```markdown
## Product Backlog — [Nombre del proyecto]

| ID | Historia de usuario | Prioridad | Estimación (puntos) | Sprint |
|---|---|---|---|---|
| US-01 | Como estudiante, quiero iniciar sesión para acceder al sistema | Alta | 3 | 1 |
| US-02 | Como estudiante, quiero ver mis notas por materia | Alta | 5 | 1 |
| US-03 | Como admin, quiero registrar notas de estudiantes | Alta | 8 | 2 |
| US-04 | Como estudiante, quiero descargar mi historial en PDF | Media | 5 | 3 |
| US-05 | Como admin, quiero generar reportes de rendimiento | Baja | 8 | 3 |
```

### 2. Formato de Historia de Usuario
```markdown
## US-01: Iniciar Sesión

**Como** estudiante universitario
**Quiero** iniciar sesión con mi email y contraseña
**Para** acceder a mi información académica de forma segura

**Criterios de aceptación:**
- [ ] El sistema acepta email + contraseña válidos y redirige al dashboard
- [ ] El sistema muestra error genérico si las credenciales son incorrectas
- [ ] El sistema bloquea el acceso tras 5 intentos fallidos
- [ ] La sesión expira después de 24 horas de inactividad

**Estimación:** 3 puntos
**Prioridad:** Alta
```

### 3. Sprint Planning
```markdown
## Sprint 1 — [Fecha inicio] al [Fecha fin]

**Objetivo del sprint:** El usuario puede autenticarse y ver su información básica

**Historias seleccionadas:**
| Historia | Puntos | Responsable |
|---|---|---|
| US-01: Login | 3 | Juan |
| US-02: Ver notas | 5 | María |
| US-06: Navbar y layout base | 2 | Juan |

**Total de puntos:** 10

**Definition of Done (DoD):**
- Código en GitHub (branch mergeado a main)
- Funciona sin errores en local
- Revisado por al menos un compañero
- Documentado si es funcionalidad nueva
```

### 4. Daily Standup (para equipos uni)
Con 2-3 personas no hace falta reunión formal, pero sí un registro rápido.
Usar un canal de WhatsApp o Discord con este formato diario:

```
📅 [Fecha]
✅ Ayer hice: [qué completé]
🔨 Hoy voy a hacer: [qué planeo]
🚧 Bloqueos: [si hay algo que me traba]
```

### 5. Sprint Review / Retrospectiva
```markdown
## Retrospectiva Sprint 1

### ✅ ¿Qué salió bien?
- Completamos todas las historias planificadas
- La comunicación por Discord funcionó bien

### ❌ ¿Qué salió mal?
- Estimamos mal US-02, tardó el doble
- Conflictos de merge al final del sprint

### 🔧 ¿Qué mejoramos para el próximo sprint?
- Crear branches por feature, no trabajar todos en main
- Hacer estimaciones más conservadoras
```

---

## Estimación con Planning Poker

Puntos de Fibonacci que usamos: **1 · 2 · 3 · 5 · 8 · 13**

| Puntos | Significa |
|---|---|
| 1 | Trivial — menos de 1 hora |
| 2 | Simple — unas horas |
| 3 | Mediano — medio día |
| 5 | Complejo — 1-2 días |
| 8 | Muy complejo — 3-4 días |
| 13 | Épica — dividir en historias más chicas |

---

## Tablero Kanban (GitHub Projects es gratis)

Columnas recomendadas:
```
📋 Backlog → 🔜 To Do (Sprint) → 🔨 In Progress → 👀 In Review → ✅ Done
```

Para configurarlo: GitHub → tu repo → Projects → New project → Board

---

## Burndown Chart (si el profe lo pide)

Llevar registro de puntos completados por día del sprint:

```markdown
| Día | Puntos restantes ideales | Puntos restantes reales |
|---|---|---|
| Día 1 | 10 | 10 |
| Día 3 | 7 | 9 |
| Día 5 | 5 | 6 |
| Día 7 | 2 | 3 |
| Día 8 | 0 | 0 |
```

---

## Git + Scrum — flujo de trabajo en equipo

```bash
# Cada historia de usuario → su propio branch
git checkout -b feature/US-01-login

# Commits frecuentes con referencia a la historia
git commit -m "feat(auth): add login form validation — US-01"
git commit -m "feat(auth): implement JWT on login endpoint — US-01"

# Al terminar → Pull Request a main
# Otro integrante revisa → aprueba → merge

# Nunca pushear directo a main
```

### Branch naming convention
```
feature/US-01-login
feature/US-03-registrar-notas
fix/US-02-notas-no-cargan
hotfix/login-crash-produccion
```

---

## Tips para la entrega universitaria
- GitHub Projects es suficiente como herramienta Scrum, no necesitás Jira
- Guardar capturas del tablero Kanban en cada sprint para evidencia
- El profe valora que el burndown refleje la realidad, no que sea perfecto
- Vincular commits a historias (mencionando el ID) demuestra trazabilidad
