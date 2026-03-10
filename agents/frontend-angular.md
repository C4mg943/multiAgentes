# 🅰️ Agente Frontend — Angular

> Leé este archivo cuando trabajes en proyectos con Angular (típico en cursos empresariales)

## Stack exacto
- **Angular** 17+ con Standalone Components (no NgModules salvo que el proyecto lo exija)
- **TypeScript** 5.x — strict mode activado
- **Angular Material** o **PrimeNG** para componentes UI
- **RxJS** 7+ para manejo de estado reactivo y operaciones asíncronas
- **NgRx** solo si el proyecto es grande (>5 devs o lo pide la cátedra), sino servicios + signals
- **Angular Signals** para estado local reactivo (Angular 17+)
- **Reactive Forms** para formularios complejos, Template-driven para simples

## Estructura de carpetas
```
src/
├── app/
│   ├── core/                 # Servicios singleton, guards, interceptors
│   │   ├── guards/
│   │   ├── interceptors/
│   │   └── services/
│   ├── features/             # Módulos por funcionalidad
│   │   ├── auth/
│   │   │   ├── login/
│   │   │   │   ├── login.component.ts
│   │   │   │   └── login.component.html
│   │   │   └── auth.routes.ts
│   │   └── dashboard/
│   ├── shared/               # Componentes, pipes y directivas reutilizables
│   │   ├── components/
│   │   ├── pipes/
│   │   └── directives/
│   ├── app.config.ts         # Configuración de la app (standalone)
│   └── app.routes.ts         # Rutas principales
└── environments/
```

## Componente standalone moderno (Angular 17+)
```typescript
// ✅ Componente bien hecho con signals
import { Component, signal, computed, inject } from '@angular/core'
import { CommonModule } from '@angular/common'
import { UserService } from '@core/services/user.service'

@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div *ngIf="isLoading(); else content">Cargando...</div>
    <ng-template #content>
      <ul>
        @for (user of users(); track user.id) {
          <li>{{ user.name }}</li>
        }
      </ul>
    </ng-template>
  `
})
export class UserListComponent {
  private userService = inject(UserService) // inject() en lugar de constructor

  users = signal<User[]>([])
  isLoading = signal(true)
  userCount = computed(() => this.users().length)

  ngOnInit() {
    this.userService.getAll().subscribe({
      next: (data) => { this.users.set(data); this.isLoading.set(false) },
      error: (err) => console.error(err)
    })
  }
}
```

## Servicios — patrón correcto
```typescript
@Injectable({ providedIn: 'root' }) // singleton automático
export class ProductService {
  private http = inject(HttpClient)
  private apiUrl = environment.apiUrl

  getAll(): Observable<Product[]> {
    return this.http.get<Product[]>(`${this.apiUrl}/products`)
  }

  getById(id: string): Observable<Product> {
    return this.http.get<Product>(`${this.apiUrl}/products/${id}`)
  }
}
```

## RxJS — operadores que más uso
```typescript
// Pipe de operadores comunes
this.searchTerm$.pipe(
  debounceTime(300),        // esperar que el usuario deje de escribir
  distinctUntilChanged(),   // no repetir si el valor no cambió
  switchMap(term => this.service.search(term)), // cancelar llamadas anteriores
  catchError(err => { this.error.set(true); return EMPTY })
).subscribe(results => this.results.set(results))
```

## Guards para rutas protegidas
```typescript
// core/guards/auth.guard.ts
export const authGuard: CanActivateFn = () => {
  const authService = inject(AuthService)
  const router = inject(Router)
  return authService.isAuthenticated()
    ? true
    : router.createUrlTree(['/login'])
}
```

## Lo que SIEMPRE debo recordarme
- Usar `inject()` en lugar de inyección por constructor en componentes nuevos
- Desuscribirse de Observables: usar `takeUntilDestroyed()` o `async` pipe
- `@for` y `@if` (nueva sintaxis) en vez de `*ngFor` y `*ngIf` (Angular 17+)
- Lazy loading en todas las rutas con `loadComponent`
- Interceptor para agregar el JWT automáticamente a todas las peticiones
