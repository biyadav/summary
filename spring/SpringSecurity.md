# Spring Security Advanced Study Guide

This comprehensive guide covers Spring Security for Spring Boot 3+ applications, including core concepts, modern configuration patterns, Java code examples, best practices, common pitfalls, and troubleshooting techniques.

## Table of Contents
1. [Core Concepts](#1-core-concepts)
2. [Modern Configuration Patterns](#2-modern-configuration-patterns)
3. [Authentication Mechanisms](#3-authentication-mechanisms)
4. [Authorization and Method Security](#4-authorization-and-method-security)
5. [CSRF and CORS Configuration](#5-csrf-and-cors-configuration)
6. [Testing Security](#6-testing-security)
7. [Observability and Operations](#7-observability-and-operations)
8. [Best Practices](#8-best-practices)
9. [Common Pitfalls](#9-common-pitfalls)
10. [Troubleshooting Guide](#10-troubleshooting-guide)
11. [Advanced Topics](#11-advanced-topics)
12. [Resources](#12-resources)

---

## 1) Core Concepts

### 1.1 Authentication vs Authorization

**Authentication (AuthN)**: Verifying who the user is
- Results in an `Authentication` object stored in `SecurityContext`
- Examples: username/password, JWT tokens, OAuth2, SAML

**Authorization (AuthZ)**: Determining what an authenticated user can do
- Based on roles, authorities, or permissions
- Applied at URL level or method level

```java
// Authentication - who are you?
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
String username = auth.getName();

// Authorization - what can you do?
boolean canEdit = auth.getAuthorities().stream()
    .anyMatch(authority -> authority.getAuthority().equals("ROLE_EDITOR"));
```

**Explanation of Key Classes:**

- **`Authentication`**: Core interface representing the token for an authentication request or authenticated principal. Contains the principal (usually username), credentials (usually password), and authorities (roles/permissions).

- **`SecurityContextHolder`**: Central class that holds the security context of the current thread. Uses ThreadLocal storage by default, ensuring each thread has its own security context.

- **`GrantedAuthority`**: Represents an authority granted to an Authentication object. Typically roles (like ROLE_USER) or permissions (like READ_PRIVILEGES).

### 1.2 Security Filter Chain

Spring Security uses a chain of servlet filters to process requests:

```java
@Configuration
@EnableWebSecurity  // Enables Spring Security's web security support
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/public/**").permitAll()  // Allow anonymous access
                .anyRequest().authenticated()                // Require authentication for all other requests
            )
            .formLogin(withDefaults());  // Enable form-based login with default settings
        return http.build();
    }
}
```

**How the Security Filter Chain Works:**

1. **`@EnableWebSecurity`**: This annotation enables Spring Security's web security support and provides the Spring MVC integration. It imports the `WebSecurityConfiguration` class.

2. **`SecurityFilterChain`**: Defines a filter chain which is capable of being matched against an HttpServletRequest to decide whether it applies to that request. Spring Security 6+ replaced the deprecated `WebSecurityConfigurerAdapter` with this bean-based approach.

3. **`HttpSecurity`**: Allows configuring web-based security for specific http requests. It's a builder pattern that lets you customize various aspects like authentication, authorization, CSRF protection, etc.

4. **Request Processing Flow**:
   - Each incoming request passes through multiple filters in a specific order
   - Common filters include: SecurityContextPersistenceFilter, UsernamePasswordAuthenticationFilter, FilterSecurityInterceptor
   - Each filter has a specific responsibility (authentication, authorization, CSRF protection, etc.)

5. **`requestMatchers()`**: Specifies which requests this security configuration applies to. Uses Ant patterns or regex patterns.

6. **`permitAll()`**: Allows unrestricted access to the specified endpoints - no authentication required.

7. **`authenticated()`**: Requires the user to be authenticated (logged in) to access the resource.

### 1.3 SecurityContext and SecurityContextHolder

Thread-local storage for authentication information:

```java
@Component
public class SecurityService {
    
    public String getCurrentUsername() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        return auth != null ? auth.getName() : null;
    }
    
    public boolean hasRole(String role) {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        return auth.getAuthorities().stream()
            .anyMatch(authority -> authority.getAuthority().equals("ROLE_" + role));
    }
}
```

**Deep Dive into SecurityContext Components:**

**`SecurityContextHolder`**:
- **Purpose**: Central registry for storing security information about the current thread
- **Storage Strategy**: Uses ThreadLocal by default (MODE_THREADLOCAL), but can be configured for:
  - MODE_INHERITABLETHREADLOCAL: Child threads inherit parent's security context
  - MODE_GLOBAL: All threads share the same security context (rarely used)

**`SecurityContext`**:
- **Role**: Container for the Authentication object
- **Lifecycle**: Created when user authenticates, cleared when user logs out or session expires
- **Thread Safety**: Each thread has its own context, preventing interference between concurrent requests

**Key Methods Explained**:

1. **`getContext()`**: Retrieves the SecurityContext for the current thread
2. **`getAuthentication()`**: Gets the Authentication object from the context
3. **`getName()`**: Returns the principal's name (usually username)
4. **`getAuthorities()`**: Returns collection of granted authorities (roles/permissions)

**Important Notes**:
- Always check for null Authentication object - it can be null if user is not authenticated
- The security context is automatically propagated through the request lifecycle
- For async operations, you may need to manually propagate the security context

### 1.4 UserDetails and UserDetailsService

Interface for loading user-specific data:

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
        
        return org.springframework.security.core.userdetails.User.builder()
            .username(user.getUsername())
            .password(user.getPassword())
            .authorities(user.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
                .collect(Collectors.toList()))
            .accountExpired(!user.isAccountNonExpired())
            .accountLocked(!user.isAccountNonLocked())
            .credentialsExpired(!user.isCredentialsNonExpired())
            .disabled(!user.isEnabled())
            .build();
    }
}
```

**Understanding UserDetails and UserDetailsService:**

**`UserDetailsService` Interface**:
- **Purpose**: Core interface for loading user-specific data during authentication
- **Single Method**: `loadUserByUsername(String username)` - called by authentication providers
- **When Called**: During authentication process when user credentials need to be verified
- **Return Value**: UserDetails object containing user information and authorities

**`UserDetails` Interface**:
Represents core user information with these key methods:

1. **`getUsername()`**: Returns the username (unique identifier)
2. **`getPassword()`**: Returns the password used to authenticate the user
3. **`getAuthorities()`**: Returns the authorities (roles/permissions) granted to the user
4. **Account Status Methods**:
   - **`isAccountNonExpired()`**: Account hasn't expired
   - **`isAccountNonLocked()`**: Account isn't locked
   - **`isCredentialsNonExpired()`**: User's credentials haven't expired
   - **`isEnabled()`**: User account is enabled/active

**Implementation Details**:

- **Built-in User Class**: Spring provides `org.springframework.security.core.userdetails.User` implementation
- **Builder Pattern**: Modern approach for creating UserDetails objects with fluent API
- **Authority Mapping**: Converting domain roles to Spring Security authorities (usually prefixed with "ROLE_")
- **Exception Handling**: Throw `UsernameNotFoundException` when user doesn't exist

**Security Considerations**:
- Password should already be encoded when stored in database
- Don't expose sensitive information in exception messages
- Consider implementing account lockout after multiple failed attempts
- Validate that username parameter is not null or empty

---

## 2) Modern Configuration Patterns

Spring Security 6+ removes `WebSecurityConfigurerAdapter`. Configure using bean definitions.

### 2.1 Basic HTTP Basic Authentication

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity // Enables @PreAuthorize, @Secured, etc.
public class BasicSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable()) // Disable for APIs
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/actuator/health", "/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic(withDefaults());
        
        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails user = User.withUsername("user")
            .password(passwordEncoder().encode("password"))
            .roles("USER")
            .build();
        
        UserDetails admin = User.withUsername("admin")
            .password(passwordEncoder().encode("admin"))
            .roles("USER", "ADMIN")
            .build();
        
        return new InMemoryUserDetailsManager(user, admin);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
}
```

### 2.2 Form-based Authentication (MVC Applications)

```java
@Configuration
@EnableWebSecurity
public class FormSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/", "/home", "/css/**", "/js/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .loginProcessingUrl("/perform-login")
                .defaultSuccessUrl("/dashboard", true)
                .failureUrl("/login?error=true")
                .usernameParameter("email")
                .passwordParameter("pass")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login?logout")
                .invalidateHttpSession(true)
                .deleteCookies("JSESSIONID")
                .permitAll()
            )
            .rememberMe(remember -> remember
                .tokenValiditySeconds(1209600) // 14 days
                .userDetailsService(userDetailsService())
            );
        
        return http.build();
    }
}
```

### 2.3 JWT Resource Server (Stateless APIs)

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class JwtSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/actuator/health", "/api/public/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/products/**").hasAuthority("SCOPE_read")
                .requestMatchers(HttpMethod.POST, "/api/products/**").hasAuthority("SCOPE_write")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .decoder(jwtDecoder())
                    .jwtAuthenticationConverter(jwtAuthenticationConverter())
                )
            );
        
        return http.build();
    }

    @Bean
    public JwtDecoder jwtDecoder() {
        return JwtDecoders.fromIssuerLocation("https://your-auth-server.com");
    }

    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtGrantedAuthoritiesConverter authoritiesConverter = new JwtGrantedAuthoritiesConverter();
        authoritiesConverter.setAuthorityPrefix("SCOPE_");
        authoritiesConverter.setAuthoritiesClaimName("scope");

        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(authoritiesConverter);
        return converter;
    }
}
```

**JWT Resource Server Explained:**

**What is a JWT Resource Server?**
- Server that validates JWT tokens to authorize access to protected resources
- **Stateless**: No server-side session storage required  
- **Scalable**: Tokens contain all necessary information
- **Microservices**: Perfect for distributed architectures

**Key Components Breakdown:**

**1. Session Management Configuration:**
```java
.sessionManagement(session -> session
    .sessionCreationPolicy(SessionCreationPolicy.STATELESS))
```
- **`STATELESS`**: Never create or use HTTP sessions
- **Benefits**: Better performance, horizontal scaling, stateless architecture
- **Security**: Each request must contain valid JWT token

**2. OAuth2 Resource Server Setup:**
```java
.oauth2ResourceServer(oauth2 -> oauth2.jwt(...))
```
- **Purpose**: Configures JWT token validation
- **Filter**: Adds `BearerTokenAuthenticationFilter` to filter chain
- **Token Location**: Expects JWT in `Authorization: Bearer <token>` header

**3. JWT Decoder Configuration:**

**`JwtDecoder`**:
- **Role**: Validates and parses JWT tokens
- **Validation**: Checks signature, expiration, issuer, audience
- **fromIssuerLocation()**: Auto-discovers JWT configuration from OAuth2 server
  - Fetches JWKS (JSON Web Key Set) from `/.well-known/openid_configuration`
  - Caches public keys for signature verification

**4. JWT Authentication Converter:**

**`JwtAuthenticationConverter`**:
- **Purpose**: Converts JWT claims to Spring Security authorities
- **Custom Logic**: Maps JWT claims to `GrantedAuthority` objects

**`JwtGrantedAuthoritiesConverter`**:
- **`setAuthorityPrefix("SCOPE_")`**: Prefixes authorities with "SCOPE_"
- **`setAuthoritiesClaimName("scope")`**: Reads authorities from "scope" claim
- **Example**: JWT claim `"scope": "read write"` becomes authorities `[SCOPE_read, SCOPE_write]`

**Authority-Based Authorization:**
```java
.requestMatchers(HttpMethod.GET, "/api/products/**").hasAuthority("SCOPE_read")
.requestMatchers(HttpMethod.POST, "/api/products/**").hasAuthority("SCOPE_write")
```
- **Fine-grained**: Controls access based on HTTP method and path
- **OAuth2 Scopes**: Maps to standard OAuth2 scope-based authorization
- **Principle of Least Privilege**: Users get minimum required permissions

### 2.4 Multiple Security Filter Chains

```java
@Configuration
@EnableWebSecurity
public class MultipleSecurityConfig {

    @Bean
    @Order(1)
    public SecurityFilterChain apiFilterChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/api/**")
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(authz -> authz
                .anyRequest().authenticated())
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(withDefaults()));
        
        return http.build();
    }

    @Bean
    @Order(2)
    public SecurityFilterChain webFilterChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/**")
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/", "/home").permitAll()
                .anyRequest().authenticated())
            .formLogin(withDefaults());
        
        return http.build();
    }
}
```

---

## 3) Authentication Mechanisms

### 3.1 Database Authentication

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true)
    private String username;
    
    private String password;
    private boolean enabled = true;
    private boolean accountNonExpired = true;
    private boolean accountNonLocked = true;
    private boolean credentialsNonExpired = true;
    
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
    
    // getters and setters
}

@Service
@Transactional
public class UserDetailsServiceImpl implements UserDetailsService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
        
        return new CustomUserPrincipal(user);
    }
}

public class CustomUserPrincipal implements UserDetails {
    private User user;
    
    public CustomUserPrincipal(User user) {
        this.user = user;
    }
    
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
            .collect(Collectors.toList());
    }
    
    @Override
    public String getPassword() {
        return user.getPassword();
    }
    
    @Override
    public String getUsername() {
        return user.getUsername();
    }
    
    @Override
    public boolean isAccountNonExpired() {
        return user.isAccountNonExpired();
    }
    
    @Override
    public boolean isAccountNonLocked() {
        return user.isAccountNonLocked();
    }
    
    @Override
    public boolean isCredentialsNonExpired() {
        return user.isCredentialsNonExpired();
    }
    
    @Override
    public boolean isEnabled() {
        return user.isEnabled();
    }
    
    public User getUser() {
        return user;
    }
}
```

### 3.2 Custom Authentication Provider

```java
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = authentication.getName();
        String password = authentication.getCredentials().toString();
        
        UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        
        if (passwordEncoder.matches(password, userDetails.getPassword())) {
            return new UsernamePasswordAuthenticationToken(
                userDetails, password, userDetails.getAuthorities());
        } else {
            throw new BadCredentialsException("Invalid credentials");
        }
    }
    
    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(UsernamePasswordAuthenticationToken.class);
    }
}
```

### 3.3 OAuth2 Login

```java
@Configuration
@EnableWebSecurity
public class OAuth2SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/", "/login**", "/error**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
                .userInfoEndpoint(userInfo -> userInfo
                    .userService(customOAuth2UserService())
                )
            );
        
        return http.build();
    }

    @Bean
    public OAuth2UserService<OAuth2UserRequest, OAuth2User> customOAuth2UserService() {
        return new CustomOAuth2UserService();
    }
}

@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {
    
    private final OAuth2UserService<OAuth2UserRequest, OAuth2User> delegate = new DefaultOAuth2UserService();
    
    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2User oauth2User = delegate.loadUser(userRequest);
        
        // Custom logic to process the OAuth2 user
        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        String userNameAttributeName = userRequest.getClientRegistration()
            .getProviderDetails().getUserInfoEndpoint().getUserNameAttributeName();
        
        return new CustomOAuth2User(oauth2User, userNameAttributeName);
    }
}
```

---

## 4) Authorization and Method Security

### 4.1 URL-based Authorization

```java
@Configuration
@EnableWebSecurity
public class AuthorizationConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                // Public endpoints
                .requestMatchers("/", "/home", "/about").permitAll()
                .requestMatchers("/css/**", "/js/**", "/images/**").permitAll()
                
                // Role-based access
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/manager/**").hasAnyRole("ADMIN", "MANAGER")
                
                // Authority-based access
                .requestMatchers(HttpMethod.GET, "/api/users/**").hasAuthority("READ_USERS")
                .requestMatchers(HttpMethod.POST, "/api/users/**").hasAuthority("WRITE_USERS")
                
                // Expression-based access
                .requestMatchers("/api/users/{userId}/**")
                    .access(new WebExpressionAuthorizationManager(
                        "#userId == authentication.name or hasRole('ADMIN')"))
                
                .anyRequest().authenticated()
            );
        
        return http.build();
    }
}
```

### 4.2 Method-level Security

```java
@Configuration
@EnableMethodSecurity(
    prePostEnabled = true,    // Enables @PreAuthorize and @PostAuthorize
    securedEnabled = true,    // Enables @Secured annotation  
    jsr250Enabled = true      // Enables @RolesAllowed annotation
)
public class MethodSecurityConfig {
    
    @Bean
    public MethodSecurityExpressionHandler methodSecurityExpressionHandler() {
        DefaultMethodSecurityExpressionHandler handler = new DefaultMethodSecurityExpressionHandler();
        handler.setPermissionEvaluator(new CustomPermissionEvaluator());
        return handler;
    }
}

@Service
public class UserService {
    
    @PreAuthorize("hasRole('ADMIN')")
    public List<User> getAllUsers() {
        // Implementation
        return new ArrayList<>();
    }
    
    @PreAuthorize("hasRole('ADMIN') or authentication.name == #username")
    public User getUserByUsername(String username) {
        // Implementation
        return new User();
    }
    
    @PreAuthorize("@userService.isOwner(authentication.name, #userId)")
    public void updateUser(Long userId, User user) {
        // Implementation
    }
    
    @PostAuthorize("returnObject.username == authentication.name or hasRole('ADMIN')")
    public User findUserById(Long id) {
        // Implementation
        return new User();
    }
    
    @Secured({"ROLE_ADMIN", "ROLE_MANAGER"})
    public void deleteUser(Long userId) {
        // Implementation
    }
    
    @RolesAllowed("ADMIN")
    public void adminOnlyMethod() {
        // Implementation
    }
    
    public boolean isOwner(String username, Long userId) {
        // Custom logic to check ownership
        return true;
    }
}
```

**Method Security Explained:**

**What is Method Security?**
- Fine-grained authorization at the method level using annotations
- Complements URL-based security for business logic protection
- Works with AOP (Aspect-Oriented Programming) using proxies
- Evaluated before or after method execution

**Key Annotations:**

**1. `@PreAuthorize`** - Evaluated Before Method Execution:
```java
@PreAuthorize("hasRole('ADMIN')")  // Simple role check
@PreAuthorize("hasRole('ADMIN') or authentication.name == #username")  // Complex expression
@PreAuthorize("@userService.isOwner(authentication.name, #userId)")     // Custom method call
```
- **When**: Checks permission before method runs
- **SpEL Support**: Full Spring Expression Language support
- **Method Parameters**: Access via `#parameterName`
- **Authentication**: Access current user via `authentication`

**2. `@PostAuthorize`** - Evaluated After Method Execution:
```java
@PostAuthorize("returnObject.username == authentication.name or hasRole('ADMIN')")
```
- **When**: Checks permission after method runs
- **Return Object**: Access method result via `returnObject`
- **Use Case**: Filter results based on returned data
- **Performance**: Method still executes, but result may be denied

**3. `@Secured`** - Simple Role-Based:
```java
@Secured({"ROLE_ADMIN", "ROLE_MANAGER"})  // Multiple roles (OR logic)
```
- **Simple**: Only supports role checking
- **No SpEL**: Cannot use complex expressions
- **Multiple Roles**: User needs ANY of the specified roles

**4. `@RolesAllowed`** - JSR-250 Standard:
```java
@RolesAllowed("ADMIN")  // Note: No "ROLE_" prefix
```
- **Standard**: Part of JSR-250 specification
- **Role Names**: Uses role names without "ROLE_" prefix
- **Simple**: Like @Secured but industry standard

**Configuration Details:**

**`@EnableMethodSecurity` Parameters:**
- **`prePostEnabled = true`**: Activates @PreAuthorize/@PostAuthorize
- **`securedEnabled = true`**: Activates @Secured annotation
- **`jsr250Enabled = true`**: Activates @RolesAllowed annotation

**Method Security Expression Handler:**
- **Purpose**: Processes SpEL expressions in security annotations
- **Custom Evaluators**: Add domain-specific permission logic
- **Built-in Variables**: `authentication`, `principal`, method parameters

**Spring Expression Language (SpEL) in Security:**

**Built-in Security Expressions:**
- `hasRole('ROLE_NAME')` - Check if user has specific role
- `hasAnyRole('ROLE1', 'ROLE2')` - Check if user has any of the roles
- `hasAuthority('PERMISSION')` - Check specific authority
- `hasAnyAuthority('PERM1', 'PERM2')` - Check any authority
- `authentication.name` - Current username
- `principal` - Current UserDetails object

**Method Parameter Access:**
- `#parameterName` - Access method parameters by name
- `#root.args[0]` - Access first parameter by index
- `#root.methodName` - Access method name

**Custom Method Calls:**
```java
@PreAuthorize("@securityService.canAccess(authentication, #resourceId)")
```
- Call beans using `@beanName.methodName()`
- Pass authentication context and method parameters
- Implement complex business rules

**Important Considerations:**

**Proxy Limitations:**
- Method security uses AOP proxies
- Self-invocation bypasses security (calling `this.securedMethod()`)
- Solution: Inject self or use different service

**Performance:**
- @PreAuthorize: Fails fast, method doesn't execute
- @PostAuthorize: Method executes, then checks result
- Complex expressions evaluated on every call

**Testing:**
- Use `@WithMockUser` for testing secured methods
- Test both authorized and unauthorized scenarios
- Verify method parameters are correctly accessed in expressions
```

### 4.3 Custom Permission Evaluator

```java
@Component
public class CustomPermissionEvaluator implements PermissionEvaluator {
    
    @Autowired
    private UserService userService;
    
    @Override
    public boolean hasPermission(Authentication authentication, Object targetDomainObject, Object permission) {
        if (authentication == null || targetDomainObject == null || !(permission instanceof String)) {
            return false;
        }
        
        String targetType = targetDomainObject.getClass().getSimpleName().toLowerCase();
        return hasPrivilege(authentication, targetType, permission.toString(), targetDomainObject);
    }
    
    @Override
    public boolean hasPermission(Authentication authentication, Serializable targetId, String targetType, Object permission) {
        if (authentication == null || targetType == null || !(permission instanceof String)) {
            return false;
        }
        
        return hasPrivilege(authentication, targetType, permission.toString(), targetId);
    }
    
    private boolean hasPrivilege(Authentication auth, String targetType, String permission, Object target) {
        String username = auth.getName();
        
        // Custom permission logic
        if ("user".equals(targetType) && "edit".equals(permission)) {
            if (target instanceof User) {
                User user = (User) target;
                return user.getUsername().equals(username) || 
                       auth.getAuthorities().stream()
                           .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"));
            }
        }
        
        return false;
    }
}

// Usage in service methods
@Service
public class DocumentService {
    
    @PreAuthorize("hasPermission(#document, 'read')")
    public Document getDocument(Document document) {
        return document;
    }
    
    @PreAuthorize("hasPermission(#documentId, 'Document', 'write')")
    public void updateDocument(Long documentId, Document document) {
        // Implementation
    }
}
```

---

## 5) CSRF and CORS Configuration

### 5.1 CSRF Protection

```java
@Configuration
@EnableWebSecurity
public class CsrfConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf
                .ignoringRequestMatchers("/api/**") // Disable for API endpoints
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                .csrfTokenRequestHandler(new CsrfTokenRequestAttributeHandler())
            )
            .authorizeHttpRequests(authz -> authz
                .anyRequest().authenticated()
            );
        
        return http.build();
    }
}

// Controller to expose CSRF token to frontend
@RestController
public class CsrfController {
    
    @GetMapping("/csrf-token")
    public CsrfToken csrfToken(CsrfToken token) {
        return token;
    }
}

// Custom CSRF token repository (if needed)
@Component
public class CustomCsrfTokenRepository implements CsrfTokenRepository {
    
    @Override
    public CsrfToken generateToken(HttpServletRequest request) {
        return new DefaultCsrfToken("X-CSRF-TOKEN", "_csrf", UUID.randomUUID().toString());
    }
    
    @Override
    public void saveToken(CsrfToken token, HttpServletRequest request, HttpServletResponse response) {
        // Custom save logic (e.g., to database)
    }
    
    @Override
    public CsrfToken loadToken(HttpServletRequest request) {
        // Custom load logic
        return null;
    }
}
```

### 5.2 CORS Configuration

```java
@Configuration
@EnableWebSecurity
public class CorsConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(authz -> authz
                .anyRequest().authenticated()
            );
        
        return http.build();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        
        // Allow specific origins
        configuration.setAllowedOrigins(Arrays.asList(
            "http://localhost:3000",
            "https://myapp.com",
            "https://staging.myapp.com"
        ));
        
        // Allow specific methods
        configuration.setAllowedMethods(Arrays.asList(
            "GET", "POST", "PUT", "DELETE", "OPTIONS", "PATCH"
        ));
        
        // Allow specific headers
        configuration.setAllowedHeaders(Arrays.asList(
            "Authorization",
            "Content-Type",
            "X-Requested-With",
            "Accept",
            "Origin",
            "Access-Control-Request-Method",
            "Access-Control-Request-Headers"
        ));
        
        // Allow credentials
        configuration.setAllowCredentials(true);
        
        // Set max age for preflight requests
        configuration.setMaxAge(3600L);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        
        return source;
    }
}

// Alternative: Using @CrossOrigin annotation
@RestController
@CrossOrigin(
    origins = {"http://localhost:3000", "https://myapp.com"},
    methods = {RequestMethod.GET, RequestMethod.POST},
    allowCredentials = "true"
)
public class ApiController {
    
    @GetMapping("/api/data")
    public ResponseEntity<String> getData() {
        return ResponseEntity.ok("Data");
    }
}
```

---

## 6) Testing Security

### 6.1 MockMvc Security Testing

```java
@WebMvcTest(UserController.class)
@Import(SecurityConfig.class)
class UserControllerSecurityTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void shouldRequireAuthentication() throws Exception {
        mockMvc.perform(get("/api/users"))
                .andExpect(status().isUnauthorized());
    }
    
    @Test
    @WithMockUser
    void shouldAllowAuthenticatedUser() throws Exception {
        mockMvc.perform(get("/api/users"))
                .andExpect(status().isOk());
    }
    
    @Test
    @WithMockUser(roles = "ADMIN")
    void shouldAllowAdminAccess() throws Exception {
        mockMvc.perform(get("/api/admin/users"))
                .andExpect(status().isOk());
    }
    
    @Test
    @WithMockUser(roles = "USER")
    void shouldDenyUserAccessToAdmin() throws Exception {
        mockMvc.perform(get("/api/admin/users"))
                .andExpected(status().isForbidden());
    }
    
    @Test
    void shouldHandleBasicAuth() throws Exception {
        mockMvc.perform(get("/api/users")
                .with(httpBasic("user", "password")))
                .andExpect(status().isOk());
    }
    
    @Test
    @WithUserDetails("admin@example.com")
    void shouldLoadUserFromUserDetailsService() throws Exception {
        mockMvc.perform(get("/api/profile"))
                .andExpect(status().isOk());
    }
}

// Custom security annotation
@Retention(RetentionPolicy.RUNTIME)
@WithSecurityContext(factory = WithMockCustomUserSecurityContextFactory.class)
public @interface WithMockCustomUser {
    String username() default "user";
    String[] roles() default {"USER"};
}

public class WithMockCustomUserSecurityContextFactory 
        implements WithSecurityContextFactory<WithMockCustomUser> {
    
    @Override
    public SecurityContext createSecurityContext(WithMockCustomUser customUser) {
        SecurityContext context = SecurityContextHolder.createEmptyContext();
        
        CustomUserPrincipal principal = new CustomUserPrincipal(
            createUser(customUser.username(), customUser.roles())
        );
        
        Authentication auth = new UsernamePasswordAuthenticationToken(
            principal, "password", principal.getAuthorities()
        );
        
        context.setAuthentication(auth);
        return context;
    }
    
    private User createUser(String username, String[] roles) {
        // Create user with specified roles
        return new User();
    }
}
```

### 6.2 JWT Testing

```java
@SpringBootTest
@AutoConfigureMockMvc
class JwtSecurityTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void shouldRejectRequestWithoutToken() throws Exception {
        mockMvc.perform(get("/api/protected"))
                .andExpect(status().isUnauthorized());
    }
    
    @Test
    void shouldAcceptValidJwtToken() throws Exception {
        mockMvc.perform(get("/api/protected")
                .with(jwt().jwt(builder -> builder
                    .claim("sub", "user")
                    .claim("scope", "read write")
                )))
                .andExpect(status().isOk());
    }
    
    @Test
    void shouldRejectExpiredToken() throws Exception {
        mockMvc.perform(get("/api/protected")
                .with(jwt().jwt(builder -> builder
                    .claim("sub", "user")
                    .expiresAt(Instant.now().minusSeconds(3600))
                )))
                .andExpect(status().isUnauthorized());
    }
    
    @Test
    void shouldEnforceScopeBasedAccess() throws Exception {
        mockMvc.perform(post("/api/users")
                .with(jwt().jwt(builder -> builder
                    .claim("sub", "user")
                    .claim("scope", "read") // Missing write scope
                ))
                .contentType(MediaType.APPLICATION_JSON)
                .content("{}"))
                .andExpect(status().isForbidden());
    }
}
```

### 6.3 Method Security Testing

```java
@SpringBootTest
@TestMethodSecurity
class MethodSecurityTest {
    
    @Autowired
    private UserService userService;
    
    @Test
    @WithMockUser(roles = "ADMIN")
    void adminCanAccessAllUsers() {
        List<User> users = userService.getAllUsers();
        assertThat(users).isNotNull();
    }
    
    @Test
    @WithMockUser(roles = "USER")
    void userCannotAccessAllUsers() {
        assertThatThrownBy(() -> userService.getAllUsers())
                .isInstanceOf(AccessDeniedException.class);
    }
    
    @Test
    @WithMockUser(username = "john")
    void userCanAccessOwnProfile() {
        User user = userService.getUserByUsername("john");
        assertThat(user).isNotNull();
    }
    
    @Test
    @WithMockUser(username = "john")
    void userCannotAccessOtherProfile() {
        assertThatThrownBy(() -> userService.getUserByUsername("jane"))
                .isInstanceOf(AccessDeniedException.class);
    }
}
```

---

## 7) Observability and Operations

### 7.1 Security Events and Auditing

```java
@Configuration
@EnableJpaAuditing
public class AuditConfig {
    
    @Bean
    public AuditorAware<String> auditorProvider() {
        return new SecurityAuditorAware();
    }
}

@Component
public class SecurityAuditorAware implements AuditorAware<String> {
    
    @Override
    public Optional<String> getCurrentAuditor() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        
        if (authentication == null || !authentication.isAuthenticated()) {
            return Optional.empty();
        }
        
        return Optional.of(authentication.getName());
    }
}

@EventListener
@Component
public class AuthenticationEventListener {
    
    private static final Logger logger = LoggerFactory.getLogger(AuthenticationEventListener.class);
    
    @EventListener
    public void onAuthenticationSuccess(AuthenticationSuccessEvent event) {
        logger.info("Authentication successful for user: {}", 
                   event.getAuthentication().getName());
    }
    
    @EventListener
    public void onAuthenticationFailure(AbstractAuthenticationFailureEvent event) {
        logger.warn("Authentication failed for user: {}, reason: {}", 
                   event.getAuthentication().getName(),
                   event.getException().getMessage());
    }
    
    @EventListener
    public void onAuthorizationFailure(AuthorizationDeniedEvent event) {
        logger.warn("Authorization denied for user: {}, resource: {}", 
                   event.getAuthentication().getName(),
                   event.getAuthorizationDecision());
    }
}
```

### 7.2 Security Actuator Endpoints

```java
@Configuration
public class ActuatorSecurityConfig {
    
    @Bean
    @Order(1)
    public SecurityFilterChain actuatorFilterChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/actuator/**")
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/actuator/health", "/actuator/info").permitAll()
                .requestMatchers("/actuator/**").hasRole("ACTUATOR")
                .anyRequest().authenticated()
            )
            .httpBasic(withDefaults());
        
        return http.build();
    }
}

// Custom health indicator
@Component
public class SecurityHealthIndicator implements HealthIndicator {
    
    @Override
    public Health health() {
        SecurityContext context = SecurityContextHolder.getContext();
        Authentication auth = context.getAuthentication();
        
        if (auth != null && auth.isAuthenticated()) {
            return Health.up()
                    .withDetail("authenticated", true)
                    .withDetail("user", auth.getName())
                    .build();
        } else {
            return Health.down()
                    .withDetail("authenticated", false)
                    .build();
        }
    }
}
```

### 7.3 Security Metrics

```java
@Component
public class SecurityMetrics {
    
    private final MeterRegistry meterRegistry;
    private final Counter authenticationSuccessCounter;
    private final Counter authenticationFailureCounter;
    private final Timer authenticationTimer;
    
    public SecurityMetrics(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.authenticationSuccessCounter = Counter.builder("authentication.success")
                .description("Number of successful authentications")
                .register(meterRegistry);
        this.authenticationFailureCounter = Counter.builder("authentication.failure")
                .description("Number of failed authentications")
                .register(meterRegistry);
        this.authenticationTimer = Timer.builder("authentication.duration")
                .description("Authentication processing time")
                .register(meterRegistry);
    }
    
    @EventListener
    public void onAuthenticationSuccess(AuthenticationSuccessEvent event) {
        authenticationSuccessCounter.increment(
            Tags.of("type", event.getAuthentication().getClass().getSimpleName())
        );
    }
    
    @EventListener
    public void onAuthenticationFailure(AbstractAuthenticationFailureEvent event) {
        authenticationFailureCounter.increment(
            Tags.of("reason", event.getException().getClass().getSimpleName())
        );
    }
}
```

---

## 8) Best Practices

### 8.1 Password Security

```java
@Configuration
public class PasswordSecurityConfig {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        // Use BCrypt with strength 12 for production
        return new BCryptPasswordEncoder(12);
    }
    
    @Bean
    public PasswordValidator passwordValidator() {
        return new PasswordValidator(Arrays.asList(
            new LengthRule(8, 128),
            new CharacterRule(EnglishCharacterData.UpperCase, 1),
            new CharacterRule(EnglishCharacterData.LowerCase, 1),
            new CharacterRule(EnglishCharacterData.Digit, 1),
            new CharacterRule(EnglishCharacterData.Special, 1),
            new WhitespaceRule(),
            new IllegalSequenceRule(EnglishSequenceData.Alphabetical, 5, false),
            new IllegalSequenceRule(EnglishSequenceData.Numerical, 5, false),
            new RepeatCharacterRegexRule(4)
        ));
    }
}

@Service
public class PasswordService {
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Autowired
    private PasswordValidator passwordValidator;
    
    public String encodePassword(String rawPassword) {
        validatePassword(rawPassword);
        return passwordEncoder.encode(rawPassword);
    }
    
    public boolean matches(String rawPassword, String encodedPassword) {
        return passwordEncoder.matches(rawPassword, encodedPassword);
    }
    
    private void validatePassword(String password) {
        RuleResult result = passwordValidator.validate(new PasswordData(password));
        if (!result.isValid()) {
            throw new IllegalArgumentException(
                String.join(", ", passwordValidator.getMessages(result))
            );
        }
    }
}
```

**Password Security Deep Dive:**

**Why Password Encoding Matters:**
- **Never store plain text passwords**: Fundamental security requirement
- **Hashing vs Encryption**: Passwords should be hashed (one-way), not encrypted (reversible)
- **Salt**: Prevents rainbow table attacks by adding random data before hashing
- **Cost Factor**: Controls how computationally expensive hashing is

**Password Encoder Types:**

**1. BCryptPasswordEncoder (Recommended):**
```java
new BCryptPasswordEncoder(12)  // Strength parameter (4-31)
```
- **Algorithm**: Blowfish-based, designed for password hashing
- **Adaptive**: Cost can be increased as hardware improves
- **Salt**: Automatically generates random salt for each password
- **Strength**: 10-12 for production (higher = slower but more secure)
- **Time**: ~100ms at strength 10, doubles with each increment

**2. DelegatingPasswordEncoder (Default in Spring Security):**
```java
PasswordEncoderFactories.createDelegatingPasswordEncoder()
```
- **Format**: `{algorithm}encodedPassword` (e.g., `{bcrypt}$2a$10$...`)
- **Flexibility**: Supports multiple algorithms for legacy passwords
- **Migration**: Allows upgrading password algorithms over time
- **Default**: Uses BCrypt for new passwords

**Password Validation Rules Explained:**

**Length Requirements:**
```java
new LengthRule(8, 128)  // Minimum 8, maximum 128 characters
```
- **Minimum**: NIST recommends 8+ characters
- **Maximum**: Prevent DoS attacks from extremely long passwords

**Character Requirements:**
```java
new CharacterRule(EnglishCharacterData.UpperCase, 1)    // At least 1 uppercase
new CharacterRule(EnglishCharacterData.LowerCase, 1)    // At least 1 lowercase  
new CharacterRule(EnglishCharacterData.Digit, 1)       // At least 1 digit
new CharacterRule(EnglishCharacterData.Special, 1)     // At least 1 special char
```
- **Complexity**: Increases password entropy
- **Balance**: Avoid making passwords too complex to remember

**Security Rules:**
```java
new WhitespaceRule()  // No whitespace characters
new IllegalSequenceRule(EnglishSequenceData.Alphabetical, 5, false)  // No "abcde"
new IllegalSequenceRule(EnglishSequenceData.Numerical, 5, false)     // No "12345"
new RepeatCharacterRegexRule(4)  // No more than 4 consecutive identical chars
```
- **Sequences**: Prevent common patterns like "12345", "abcde"
- **Repetition**: Avoid "aaaa" or "1111" patterns
- **Whitespace**: Prevent accidental spaces

**PasswordService Implementation:**

**Key Methods:**

**1. `encodePassword(String rawPassword)`:**
- **Validation First**: Check password meets complexity requirements
- **Encoding**: Transform plain text to secure hash
- **Returns**: Encoded password ready for database storage

**2. `matches(String rawPassword, String encodedPassword)`:**
- **Purpose**: Verify user input against stored hash during login
- **Security**: Never decode the stored password
- **Process**: Hash input with same salt, compare results

**3. `validatePassword(String password)`:**
- **Early Validation**: Check rules before encoding
- **User Feedback**: Provide specific error messages
- **Fail Fast**: Reject weak passwords immediately

**Security Best Practices:**

**Storage:**
- Store only encoded passwords in database
- Never log plain text passwords
- Use HTTPS to protect passwords in transit

**Configuration:**
- Use appropriate cost factor for your hardware
- Regularly review and update password policies
- Consider implementing password history (prevent reuse)

**Monitoring:**
- Log failed password attempts
- Implement account lockout after multiple failures
- Monitor for brute force attacks

**Password Policy Examples:**

**Conservative (High Security):**
```java
new LengthRule(12, 128),
new CharacterRule(EnglishCharacterData.UpperCase, 2),
new CharacterRule(EnglishCharacterData.LowerCase, 2), 
new CharacterRule(EnglishCharacterData.Digit, 2),
new CharacterRule(EnglishCharacterData.Special, 2)
```

**Balanced (Most Applications):**
```java
new LengthRule(8, 64),
new CharacterRule(EnglishCharacterData.UpperCase, 1),
new CharacterRule(EnglishCharacterData.LowerCase, 1),
new CharacterRule(EnglishCharacterData.Digit, 1)
// Special chars optional for better UX
```

**Common Mistakes to Avoid:**
- ❌ Using MD5 or SHA-1 for passwords
- ❌ Implementing custom password hashing
- ❌ Storing passwords in reversible encryption
- ❌ Using same salt for all passwords
- ❌ Making password rules too restrictive
- ❌ Not handling password validation errors gracefully

### 8.2 Session Management

```java
@Configuration
public class SessionConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
                .maximumSessions(1)
                .maxSessionsPreventsLogin(false) // Allow new login to kick out old session
                .sessionRegistry(sessionRegistry())
                .and()
                .sessionFixation().migrateSession()
                .invalidSessionUrl("/login?expired")
            )
            .rememberMe(remember -> remember
                .tokenValiditySeconds(1209600) // 14 days
                .userDetailsService(userDetailsService())
                .tokenRepository(persistentTokenRepository())
            );
        
        return http.build();
    }
    
    @Bean
    public SessionRegistry sessionRegistry() {
        return new SessionRegistryImpl();
    }
    
    @Bean
    public PersistentTokenRepository persistentTokenRepository() {
        JdbcTokenRepositoryImpl tokenRepository = new JdbcTokenRepositoryImpl();
        tokenRepository.setDataSource(dataSource);
        return tokenRepository;
    }
}

// Session timeout configuration
@Component
public class SessionTimeoutHandler {
    
    @EventListener
    public void onSessionCreated(HttpSessionCreatedEvent event) {
        event.getSession().setMaxInactiveInterval(1800); // 30 minutes
    }
    
    @EventListener
    public void onSessionDestroyed(HttpSessionDestroyedEvent event) {
        // Clean up session-related resources
    }
}
```

### 8.3 Secure Headers

```java
@Configuration
public class SecurityHeadersConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .headers(headers -> headers
                .frameOptions().deny()
                .contentTypeOptions().and()
                .httpStrictTransportSecurity(hstsConfig -> hstsConfig
                    .maxAgeInSeconds(31536000)
                    .includeSubdomains(true)
                    .preload(true)
                )
                .contentSecurityPolicy("default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'")
                .referrerPolicy(ReferrerPolicy.STRICT_ORIGIN_WHEN_CROSS_ORIGIN)
                .permissionsPolicy(policy -> policy
                    .policy("geolocation", "none")
                    .policy("camera", "none")
                    .policy("microphone", "none")
                )
            );
        
        return http.build();
    }
}
```

---

## 9) Common Pitfalls

### 9.1 Authentication vs Authorization Confusion

**Problem**: Mixing up authentication and authorization logic.

```java
// ❌ Wrong - checking authentication in authorization logic
@PreAuthorize("authentication != null")
public void someMethod() { }

// ✅ Correct - use proper authorization checks
@PreAuthorize("hasRole('USER')")
public void someMethod() { }
```

### 9.2 Incorrect Password Encoding

**Problem**: Not encoding passwords properly or using weak encoders.

```java
// ❌ Wrong - storing plain text passwords
user.setPassword(rawPassword);

// ❌ Wrong - using weak encoding
user.setPassword(Md5PasswordEncoder.encode(rawPassword));

// ✅ Correct - using strong password encoder
user.setPassword(passwordEncoder.encode(rawPassword));
```

### 9.3 CSRF Token Issues

**Problem**: CSRF protection interfering with AJAX requests.

```java
// ❌ Wrong - disabling CSRF for everything
http.csrf().disable()

// ✅ Correct - selective CSRF configuration
http.csrf(csrf -> csrf
    .ignoringRequestMatchers("/api/**") // Only for stateless APIs
    .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
)
```

### 9.4 Method Security Not Working

**Problem**: Method security annotations not being processed.

```java
// ❌ Wrong - missing @EnableMethodSecurity
@Configuration
public class SecurityConfig { }

// ✅ Correct - enable method security
@Configuration
@EnableMethodSecurity
public class SecurityConfig { }

// ❌ Wrong - self-invocation bypasses security
@Service
public class UserService {
    public void publicMethod() {
        this.secureMethod(); // Security bypassed!
    }
    
    @PreAuthorize("hasRole('ADMIN')")
    public void secureMethod() { }
}

// ✅ Correct - call through proxy
@Service
public class UserService {
    @Autowired
    private UserService self; // Inject self to get proxy
    
    public void publicMethod() {
        self.secureMethod(); // Security enforced
    }
    
    @PreAuthorize("hasRole('ADMIN')")
    public void secureMethod() { }
}
```

### 9.5 Session Fixation

**Problem**: Not protecting against session fixation attacks.

```java
// ❌ Wrong - not handling session fixation
http.sessionManagement(session -> session
    .sessionFixation().none()
);

// ✅ Correct - migrate session on authentication
http.sessionManagement(session -> session
    .sessionFixation().migrateSession()
);
```

### 9.6 Exposing Sensitive Information

**Problem**: Leaking sensitive information in error messages or logs.

```java
// ❌ Wrong - exposing user existence
@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    User user = userRepository.findByUsername(username);
    if (user == null) {
        throw new UsernameNotFoundException("User " + username + " not found");
    }
    return new CustomUserPrincipal(user);
}

// ✅ Correct - generic error message
@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    User user = userRepository.findByUsername(username);
    if (user == null) {
        throw new UsernameNotFoundException("Invalid credentials");
    }
    return new CustomUserPrincipal(user);
}
```

---

## 10) Troubleshooting Guide

### 10.1 Authentication Issues

**Symptom**: Users cannot authenticate

**Debugging Steps**:
1. Enable security debug logging
2. Check UserDetailsService implementation
3. Verify password encoding
4. Check authentication provider configuration

```properties
# Enable debug logging
logging.level.org.springframework.security=DEBUG
logging.level.org.springframework.security.web.authentication=TRACE
```

```java
// Debug authentication process
@Component
public class AuthenticationDebugListener {
    
    private static final Logger logger = LoggerFactory.getLogger(AuthenticationDebugListener.class);
    
    @EventListener
    public void onAuthenticationSuccess(AuthenticationSuccessEvent event) {
        logger.debug("Authentication successful: {}", event.getAuthentication());
    }
    
    @EventListener
    public void onAuthenticationFailure(AbstractAuthenticationFailureEvent event) {
        logger.debug("Authentication failed: {}", event.getException().getMessage());
    }
}
```

### 10.2 Authorization Issues

**Symptom**: Authenticated users get 403 Forbidden

**Debugging Steps**:
1. Check role/authority mapping
2. Verify security matcher order
3. Check method security configuration
4. Use access decision debugging

```java
// Debug access decisions
@Component
public class AccessDecisionDebugger {
    
    @EventListener
    public void onAccessDecision(AuthorizationEvent event) {
        logger.debug("Access decision for {}: {}", 
                    event.getAuthentication().getName(),
                    event.getAuthorizationDecision());
    }
}
```

### 10.3 CORS Issues

**Symptom**: Browser blocks requests due to CORS

**Common Solutions**:
```java
// Ensure CORS is configured before authentication
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .cors(withDefaults()) // Configure CORS first
        .authorizeHttpRequests(authz -> authz
            .requestMatchers(HttpMethod.OPTIONS).permitAll() // Allow preflight
            .anyRequest().authenticated()
        );
    
    return http.build();
}
```

### 10.4 JWT Issues

**Symptom**: JWT tokens not working

**Debugging Steps**:
1. Verify JWT decoder configuration
2. Check token expiration
3. Validate issuer and audience
4. Check authority mapping

```java
// Debug JWT processing
@Component
public class JwtDebugger {
    
    @EventListener
    public void onJwtDecoded(JwtDecodedEvent event) {
        Jwt jwt = event.getJwt();
        logger.debug("JWT decoded - Subject: {}, Expires: {}, Claims: {}", 
                    jwt.getSubject(), jwt.getExpiresAt(), jwt.getClaims());
    }
}
```

### 10.5 Performance Issues

**Symptom**: Security processing is slow

**Optimization Techniques**:
```java
// Cache user details
@Service
@Transactional(readOnly = true)
public class CachingUserDetailsService implements UserDetailsService {
    
    @Cacheable(value = "users", key = "#username")
    @Override
    public UserDetails loadUserByUsername(String username) {
        // Expensive database operation
        return loadUserFromDatabase(username);
    }
    
    @CacheEvict(value = "users", key = "#username")
    public void evictUser(String username) {
        // Evict user from cache when updated
    }
}

// Optimize database queries
@Entity
public class User {
    @ManyToMany(fetch = FetchType.EAGER) // Careful with EAGER loading
    @JoinTable(name = "user_roles")
    private Set<Role> roles;
}
```

---

## 11) Advanced Topics

### 11.1 Custom Security Filter

```java
@Component
public class ApiKeyAuthenticationFilter extends OncePerRequestFilter {
    
    private static final String API_KEY_HEADER = "X-API-Key";
    
    @Autowired
    private ApiKeyService apiKeyService;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                  HttpServletResponse response,
                                  FilterChain filterChain) throws ServletException, IOException {
        
        String apiKey = request.getHeader(API_KEY_HEADER);
        
        if (apiKey != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            try {
                UserDetails userDetails = apiKeyService.loadUserByApiKey(apiKey);
                if (userDetails != null) {
                    ApiKeyAuthenticationToken authentication = 
                        new ApiKeyAuthenticationToken(userDetails, apiKey, userDetails.getAuthorities());
                    authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    SecurityContextHolder.getContext().setAuthentication(authentication);
                }
            } catch (Exception e) {
                logger.debug("API key authentication failed", e);
            }
        }
        
        filterChain.doFilter(request, response);
    }
}

public class ApiKeyAuthenticationToken extends AbstractAuthenticationToken {
    
    private final Object principal;
    private Object credentials;
    
    public ApiKeyAuthenticationToken(Object principal, Object credentials, 
                                   Collection<? extends GrantedAuthority> authorities) {
        super(authorities);
        this.principal = principal;
        this.credentials = credentials;
        setAuthenticated(true);
    }
    
    @Override
    public Object getCredentials() {
        return credentials;
    }
    
    @Override
    public Object getPrincipal() {
        return principal;
    }
}
```

### 11.2 Dynamic Role-Based Access Control

```java
@Entity
public class Permission {
    @Id
    private String name;
    private String resource;
    private String action;
    // getters and setters
}

@Entity
public class Role {
    @Id
    private String name;
    
    @ManyToMany(fetch = FetchType.EAGER)
    private Set<Permission> permissions;
    // getters and setters
}

@Component("permissionEvaluator")
public class DynamicPermissionEvaluator implements PermissionEvaluator {
    
    @Autowired
    private PermissionService permissionService;
    
    @Override
    public boolean hasPermission(Authentication authentication, Object targetDomainObject, Object permission) {
        if (authentication == null || targetDomainObject == null) {
            return false;
        }
        
        String username = authentication.getName();
        String resource = targetDomainObject.getClass().getSimpleName();
        String action = permission.toString();
        
        return permissionService.hasPermission(username, resource, action);
    }
    
    @Override
    public boolean hasPermission(Authentication authentication, Serializable targetId, 
                               String targetType, Object permission) {
        return hasPermission(authentication, targetType, permission);
    }
}

@Service
public class PermissionService {
    
    @Autowired
    private UserRepository userRepository;
    
    public boolean hasPermission(String username, String resource, String action) {
        User user = userRepository.findByUsername(username);
        if (user == null) {
            return false;
        }
        
        return user.getRoles().stream()
                .flatMap(role -> role.getPermissions().stream())
                .anyMatch(permission -> 
                    permission.getResource().equals(resource) && 
                    permission.getAction().equals(action));
    }
}
```

### 11.3 Multi-Tenant Security

```java
@Component
public class TenantAwareUserDetailsService implements UserDetailsService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        String tenantId = TenantContext.getCurrentTenant();
        
        User user = userRepository.findByUsernameAndTenantId(username, tenantId)
            .orElseThrow(() -> new UsernameNotFoundException("User not found"));
        
        return new TenantAwareUserPrincipal(user, tenantId);
    }
}

public class TenantAwareUserPrincipal implements UserDetails {
    private User user;
    private String tenantId;
    
    // Constructor and UserDetails implementation
    
    public String getTenantId() {
        return tenantId;
    }
}

@PreAuthorize("@tenantSecurityService.hasAccessToTenant(authentication, #tenantId)")
@GetMapping("/api/tenants/{tenantId}/data")
public ResponseEntity<List<Data>> getTenantData(@PathVariable String tenantId) {
    return ResponseEntity.ok(dataService.findByTenantId(tenantId));
}

@Component
public class TenantSecurityService {
    
    public boolean hasAccessToTenant(Authentication authentication, String tenantId) {
        if (authentication.getPrincipal() instanceof TenantAwareUserPrincipal) {
            TenantAwareUserPrincipal principal = (TenantAwareUserPrincipal) authentication.getPrincipal();
            return principal.getTenantId().equals(tenantId) || 
                   authentication.getAuthorities().stream()
                       .anyMatch(auth -> auth.getAuthority().equals("ROLE_SUPER_ADMIN"));
        }
        return false;
    }
}
```

---

## 12) Resources

### Official Documentation
- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
- [Spring Boot Security](https://docs.spring.io/spring-boot/docs/current/reference/html/web.html#web.security)
- [Spring Security OAuth2](https://docs.spring.io/spring-security/reference/servlet/oauth2/index.html)

### Security Standards
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Application Security Verification Standard](https://owasp.org/www-project-application-security-verification-standard/)
- [RFC 6749 - OAuth 2.0](https://tools.ietf.org/html/rfc6749)
- [RFC 7519 - JWT](https://tools.ietf.org/html/rfc7519)

### Tools and Libraries
- [Spring Security Test](https://docs.spring.io/spring-security/reference/servlet/test/index.html)
- [OWASP Dependency Check](https://owasp.org/www-project-dependency-check/)
- [Snyk](https://snyk.io/) - Vulnerability scanning
- [SonarQube](https://www.sonarqube.org/) - Code quality and security

### Books and Articles
- "Spring Security in Action" by Laurentiu Spilca
- "OAuth 2 in Action" by Justin Richer and Antonio Sanso
- [Baeldung Spring Security Tutorials](https://www.baeldung.com/security-spring)

---
