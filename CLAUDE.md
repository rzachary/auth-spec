# CLAUDE.md - Project Guidelines

## Project Overview

JWT Authentication Service built with Java 17+, Spring Boot 3.2.x, and Gradle (Kotlin DSL). File-based user authentication designed for local development and testing.

---

## Core Coding Principles

### General Philosophy
- **Simplicity over cleverness**: Write straightforward, readable code
- **Fail fast**: Validate inputs early, throw meaningful exceptions
- **Stateless design**: All state lives in the JWT; no server-side sessions
- **Security by default**: Never log sensitive data (passwords, tokens, secrets)

### Java Style Guide

**Naming Conventions:**
```java
// Classes: PascalCase
public class JwtService {}

// Methods/Variables: camelCase
public String generateToken(String username) {}

// Constants: SCREAMING_SNAKE_CASE
private static final long TOKEN_EXPIRATION_MS = 3600000L;

// Packages: lowercase
package com.example.auth.service;
```

**Code Organization:**
```java
public class ExampleService {
    // 1. Static fields
    private static final Logger log = LoggerFactory.getLogger(ExampleService.class);

    // 2. Instance fields (final first)
    private final JwtConfig jwtConfig;
    private String cachedValue;

    // 3. Constructor
    public ExampleService(JwtConfig jwtConfig) {
        this.jwtConfig = jwtConfig;
    }

    // 4. Public methods
    // 5. Package-private methods
    // 6. Private methods
}
```

**Formatting:**
- 4 spaces for indentation (no tabs)
- 120 character line limit
- Braces on same line (K&R style)
- One blank line between methods
- No trailing whitespace

**Annotations Order:**
```java
@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
@Validated
public class AuthController {}
```

### Spring Boot Conventions

**Dependency Injection:**
```java
// PREFERRED: Constructor injection
@Service
public class AuthService {
    private final JwtService jwtService;
    private final UserService userService;

    public AuthService(JwtService jwtService, UserService userService) {
        this.jwtService = jwtService;
        this.userService = userService;
    }
}

// AVOID: Field injection
@Autowired
private JwtService jwtService;  // Don't do this
```

**Configuration Properties:**
```java
@ConfigurationProperties(prefix = "jwt")
@Validated
public record JwtConfig(
    @NotBlank String secret,
    @Positive long expiration,
    @NotBlank String issuer
) {}
```

**REST Controllers:**
```java
@PostMapping("/login")
public ResponseEntity<LoginResponse> login(@Valid @RequestBody LoginRequest request) {
    // Return ResponseEntity for explicit status control
    return ResponseEntity.ok(authService.authenticate(request));
}
```

---

## Testing Instructions

### Test Framework Stack
- **JUnit 5**: Test framework
- **Mockito**: Mocking dependencies
- **AssertJ**: Fluent assertions
- **Spring Boot Test**: Integration testing
- **Spring Security Test**: Security context testing

### Running Tests

```bash
# Run all tests
./gradlew test

# Run specific test class
./gradlew test --tests "JwtServiceTest"

# Run with coverage report
./gradlew test jacocoTestReport

# Run tests continuously during development
./gradlew test --continuous
```

### Test Organization

```
src/test/java/com/example/auth/
├── controller/
│   └── AuthControllerTest.java      # @WebMvcTest
├── service/
│   ├── JwtServiceTest.java          # Unit test
│   └── AuthServiceTest.java         # Unit test with mocks
├── integration/
│   └── AuthIntegrationTest.java     # @SpringBootTest
└── TestFixtures.java                # Shared test data
```

### Test Naming Convention

```java
@Test
void methodName_scenario_expectedBehavior() {
    // Example:
    // generateToken_validUsername_returnsValidJwt()
    // authenticate_invalidPassword_throwsAuthException()
    // validateToken_expiredToken_returnsFalse()
}
```

### Test Structure (Arrange-Act-Assert)

```java
@Test
void authenticate_validCredentials_returnsToken() {
    // Arrange
    LoginRequest request = new LoginRequest("testuser", "password");
    when(userService.findByUsername("testuser")).thenReturn(Optional.of(testUser));

    // Act
    LoginResponse response = authService.authenticate(request);

    // Assert
    assertThat(response.token()).isNotBlank();
    assertThat(response.tokenType()).isEqualTo("Bearer");
}
```

### Integration Test Example

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class AuthIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void loginEndpoint_validCredentials_returns200WithToken() {
        LoginRequest request = new LoginRequest("testuser", "password");

        ResponseEntity<LoginResponse> response = restTemplate.postForEntity(
            "/api/auth/login", request, LoginResponse.class);

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody().token()).isNotBlank();
    }
}
```

### Test Coverage Expectations
- Service layer: 90%+ coverage
- Controller layer: 80%+ coverage
- Configuration classes: Not required
- DTOs/Records: Not required

---

## Repository Etiquette

### Branch Naming

```
main                    # Production-ready code
├── feature/JWT-123-add-token-refresh
├── bugfix/JWT-456-fix-expiration-check
├── hotfix/JWT-789-security-patch
└── chore/JWT-101-update-dependencies
```

**Format:** `<type>/<ticket>-<short-description>`

**Types:**
- `feature/` - New functionality
- `bugfix/` - Bug fixes
- `hotfix/` - Urgent production fixes
- `chore/` - Maintenance tasks
- `refactor/` - Code restructuring
- `docs/` - Documentation only

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

**Examples:**
```
feat(auth): add token refresh endpoint

fix(jwt): correct expiration timestamp calculation

test(service): add unit tests for JwtService

docs(readme): update API examples

chore(deps): bump spring-boot to 3.2.1
```

**Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

### Pull Request Guidelines

1. **Title**: Match commit message format
2. **Description**: Include:
   - What changed and why
   - Testing performed
   - Breaking changes (if any)
3. **Size**: Keep PRs focused (<400 lines preferred)
4. **Tests**: All tests must pass
5. **Review**: Minimum 1 approval required

### Merging Strategy

- **Feature branches → main**: Squash and merge
- **Hotfixes → main**: Merge commit (preserve history)
- **Delete branch** after merge

---

## Developer Environment Setup

### Prerequisites

| Tool | Version | Installation |
|------|---------|--------------|
| Java | 17+ | `brew install openjdk@17` or SDKMAN |
| Gradle | 8.x | Use wrapper (`./gradlew`) |
| Git | 2.x+ | `brew install git` |
| IDE | IntelliJ IDEA recommended | JetBrains Toolbox |

### Initial Setup

```bash
# Clone repository
git clone <repository-url>
cd auth-test

# Verify Java version
java -version  # Should be 17+

# Build project
./gradlew build

# Run application
./gradlew bootRun

# Verify running
curl http://localhost:8080/api/auth/health
```

### IDE Configuration (IntelliJ IDEA)

1. **Import Project**: Open as Gradle project
2. **SDK**: Set Project SDK to Java 17
3. **Annotation Processing**: Enable for Lombok
   - Settings → Build → Compiler → Annotation Processors → Enable
4. **Code Style**: Import project code style
   - Settings → Editor → Code Style → Import from `.editorconfig`
5. **Plugins**: Install Lombok plugin

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `JWT_SECRET` | Signing key (min 256 bits) | Dev default in config |
| `SERVER_PORT` | Application port | 8080 |
| `SPRING_PROFILES_ACTIVE` | Active profile | local |

### Local Development

```bash
# Run with local profile (debug logging, extended token expiration)
./gradlew bootRun --args='--spring.profiles.active=local'

# Run with custom port
SERVER_PORT=9090 ./gradlew bootRun

# Hot reload (requires Spring DevTools)
./gradlew bootRun  # Changes auto-reload
```

### Useful Gradle Tasks

```bash
./gradlew tasks          # List all tasks
./gradlew dependencies   # Show dependency tree
./gradlew bootJar        # Build executable JAR
./gradlew clean build    # Clean rebuild
./gradlew test --info    # Verbose test output
```

---

## Anti-Patterns to Avoid

### Security Anti-Patterns

```java
// ❌ NEVER: Log sensitive data
log.debug("Password: {}", password);
log.info("Token: {}", jwtToken);

// ✅ DO: Log safe identifiers only
log.debug("Authentication attempt for user: {}", username);
log.info("Token generated for user: {}", username);
```

```java
// ❌ NEVER: Hardcode secrets in code
private static final String SECRET = "my-secret-key";

// ✅ DO: Use configuration/environment
@Value("${jwt.secret}")
private String secret;
```

```java
// ❌ NEVER: Return detailed error messages to client
throw new RuntimeException("User admin not found in database table users");

// ✅ DO: Generic messages externally, detailed logs internally
log.warn("User not found: {}", username);
throw new AuthException("INVALID_CREDENTIALS", "Invalid username or password");
```

### Code Quality Anti-Patterns

```java
// ❌ NEVER: Catch generic Exception without reason
try {
    return jwtService.generateToken(user);
} catch (Exception e) {
    return null;
}

// ✅ DO: Catch specific exceptions, handle appropriately
try {
    return jwtService.generateToken(user);
} catch (JwtException e) {
    log.error("Token generation failed for user: {}", user.getUsername(), e);
    throw new AuthException("TOKEN_GENERATION_FAILED", "Unable to generate token");
}
```

```java
// ❌ NEVER: Use field injection
@Autowired
private JwtService jwtService;

// ✅ DO: Use constructor injection
private final JwtService jwtService;

public AuthService(JwtService jwtService) {
    this.jwtService = jwtService;
}
```

```java
// ❌ NEVER: Return null from service methods
public User findUser(String username) {
    return userRepository.find(username);  // Could be null
}

// ✅ DO: Use Optional for potentially absent values
public Optional<User> findUser(String username) {
    return userRepository.find(username);
}
```

### Testing Anti-Patterns

```java
// ❌ NEVER: Test multiple behaviors in one test
@Test
void testAuthentication() {
    // Tests login, token generation, validation, and refresh...
}

// ✅ DO: One behavior per test
@Test
void authenticate_validCredentials_returnsToken() {}

@Test
void authenticate_invalidPassword_throwsException() {}
```

```java
// ❌ NEVER: Use production data/secrets in tests
@Value("${jwt.secret}")
private String realSecret;

// ✅ DO: Use test-specific configuration
@TestConfiguration
static class TestConfig {
    @Bean
    public JwtConfig jwtConfig() {
        return new JwtConfig("test-secret-key-for-testing-only-256-bits", 3600000, "test");
    }
}
```

### Architecture Anti-Patterns

```java
// ❌ NEVER: Business logic in controllers
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest request) {
    User user = userRepository.findByUsername(request.username());
    if (passwordEncoder.matches(request.password(), user.getPassword())) {
        String token = Jwts.builder()...  // Business logic here
    }
}

// ✅ DO: Controllers delegate to services
@PostMapping("/login")
public ResponseEntity<LoginResponse> login(@RequestBody LoginRequest request) {
    return ResponseEntity.ok(authService.authenticate(request));
}
```

```java
// ❌ NEVER: Circular dependencies
@Service
public class AuthService {
    @Autowired private UserService userService;
}

@Service
public class UserService {
    @Autowired private AuthService authService;  // Circular!
}

// ✅ DO: Unidirectional dependencies or extract shared logic
```

### JWT-Specific Anti-Patterns

```java
// ❌ NEVER: Store sensitive data in JWT payload
claims.put("password", user.getPassword());
claims.put("ssn", user.getSsn());

// ✅ DO: Minimal claims only
claims.put("sub", user.getUsername());
claims.put("roles", user.getRoles());
```

```java
// ❌ NEVER: Use weak or short secrets
String secret = "secret";  // Too short, easily guessed

// ✅ DO: Use cryptographically secure secrets (256+ bits for HS256)
// Generated via: openssl rand -base64 32
```

```java
// ❌ NEVER: Skip token expiration
Jwts.builder()
    .setSubject(username)
    // No expiration set!
    .signWith(key)
    .compact();

// ✅ DO: Always set expiration
Jwts.builder()
    .setSubject(username)
    .setExpiration(new Date(System.currentTimeMillis() + expirationMs))
    .signWith(key)
    .compact();
```

---

## Quick Reference

### Common Commands

```bash
./gradlew bootRun                    # Start server
./gradlew test                       # Run tests
./gradlew build                      # Build project
curl localhost:8080/api/auth/health  # Health check
```

### Key Files

| File | Purpose |
|------|---------|
| `SPEC.md` | API specification |
| `build.gradle.kts` | Dependencies and build config |
| `application.yml` | Application configuration |
| `users.json` | File-based user storage |

### Default Test Credentials

| Username | Password | Roles |
|----------|----------|-------|
| admin | admin123 | ADMIN, USER |
| testuser | password | USER |
