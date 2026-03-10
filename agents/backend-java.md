# ☕ Agente Backend — Java + Spring Boot

> Leé este archivo cuando trabajes en proyectos Java (típico en materias de arquitectura empresarial)

## Stack exacto
- **Java** 21 LTS (usar records, sealed classes, pattern matching)
- **Spring Boot** 3.2+
- **Spring Security** 6 para autenticación/autorización
- **Spring Data JPA** + **Hibernate** como ORM
- **PostgreSQL** como base de datos
- **Maven** como gestor de dependencias (no Gradle salvo que el proyecto lo exija)
- **JWT** via `jjwt` (io.jsonwebtoken)
- **MapStruct** para mapeo de DTOs
- **JUnit 5 + Mockito** para tests

## Estructura de paquetes
```
com.tuapp/
├── config/               # Configuración de Spring Security, CORS, etc.
├── controller/           # REST controllers (@RestController)
├── service/              # Lógica de negocio (interfaces + implementaciones)
│   └── impl/
├── repository/           # Interfaces JPA
├── entity/               # Entidades JPA (@Entity)
├── dto/                  # Data Transfer Objects (request y response)
├── mapper/               # MapStruct mappers
├── exception/            # Excepciones propias y GlobalExceptionHandler
├── security/             # JWT filter, UserDetailsService
└── util/                 # Helpers
```

## Entidad JPA moderna con Records-style
```java
// entity/User.java
@Entity
@Table(name = "users")
@Getter @Setter @Builder @NoArgsConstructor @AllArgsConstructor // Lombok
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(nullable = false)
    private String password;

    @Column(length = 100)
    private String name;

    @Enumerated(EnumType.STRING)
    private Role role = Role.USER;

    @CreationTimestamp
    private LocalDateTime createdAt;
}
```

## DTOs — siempre separar request de response
```java
// dto/UserRequest.java (lo que llega)
public record UserRequest(
    @NotBlank @Email String email,
    @NotBlank @Size(min = 8) String password,
    @NotBlank String name
) {}

// dto/UserResponse.java (lo que sale — nunca exponer la contraseña)
public record UserResponse(Long id, String email, String name, LocalDateTime createdAt) {}
```

## Servicio con interfaz
```java
// service/UserService.java
public interface UserService {
    UserResponse findById(Long id);
    List<UserResponse> findAll();
    UserResponse create(UserRequest request);
}

// service/impl/UserServiceImpl.java
@Service
@RequiredArgsConstructor
@Transactional
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;
    private final UserMapper userMapper;

    @Override
    @Transactional(readOnly = true)
    public UserResponse findById(Long id) {
        return userRepository.findById(id)
            .map(userMapper::toResponse)
            .orElseThrow(() -> new ResourceNotFoundException("Usuario", id));
    }
}
```

## Controller REST
```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;

    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getById(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }

    @PostMapping
    public ResponseEntity<UserResponse> create(@Valid @RequestBody UserRequest request) {
        UserResponse created = userService.create(request);
        URI location = URI.create("/api/v1/users/" + created.id());
        return ResponseEntity.created(location).body(created);
    }
}
```

## Manejo global de excepciones
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(404)
            .body(new ErrorResponse(ex.getMessage(), 404));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult().getFieldErrors()
            .stream().map(e -> e.getField() + ": " + e.getDefaultMessage())
            .toList();
        return ResponseEntity.status(400).body(new ErrorResponse("Validación fallida", 400, errors));
    }
}
```

## Lo que SIEMPRE debo recordarme
- DTOs para entrada y salida, NUNCA exponer entidades JPA directamente
- `@Transactional(readOnly = true)` en métodos de solo lectura (performance)
- `@Valid` en los `@RequestBody` de los controllers para activar validaciones
- Inyección por constructor (Lombok `@RequiredArgsConstructor`), no `@Autowired` en campos
- `application.yml` en vez de `application.properties` (más legible)
- Perfiles de Spring: `dev`, `prod` con configuraciones separadas
