# Fase 8: Autenticación de Usuarios

## Objetivo

Implementar autenticación JWT con Spring Security para que:
- **Cualquier usuario (anónimo)** pueda ver TODAS las colecciones (perfil público)
- **Solo usuarios registrados** puedan crear, editar y eliminar elementos
- **Notas personales y valoraciones** sean visibles solo por el dueño

---

## Modelo de Datos

### User (colección `users`)

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único |
| username | String | ✅ | Nombre de usuario (unique, 3-20 chars) |
| email | String | ✅ | Email (unique, validado) |
| password | String | ✅ | Hash BCrypt (nunca se devuelve) |
| displayName | String | ❌ | Nombre para mostrar |
| avatarUrl | String | ❌ | URL de avatar |
| bio | String | ❌ | Biografía corta (max 200 chars) |
| createdAt | LocalDateTime | Auto | Fecha de registro |
| updatedAt | LocalDateTime | Auto | Última actualización |

### Cambios en entidades existentes

Añadir `ownerId` (String) a todas las entidades de colección:

| Entidad | Campo añadición |
|---------|-----------------|
| Book | `ownerId` |
| Game | `ownerId` |
| BoardGame | `ownerId` |
| MagicCard | `ownerId` |
| Deck | `ownerId` |
| MovieShow | `ownerId` |

**Migración:** Existentes se asocian a `ownerId = "system"` (cuenta admin por defecto).

---

## Visibilidad

| Acción | Anónimo | Usuario registrado | Admin |
|--------|---------|-------------------|-------|
| Ver colecciones de otros | ✅ | ✅ | ✅ |
| Ver notas/valoraciones propias | ❌ | ✅ | ✅ |
| Ver notas/valoraciones de otros | ❌ | ❌ | ✅ |
| Crear elementos | ❌ | ✅ | ✅ |
| Editar/eliminar elementos propios | ❌ | ✅ | ✅ |
| Editar/eliminar cualquier elemento | ❌ | ❌ | ✅ |

---

## Pasos de Implementación

### 1. Dependencias (pom.xml)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.6</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.6</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.6</version>
</dependency>
```

---

### 2. Configuración JWT (application.properties)

```properties
# JWT
app.jwt.secret=${JWT_SECRET:clave-cambiar-en-produccion-min-256-bits}
app.jwt.access-token-expiration=900000      # 15 minutos
app.jwt.refresh-token-expiration=604800000  # 7 días

# Admin por defecto
app.admin.username=${ADMIN_USERNAME:admin}
app.admin.password=${ADMIN_PASSWORD:admin123}
app.admin.email=${ADMIN_EMAIL:admin@local.dev}
```

---

### 3. Entidad User

```java
@Document(collection = "users")
public class User {
    @Id
    private String id;
    
    @Indexed(unique = true)
    @Size(min = 3, max = 20)
    @Pattern(regexp = "^[a-zA-Z0-9_]+$")
    private String username;
    
    @Indexed(unique = true)
    @Email
    private String email;
    
    @JsonIgnore
    private String password;
    
    private String displayName;
    private String avatarUrl;
    
    @Size(max = 200)
    private String bio;
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
    
    // Getters, setters, builder
}
```

---

### 4. UserRepository

```java
public interface SpringDataUserRepository extends MongoRepository<User, String> {
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
}
```

---

### 5. JwtService

```java
@Service
public class JwtService {
    
    @Value("${app.jwt.secret}")
    private String secret;
    
    @Value("${app.jwt.access-token-expiration}")
    private long accessTokenExpiration;
    
    @Value("${app.jwt.refresh-token-expiration}")
    private long refreshTokenExpiration;
    
    public String generateAccessToken(User user) {
        return Jwts.builder()
            .subject(user.getId())
            .claim("username", user.getUsername())
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + accessTokenExpiration))
            .signWith(getSigningKey())
            .compact();
    }
    
    public String generateRefreshToken(User user) {
        return Jwts.builder()
            .subject(user.getId())
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + refreshTokenExpiration))
            .signWith(getSigningKey())
            .compact();
    }
    
    public String extractUserId(String token) {
        return Jwts.parser()
            .verifyWith(getSigningKey())
            .build()
            .parseSignedClaims(token)
            .getPayload()
            .getSubject();
    }
    
    public boolean isTokenValid(String token) {
        try {
            Jwts.parser().verifyWith(getSigningKey()).build().parseSignedClaims(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }
    
    private SecretKey getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```

---

### 6. JwtAuthenticationFilter

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    @Autowired
    private JwtService jwtService;
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        
        String authHeader = request.getHeader("Authorization");
        
        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            String token = authHeader.substring(7);
            
            if (jwtService.isTokenValid(token)) {
                String userId = jwtService.extractUserId(token);
                UserDetails userDetails = userDetailsService.loadUserByUsername(userId);
                
                UsernamePasswordAuthenticationToken authToken =
                    new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities());
                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        
        filterChain.doFilter(request, response);
    }
}
```

---

### 7. SecurityConfig

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {
    
    @Autowired
    private JwtAuthenticationFilter jwtAuthFilter;
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                // Públicos: ver colecciones
                .requestMatchers(HttpMethod.GET, "/api/v1/**").permitAll()
                // Auth endpoints
                .requestMatchers("/api/v1/auth/**").permitAll()
                // Swagger
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                // POST, PUT, DELETE requieren autenticación
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

---

### 8. AuthService

```java
@Service
public class AuthService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Autowired
    private JwtService jwtService;
    
    @Autowired
    private AuthenticationManager authenticationManager;
    
    public AuthResponse register(RegisterRequest request) {
        if (userRepository.existsByUsername(request.username())) {
            throw new UserAlreadyExistsException("El usuario ya existe");
        }
        if (userRepository.existsByEmail(request.email())) {
            throw new EmailAlreadyExistsException("El email ya está registrado");
        }
        
        User user = User.builder()
            .username(request.username())
            .email(request.email())
            .password(passwordEncoder.encode(request.password()))
            .displayName(request.displayName())
            .build();
        
        userRepository.save(user);
        
        String accessToken = jwtService.generateAccessToken(user);
        String refreshToken = jwtService.generateRefreshToken(user);
        
        return new AuthResponse(accessToken, refreshToken, user.toResponse());
    }
    
    public AuthResponse login(LoginRequest request) {
        authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.username(), request.password()));
        
        User user = userRepository.findByUsername(request.username())
            .orElseThrow(() -> new UserNotFoundException("Usuario no encontrado"));
        
        String accessToken = jwtService.generateAccessToken(user);
        String refreshToken = jwtService.generateRefreshToken(user);
        
        return new AuthResponse(accessToken, refreshToken, user.toResponse());
    }
    
    public AuthResponse refreshToken(String refreshToken) {
        if (!jwtService.isTokenValid(refreshToken)) {
            throw new InvalidTokenException("Token inválido");
        }
        
        String userId = jwtService.extractUserId(refreshToken);
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new UserNotFoundException("Usuario no encontrado"));
        
        String newAccessToken = jwtService.generateAccessToken(user);
        String newRefreshToken = jwtService.generateRefreshToken(user);
        
        return new AuthResponse(newAccessToken, newRefreshToken, user.toResponse());
    }
}
```

---

### 9. AuthController

```java
@RestController
@RequestMapping("/api/v1/auth")
public class AuthController {
    
    @Autowired
    private AuthService authService;
    
    @PostMapping("/register")
    public ResponseEntity<AuthResponse> register(@Valid @RequestBody RegisterRequest request) {
        return ResponseEntity.status(201).body(authService.register(request));
    }
    
    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@Valid @RequestBody LoginRequest request) {
        return ResponseEntity.ok(authService.login(request));
    }
    
    @PostMapping("/refresh")
    public ResponseEntity<AuthResponse> refresh(@RequestBody RefreshTokenRequest request) {
        return ResponseEntity.ok(authService.refreshToken(request.refreshToken()));
    }
    
    @GetMapping("/me")
    public ResponseEntity<UserResponse> getCurrentUser(Authentication auth) {
        // Devuelve el usuario autenticado
    }
}
```

---

### 10. DTOs de Auth

```java
public record RegisterRequest(
    @NotBlank @Size(min = 3, max = 20) @Pattern(regexp = "^[a-zA-Z0-9_]+$") String username,
    @NotBlank @Email String email,
    @NotBlank @Size(min = 8) String password,
    String displayName
) {}

public record LoginRequest(
    @NotBlank String username,
    @NotBlank String password
) {}

public record RefreshTokenRequest(
    @NotBlank String refreshToken
) {}

public record AuthResponse(
    String accessToken,
    String refreshToken,
    UserResponse user
) {}

public record UserResponse(
    String id,
    String username,
    String displayName,
    String avatarUrl,
    String bio,
    LocalDateTime createdAt
) {}
```

---

### 11. UserDetailsServiceImpl

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public UserDetails loadUserByUsername(String userId) throws UsernameNotFoundException {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new UsernameNotFoundException("Usuario no encontrado"));
        
        return org.springframework.security.core.userdetails.User.builder()
            .username(user.getId())
            .password(user.getPassword())
            .roles("USER")
            .build();
    }
}
```

---

### 12. Anotación @CurrentUser

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface CurrentUser {
}
```

```java
@Component
public class CurrentUserHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {
    
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(CurrentUser.class)
            && parameter.getParameterType().equals(String.class);
    }
    
    @Override
    public Object resolveArgument(MethodParameter parameter,
                                  ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest,
                                  WebDataBinderFactory binderFactory) {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth == null || !auth.isAuthenticated()) {
            return null;
        }
        return auth.getName(); // userId
    }
}
```

---

### 13. Modificar servicios existentes para ownerId

Ejemplo con BookService:

```java
@Service
public class BookService implements BookUseCase {
    
    public Book save(Book book, String ownerId) {
        book.setOwnerId(ownerId);
        return bookRepository.save(book);
    }
    
    public Book update(String id, Book updates, String ownerId) {
        Book existing = bookRepository.findById(id)
            .orElseThrow(() -> new BookNotFoundException(id));
        
        if (!existing.getOwnerId().equals(ownerId) && !isAdmin(ownerId)) {
            throw new UnauthorizedException("No puedes editar este libro");
        }
        
        // ... actualizar campos
    }
}
```

---

### 14. Poblar ownerId en GET

Filtros de visibilidad en controladores:

```java
@GetMapping("/{id}")
public ResponseEntity<BookResponse> getBook(@PathVariable String id,
                                            @CurrentUser String currentUserId) {
    Book book = bookService.findById(id);
    BookResponse response = bookMapper.toResponse(book);
    
    // Si no es el dueño, ocultar notas y valoraciones
    if (currentUserId == null || !book.getOwnerId().equals(currentUserId)) {
        response = response.withNotes(null).withComment(null).withStart(null);
    }
    
    return ResponseEntity.ok(response);
}
```

---

### 15. Frontend: AuthContext

```typescript
// src/context/AuthContext.tsx
interface AuthContextType {
  user: UserResponse | null;
  accessToken: string | null;
  login: (username: string, password: string) => Promise<void>;
  register: (data: RegisterData) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
}

export const AuthProvider = ({ children }: { children: ReactNode }) => {
  const [user, setUser] = useState<UserResponse | null>(null);
  const [accessToken, setAccessToken] = useState<string | null>(null);

  const login = async (username: string, password: string) => {
    const res = await fetch('/api/v1/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ username, password }),
    });
    const data = await res.json();
    setAccessToken(data.accessToken);
    setUser(data.user);
    localStorage.setItem('refreshToken', data.refreshToken);
  };

  // ... register, logout, token refresh

  return (
    <AuthContext.Provider value={{ user, accessToken, login, register, logout, isAuthenticated: !!user }}>
      {children}
    </AuthContext.Provider>
  );
};
```

---

### 16. Frontend: ProtectedRoute

```typescript
// src/components/ProtectedRoute.tsx
export const ProtectedRoute = ({ children }: { children: ReactNode }) => {
  const { isAuthenticated } = useAuth();
  
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  
  return <>{children}</>;
};
```

---

### 17. Frontend: Axios Interceptor

```typescript
// src/api/client.ts
const apiClient = axios.create({ baseURL: '/api/v1' });

apiClient.interceptors.request.use(config => {
  const token = localStorage.getItem('accessToken');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

apiClient.interceptors.response.use(
  response => response,
  async error => {
    if (error.response?.status === 401) {
      // Intentar refresh token
      const refreshToken = localStorage.getItem('refreshToken');
      if (refreshToken) {
        try {
          const res = await refreshTokenApi(refreshToken);
          localStorage.setItem('accessToken', res.accessToken);
          error.config.headers.Authorization = `Bearer ${res.accessToken}`;
          return apiClient.request(error.config);
        } catch {
          // Refresh fallido → logout
          logout();
          window.location.href = '/login';
        }
      }
    }
    return Promise.reject(error);
  }
);
```

---

### 18. Frontend: Páginas Auth

- `LoginPage.tsx` → Formulario login + link a register
- `RegisterPage.tsx` → Formulario registro + link a login
- `ProfilePage.tsx` → Perfil del usuario (editable)

---

### 19. Frontend: Rutas protegidas

```typescript
<Route path="/login" element={<LoginPage />} />
<Route path="/register" element={<RegisterPage />} />
<Route path="/perfil" element={<ProtectedRoute><ProfilePage /></ProtectedRoute>} />

// Rutas que requieren auth para crear
<Route path="/nuevo" element={<ProtectedRoute><BookCreatePage /></ProtectedRoute>} />
<Route path="/juegos/nuevo" element={<ProtectedRoute><GameCreatePage /></ProtectedRoute>} />
// ... resto de rutas de creación/edición protegidas
```

---

### 20. Tests

**Backend:**
- `JwtServiceTest` → Generación y validación de tokens
- `AuthServiceTest` → Registro, login, refresh
- `AuthControllerTest` → Endpoints HTTP con MockMvc
- `JwtAuthenticationFilterTest` → Filtro con SecurityContext
- `SecurityConfigTest` → Acceso público vs protegido

**Frontend:**
- `AuthContext.test.tsx` → Estado de auth
- `ProtectedRoute.test.tsx` → Redirección
- `LoginPage.test.tsx` → Formulario

---

## Estructura de Paquetes Final

```
com.wikicollection/
├── domain/
│   ├── model/
│   │   └── User.java                    ← NUEVO
│   └── port/
│       ├── in/
│       │   └── UserUseCase.java         ← NUEVO
│       └── out/
│           └── UserRepository.java      ← NUEVO
├── application/
│   ├── exception/
│   │   ├── UserAlreadyExistsException.java  ← NUEVO
│   │   ├── InvalidTokenException.java       ← NUEVO
│   │   └── UnauthorizedException.java       ← NUEVO
│   └── service/
│       ├── AuthService.java             ← NUEVO
│       └── JwtService.java              ← NUEVO
└── infrastructure/
    ├── adapter/
    │   ├── in/web/
    │   │   ├── AuthController.java      ← NUEVO
    │   │   └── dto/
    │   │       ├── RegisterRequest.java ← NUEVO
    │   │       ├── LoginRequest.java    ← NUEVO
    │   │       ├── RefreshTokenRequest.java ← NUEVO
    │   │       ├── AuthResponse.java    ← NUEVO
    │   │       └── UserResponse.java    ← NUEVO
    │   └── out/
    │       └── persistence/
    │           ├── UserEntity.java      ← NUEVO
    │           ├── UserEntityMapper.java ← NUEVO
    │           ├── UserPersistenceAdapter.java ← NUEVO
    │           └── SpringDataUserRepository.java ← NUEVO
    └── config/
        ├── SecurityConfig.java          ← NUEVO
        └── JwtAuthenticationFilter.java ← NUEVO
```

---

## Estado: ✅ COMPLETADA

- [x] Añadir dependencias (Spring Security, JJWT)
- [x] Crear User entity + UserRepository
- [x] Implementar JwtService
- [x] Implementar JwtAuthenticationFilter
- [x] Implementar SecurityConfig
- [x] Implementar AuthService
- [x] Implementar AuthController
- [x] Añadir ownerId a entidades existentes
- [x] Modificar servicios para verificar ownership
- [x] Crear DTOs de Auth
- [x] Crear UserDetailsServiceImpl
- [x] Crear @CurrentUser annotation
- [x] Frontend: AuthContext
- [x] Frontend: ProtectedRoute
- [x] Frontend: Axios interceptor
- [x] Frontend: LoginPage + RegisterPage + ProfilePage
- [x] Frontend: Proteger rutas de creación/edición
- [x] Tests backend (5 archivos)
- [x] Tests frontend (3 archivos)
- [x] Migrar datos existentes (ownerId = "system")

---

## Criterios de Aceptación

- [ ] Un usuario puede registrarse vía `POST /api/v1/auth/register`
- [ ] Un usuario puede logearse vía `POST /api/v1/auth/login`
- [ ] El access token expira en 15 minutos
- [ ] El refresh token expira en 7 días
- [ ] Se puede refrescar el access token vía `POST /api/v1/auth/refresh`
- [ ] Los endpoints GET son públicos (sin auth)
- [ ] Los endpoints POST/PUT/DELETE requieren auth
- [ ] Un usuario solo puede editar/eliminar sus propios elementos
- [ ] Las notas/valoraciones son privadas (solo visibles por el dueño)
- [ ] Las colecciones son públicas (visibles por anónimos)
- [ ] El frontend redirige a login si intenta crear sin auth
- [ ] El frontend maneja expiración de token con refresh automático
- [ ] Los tests pasan (`mvn verify` y `npm test`)
- [ ] JaCoCo mantiene ≥80% cobertura

---

## Notas

- **Passwords:** BCrypt con strength 10
- **JWT Secret:** Generar con `openssl rand -base64 32`
- **Refresh tokens:** Se almacenan solo en localStorage (no en MongoDB por simplicidad)
- **Revocación:** Para logout, simplemente eliminar el refresh token del localStorage. Si necesitas revocación real, añadir una blacklist en Redis o base de datos.
- **Migración:** Los datos existentes se asocian a `ownerId = "system"`. Opcionalmente, crear script para asignar a usuarios reales.
- **Roles:** Inicialmente solo `USER`. El admin por defecto se configura por env vars.
- **OAuth2 futuro:** La estructura permite añadir Google/GitHub login más adelante sin reescribir.

---

## Referencias

- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
- [JJWT Documentation](https://github.com/jwtk/jjwt)
- [JWT Best Practices](https://datatracker.ietf.org/doc/html/rfc8725)
