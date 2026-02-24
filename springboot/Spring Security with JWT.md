# Spring Security with JWT - Notes

## 📂 Project Structure Overview
```
src/main/java/com/nick/taskmanager/
├── model/              # Database entities
├── repository/         # Database access
├── dto/               # Data Transfer Objects
├── security/          # JWT & Authentication filters
├── config/            # Security configuration
├── controller/        # REST endpoints
└── TaskManagerApplication.java
```

---

# 1️⃣ DTOs (Data Transfer Objects) 📦

## What are DTOs?
**Simple POJOs** that carry data between client and server. They define the exact structure of API requests/responses.

### 🔐 LoginRequest DTO
```java
package com.nick.taskmanager.dto;

public class LoginRequest {
    private String username;
    private String password;
    
    // Spring needs empty constructor for JSON deserialization
    public LoginRequest() {}
    
    // Getters/Setters - REQUIRED for Spring to access private fields
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public String getPassword() { return password; }
    public void setPassword(String password) { this.password = password; }
}
```

**📝 What happens:**
```json
// Request from client
POST /api/auth/login
{
    "username": "john.doe",    // → setUsername("john.doe")
    "password": "mypassword"    // → setPassword("mypassword")
}
```

### 📝 SignupRequest DTO
```java
public class SignupRequest {
    private String username;
    private String email;      // Extra field for registration
    private String password;
    
    // With validation (RECOMMENDED)
    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 20, message = "3-20 characters")
    private String username;
    
    @Email(message = "Invalid email format")
    @NotBlank(message = "Email is required")
    private String email;
    
    @NotBlank(message = "Password is required")
    @Size(min = 6, message = "Minimum 6 characters")
    private String password;
}
```

### 🎫 JwtResponse DTO
```java
public class JwtResponse {
    private String token;      // JWT token
    private String type = "Bearer";  // Token type (fixed)
    private String username;   // User's username
    private String email;      // User's email
    private List<String> roles; // User's authorities
    
    public JwtResponse(String token, String username, String email, List<String> roles) {
        this.token = token;
        this.username = username;
        this.email = email;
        this.roles = roles;
    }
}
```

**📤 Response sent to client:**
```json
{
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "type": "Bearer",
    "username": "john.doe",
    "email": "john@example.com",
    "roles": ["ROLE_USER"]
}
```

### 💬 MessageResponse DTO
```java
public class MessageResponse {
    private String message;    // Simple status message
    
    public MessageResponse(String message) {
        this.message = message;
    }
}
```

**📤 Usage:**
```json
// Success
{"message": "User registered successfully!"}

// Error  
{"message": "Error: Username is already taken!"}
```

---

# 2️⃣ Security Configuration 🛡️

## SecurityConfig - The Control Center

```java
package com.nick.taskmanager.config;

@Configuration           // This is a configuration class
@EnableWebSecurity      // Enable Spring Security
@EnableMethodSecurity   // Enable @PreAuthorize annotations
public class SecurityConfig {

    // 1. DEPENDENCIES - What we need
    private final UserDetailsService userDetailsService;
    private final JwtAuthenticationFilter jwtAuthenticationFilter;
    
    // 2. PASSWORD ENCODER - Never store plain text passwords!
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();  // Strong hashing + salt
    }
    
    // 3. AUTHENTICATION PROVIDER - How to authenticate users
    @Bean
    public DaoAuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider authProvider = 
            new DaoAuthenticationProvider(userDetailsService);
        authProvider.setPasswordEncoder(passwordEncoder());
        return authProvider;
    }
    
    // 4. AUTHENTICATION MANAGER - Main authentication orchestrator
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }
    
    // 5. SECURITY FILTER CHAIN - ALL security rules defined here!
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // Disable CSRF (we use JWT, not cookies)
            .csrf(csrf -> csrf.disable())
            
            // Enable CORS (for frontend access)
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            
            // STATELESS - No sessions!
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            
            // AUTHORIZATION RULES - WHO can access WHAT
            .authorizeHttpRequests(auth -> auth
                // 🔓 Public endpoints - anyone can access
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/", "/health").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                
                // 🔒 Protected endpoints - need authentication
                .requestMatchers("/actuator/**").hasRole("ADMIN")
                
                // 🚫 Everything else - authentication required
                .anyRequest().authenticated()
            );
        
        // Plug in our custom authentication provider
        http.authenticationProvider(authenticationProvider());
        
        // Add JWT filter BEFORE Spring's default filter
        http.addFilterBefore(jwtAuthenticationFilter, 
                            UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    // 6. CORS CONFIGURATION - Allow frontend to call API
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(List.of("*"));  // Any domain
        config.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE"));
        config.setAllowedHeaders(List.of("*"));
        config.setExposedHeaders(List.of("Authorization"));
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return source;
    }
}
```

### 🎯 Security Rules Explained:

| URL Pattern | Access | Why? |
|------------|--------|------|
| `/api/auth/**` | ✅ Everyone | Login/Register - no token needed |
| `/swagger-ui/**` | ✅ Everyone | API documentation |
| `/actuator/**` | 👑 ADMIN only | Sensitive metrics |
| `/api/tasks` | 🔐 Authenticated users | Need valid JWT |
| Any other URL | 🔐 Authenticated users | Protected by default |

---

# 3️⃣ JWT Utilities 🎫

## JwtUtils - Token Management

```java
package com.nick.taskmanager.security;

@Component
public class JwtUtils {
    
    @Value("${app.jwtSecret}")      // From application.properties
    private String jwtSecret;
    
    @Value("${app.jwtExpirationMs}") // From application.properties
    private int jwtExpirationMs;
    
    // 1. CREATE SIGNING KEY - For signing/verifying tokens
    private SecretKey getSigningKey() {
        return Keys.hmacShaKeyFor(jwtSecret.getBytes());
    }
    
    // 2. GENERATE TOKEN - Called after successful login
    public String generateJwtToken(Authentication authentication) {
        UserDetails userPrincipal = (UserDetails) authentication.getPrincipal();
        
        return Jwts.builder()
                .setSubject(userPrincipal.getUsername())  // Who owns this token
                .setIssuedAt(new Date())                  // When created
                .setExpiration(new Date(new Date().getTime() + jwtExpirationMs)) // When expires
                .signWith(getSigningKey(), SignatureAlgorithm.HS256) // Sign it
                .compact();  // Build the token string
    }
    
    // 3. EXTRACT USERNAME - Get user from token
    public String getUserNameFromJwtToken(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(getSigningKey())  // Verify signature
                .build()
                .parseClaimsJws(token)           // Parse token
                .getBody()                       // Get payload
                .getSubject();                   // Get username
    }
    
    // 4. VALIDATE TOKEN - Check if token is valid
    public boolean validateJwtToken(String authToken) {
        try {
            Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(authToken);      // Throws exception if invalid
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            // Token expired, bad signature, malformed, etc.
            return false;
        }
    }
}
```

### 🔑 JWT Token Structure:
```
Header:    {"alg": "HS256", "typ": "JWT"}
Payload:   {"sub": "john.doe", "iat": 1700000000, "exp": 1700003600}
Signature: HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)

Complete:  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
           eyJzdWIiOiJqb2huLmRvZSIsImlhdCI6MTcwMDAwMDAwMCwiZXhwIjoxNzAwMDAzNjAwfQ.
           SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

---

# 4️⃣ JWT Authentication Filter 🔍

## JwtAuthenticationFilter - Every Request Interceptor

```java
package com.nick.taskmanager.security;

@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    private final JwtUtils jwtUtils;
    private final UserDetailsService userDetailsService;
    
    @Autowired
    public JwtAuthenticationFilter(JwtUtils jwtUtils, 
                                  UserDetailsService userDetailsService) {
        this.jwtUtils = jwtUtils;
        this.userDetailsService = userDetailsService;
    }
    
    @Override
    protected void doFilterInternal(
            HttpServletRequest request,     // Incoming request
            HttpServletResponse response,   // Outgoing response
            FilterChain filterChain)        // Next filter in chain
            throws ServletException, IOException {
        
        try {
            // STEP 1: Extract JWT from Authorization header
            String jwt = parseJwt(request);
            
            // STEP 2: If token exists and is valid
            if (jwt != null && jwtUtils.validateJwtToken(jwt)) {
                
                // STEP 3: Get username from token
                String username = jwtUtils.getUserNameFromJwtToken(jwt);
                
                // STEP 4: Load user details from database
                UserDetails userDetails = 
                    userDetailsService.loadUserByUsername(username);
                
                // STEP 5: Create Authentication object
                UsernamePasswordAuthenticationToken authentication = 
                    new UsernamePasswordAuthenticationToken(
                        userDetails,      // Principal (the user)
                        null,            // Credentials (already authenticated)
                        userDetails.getAuthorities()  // Roles/permissions
                    );
                
                // STEP 6: Add request details
                authentication.setDetails(
                    new WebAuthenticationDetailsSource().buildDetails(request)
                );
                
                // STEP 7: Set authentication in SecurityContext
                SecurityContextHolder.getContext()
                    .setAuthentication(authentication);
            }
        } catch (Exception e) {
            logger.error("Cannot set authentication: " + e.getMessage());
        }
        
        // STEP 8: Continue with the filter chain
        filterChain.doFilter(request, response);
    }
    
    // Helper: Extract JWT from "Authorization: Bearer <token>"
    private String parseJwt(HttpServletRequest request) {
        String headerAuth = request.getHeader("Authorization");
        
        if (StringUtils.hasText(headerAuth) && headerAuth.startsWith("Bearer ")) {
            return headerAuth.substring(7);  // Remove "Bearer "
        }
        return null;
    }
}
```

### 📋 What Happens on Every Request:

```
1️⃣ REQUEST
   GET /api/tasks
   Headers: Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

2️⃣ FILTER EXECUTES
   ├─ parseJwt() → "eyJhbGciOiJIUzI1NiIs..."
   ├─ validateJwtToken() → ✓ Valid signature, not expired
   ├─ getUserNameFromJwtToken() → "john.doe"
   ├─ loadUserByUsername() → UserDetails from database
   ├─ Create UsernamePasswordAuthenticationToken
   └─ SecurityContextHolder.getContext().setAuthentication()

3️⃣ CONTROLLER EXECUTES
   @GetMapping("/api/tasks")
   public List<Task> getTasks() {
       // Authentication is already set!
       Authentication auth = SecurityContextHolder.getContext().getAuthentication();
       String username = auth.getName();  // "john.doe"
       // ...
   }
```

---

# 5️⃣ UserDetailsService - Bridge to Database 💾

## UserDetailsServiceImpl - Load Users from DB

```java
package com.nick.taskmanager.security;

@Service  // Make it a Spring bean
public class UserDetailsServiceImpl implements UserDetailsService {
    
    private final UserRepository userRepository;
    
    @Autowired
    public UserDetailsServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    // This method is CALLED AUTOMATICALLY by Spring Security during authentication
    @Override
    public UserDetails loadUserByUsername(String username) 
            throws UsernameNotFoundException {
        
        // 1. FETCH user from database
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> 
                new UsernameNotFoundException("User not found: " + username)
            );
        
        // 2. CONVERT your User entity to Spring's UserDetails
        Collection<GrantedAuthority> authorities = user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority(role))  // "ROLE_USER" → GrantedAuthority
            .collect(Collectors.toList());
        
        // 3. RETURN Spring Security User object
        return new org.springframework.security.core.userdetails.User(
            user.getUsername(),     // Username
            user.getPassword(),     // Hashed password
            user.isEnabled(),       // Account enabled?
            true,                  // Account non-expired
            true,                  // Credentials non-expired
            true,                  // Account non-locked
            authorities            // Roles
        );
    }
}
```

### 🔄 Authentication Flow:

```
1. LOGIN REQUEST
   ├─ Username: "john.doe"
   └─ Password: "mypassword"

2. AuthenticationManager.authenticate()
   ├─ Calls loadUserByUsername("john.doe")
   ├─ UserRepository finds user from DB
   │   └─ Hashed password: $2a$10$X7...
   ├─ PasswordEncoder.matches("mypassword", "$2a$10$X7...")
   │   └─ ✓ Passwords match!
   └─ Returns Authentication object

3. LOGIN SUCCESSFUL
   └─ JWT token generated and returned
```

---

# 6️⃣ User Entity & Repository 👤

## User.java - Database Entity

```java
package com.nick.taskmanager.model;

@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    private String username;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String password;  // Will store HASHED password!
    
    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "user_roles", 
                     joinColumns = @JoinColumn(name = "user_id"))
    @Column(name = "role")
    private Set<String> roles = new HashSet<>();
    
    private boolean enabled = true;
    
    // Constructors, Getters, Setters...
}
```

## UserRepository.java - Database Access

```java
package com.nick.taskmanager.repository;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Spring Data JPA will implement these automatically!
    
    Optional<User> findByUsername(String username);
    
    Optional<User> findByEmail(String email);
    
    Boolean existsByUsername(String username);
    
    Boolean existsByEmail(String email);
}
```

---

# 7️⃣ Auth Controller - Login & Register 🎮

## AuthController.java - Authentication Endpoints

```java
package com.nick.taskmanager.controller;

@RestController
@RequestMapping("/api/auth")
@Tag(name = "Authentication", description = "Login & Registration APIs")
public class AuthController {
    
    private final AuthenticationManager authenticationManager;
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtUtils jwtUtils;
    
    @Autowired
    public AuthController(...) {
        // Constructor injection
    }
    
    // 🔐 LOGIN - Authenticate existing user
    @PostMapping("/login")
    public ResponseEntity<?> authenticateUser(@RequestBody LoginRequest loginRequest) {
        
        // 1. Authenticate credentials
        Authentication authentication = authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                loginRequest.getUsername(),
                loginRequest.getPassword()
            )
        );
        
        // 2. Set authentication in context
        SecurityContextHolder.getContext().setAuthentication(authentication);
        
        // 3. Generate JWT token
        String jwt = jwtUtils.generateJwtToken(authentication);
        
        // 4. Get user details
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        User user = userRepository.findByUsername(userDetails.getUsername())
            .orElseThrow();
        
        // 5. Extract roles
        List<String> roles = userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList());
        
        // 6. Return token + user info
        return ResponseEntity.ok(
            new JwtResponse(jwt, user.getUsername(), user.getEmail(), roles)
        );
    }
    
    // 📝 REGISTER - Create new user
    @PostMapping("/signup")
    public ResponseEntity<?> registerUser(@RequestBody SignupRequest signupRequest) {
        
        // 1. Check if username exists
        if (userRepository.existsByUsername(signupRequest.getUsername())) {
            return ResponseEntity
                .badRequest()
                .body("Error: Username is already taken!");
        }
        
        // 2. Check if email exists
        if (userRepository.existsByEmail(signupRequest.getEmail())) {
            return ResponseEntity
                .badRequest()
                .body("Error: Email is already in use!");
        }
        
        // 3. Create new user (ENCODE PASSWORD!)
        User user = new User();
        user.setUsername(signupRequest.getUsername());
        user.setEmail(signupRequest.getEmail());
        user.setPassword(passwordEncoder.encode(signupRequest.getPassword()));
        user.setRoles(Set.of("ROLE_USER"));  // Default role
        
        // 4. Save to database
        userRepository.save(user);
        
        // 5. Return success
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body("User registered successfully!");
    }
}
```

---

# 8️⃣ DataInitializer - Bootstrap Data 🚀

```java
package com.nick.taskmanager.config;

@Component
public class DataInitializer implements CommandLineRunner {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    
    @Override
    public void run(String... args) throws Exception {
        
        // Create admin user if not exists
        if (!userRepository.existsByUsername("admin")) {
            
            User admin = new User();
            admin.setUsername("admin");
            admin.setEmail("admin@taskmanager.com");
            admin.setPassword(passwordEncoder.encode("admin123"));  // HASH!
            admin.setRoles(Set.of("ROLE_USER", "ROLE_ADMIN"));
            
            userRepository.save(admin);
            
            System.out.println("=================================");
            System.out.println("✅ Admin user created!");
            System.out.println("   Username: admin");
            System.out.println("   Password: admin123");
            System.out.println("=================================");
        }
    }
}
```

---

# 🎯 Complete Authentication Flow Summary

## 📋 Step-by-Step:

### **REGISTRATION:**
```
Client → POST /api/auth/signup
      → {username, email, password}
      ↓
Controller → Check if exists
          → Encode password (BCrypt)
          → Save to database
          ↓
Client ← 201 Created
      ← "User registered successfully!"
```

### **LOGIN:**
```
Client → POST /api/auth/login
      → {username, password}
      ↓
Controller → AuthenticationManager.authenticate()
          → UserDetailsService.loadUserByUsername()
          → PasswordEncoder.matches()
          → JwtUtils.generateJwtToken()
          ↓
Client ← 200 OK
      ← {token, username, email, roles}
      ↓
Browser stores token in localStorage
```

### **AUTHENTICATED REQUEST:**
```
Client → GET /api/tasks
      → Header: "Authorization: Bearer <token>"
      ↓
JwtAuthenticationFilter → Parse JWT
                      → Validate signature
                      → Check expiration
                      → Get username
                      → Load UserDetails
                      → Set Authentication in SecurityContext
      ↓
Controller → SecurityContext.getAuthentication()
          → auth.getName() = "john.doe"
          → Process request for this user
      ↓
Client ← 200 OK
      ← [{task1}, {task2}]
```

---

# 📁 application.properties

```properties
# JWT Configuration
app.jwtSecret=mySecretKeyForJwtTokenSigningAndVerification2024
app.jwtExpirationMs=86400000  # 24 hours in milliseconds

# Database
spring.datasource.url=jdbc:postgresql://localhost:5432/taskdb
spring.datasource.username=postgres
spring.datasource.password=postgres

# Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# Logging
logging.level.org.springframework.security=DEBUG
```

---

# 🎓 Key Takeaways

## ✅ **DO:**
- Always hash passwords with BCrypt
- Use DTOs for request/response (never expose entities)
- Keep JWT secret in properties/environment variables
- Make authentication stateless (no sessions)
- Set reasonable token expiration times
- Use `@PreAuthorize` for method-level security

## ❌ **DON'T:**
- Store plain text passwords
- Expose database entities directly to clients
- Put sensitive data in JWT payload
- Use sessions with JWT
- Hardcode credentials in production

## 📚 **Remember:**
1. **DTOs** = API contract
2. **JwtUtils** = Token factory
3. **JwtAuthenticationFilter** = Request interceptor
4. **UserDetailsService** = Database bridge
5. **SecurityConfig** = Rule book
6. **AuthController** = Authentication gate

---

# 🚀 Quick Test Commands

```bash
# 1. Register new user
curl -X POST http://localhost:8080/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"username":"john","email":"john@test.com","password":"password123"}'

# 2. Login
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"john","password":"password123"}'

# 3. Access protected endpoint with JWT
curl -X GET http://localhost:8080/api/tasks \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```
