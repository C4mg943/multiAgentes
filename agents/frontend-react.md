# 🎨 Agente Frontend — React + Next.js

> Leé este archivo cuando trabajes en proyectos de interfaz con React o Next.js

## Stack exacto
- **React** 18.3 con hooks funcionales (CERO clases)
- **Next.js** 14 con App Router (no Pages Router)
- **TypeScript** 5.x — strict mode activado
- **Tailwind CSS** 3.x — utility-first, sin CSS modules salvo excepciones
- **Zustand** para estado global (no Redux, es innecesariamente complejo para uni)
- **React Query (TanStack)** para fetching y caché de datos del servidor
- **React Hook Form + Zod** para formularios con validación
- **Axios** para llamadas HTTP

## Estructura de carpetas (Next.js App Router)
```
src/
├── app/                      # Rutas (App Router)
│   ├── (auth)/               # Grupo de rutas autenticadas
│   │   ├── dashboard/
│   │   │   └── page.tsx
│   │   └── layout.tsx
│   ├── api/                  # Route handlers
│   ├── globals.css
│   ├── layout.tsx            # Root layout
│   └── page.tsx              # Home
├── components/
│   ├── ui/                   # Componentes base reutilizables (Button, Input, Modal)
│   ├── forms/                # Formularios específicos
│   └── [feature]/            # Componentes por feature
├── hooks/                    # Custom hooks
├── lib/                      # Utilidades, configuración (axios instance, etc.)
├── store/                    # Zustand stores
├── types/                    # Interfaces y tipos TypeScript
└── services/                 # Llamadas a API organizadas por entidad
```

## Convenciones de componentes
```tsx
// ✅ ASÍ: componente bien hecho
interface UserCardProps {
  user: User
  onSelect?: (id: string) => void
  className?: string
}

export function UserCard({ user, onSelect, className }: UserCardProps) {
  return (
    <div className={cn("rounded-lg border p-4", className)}>
      {/* contenido */}
    </div>
  )
}

// ❌ NO HACER: export default en componentes (dificulta refactoring)
// ❌ NO HACER: props sin tipar
// ❌ NO HACER: lógica de negocio dentro del componente
```

## Custom hooks — siempre extraer lógica
```tsx
// hooks/useAuth.ts
export function useAuth() {
  const { data: user, isLoading } = useQuery({
    queryKey: ['user'],
    queryFn: authService.getMe,
  })
  return { user, isLoading, isAuthenticated: !!user }
}
```

## Manejo de estado — cuándo usar qué
| Situación | Solución |
|---|---|
| Estado local de UI (modal abierto, tab activo) | `useState` |
| Estado derivado o cálculos | `useMemo` |
| Datos del servidor | React Query |
| Estado global de app (usuario, carrito) | Zustand |
| Estado de formularios | React Hook Form |

## Tailwind — mis patrones
```tsx
// Variantes con cn() (instalar clsx + tailwind-merge)
import { cn } from "@/lib/utils"

// Responsive siempre mobile-first
<div className="flex flex-col gap-4 md:flex-row md:gap-8">

// Colores del proyecto centralizados en tailwind.config.ts
// No hardcodear colores arbitrarios
```

## Fetching de datos (Server vs Client)
```tsx
// Server Component — preferido para datos estáticos o SSR
async function ProductList() {
  const products = await productService.getAll() // fetch directo, sin hooks
  return <ul>{products.map(p => <ProductCard key={p.id} product={p} />)}</ul>
}

// Client Component — solo cuando hay interactividad
'use client'
function InteractiveWidget() {
  const { data } = useQuery({ queryKey: ['items'], queryFn: getItems })
  // ...
}
```

## Lo que SIEMPRE debo recordarme
- `'use client'` solo cuando realmente se necesita (estado, eventos, efectos)
- Imágenes siempre con `next/image`
- Links siempre con `next/link`
- Variables de entorno: `NEXT_PUBLIC_` solo para las que van al browser
- Nunca fetch en `useEffect` si React Query puede hacerlo
