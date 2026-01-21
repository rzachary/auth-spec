# JWT Authentication Service Specification

## Overview

A lightweight JWT-based authentication service built with Java, Spring Boot, and Gradle. Designed for local development and testing with file-based user storage.

---

## Technology Stack

| Component | Technology | Version |
|-----------|------------|---------|
| Language | Java | 17+ |
| Framework | Spring Boot | 3.2.x |
| Build Tool | Gradle | 8.x (Kotlin DSL) |
| JWT Library | jjwt (io.jsonwebtoken) | 0.12.x |
| Security | Spring Security | 6.x |

---

## Project Structure

```
src/
├── main/
│   ├── java/com/example/auth/
│   │   ├── AuthApplication.java
│   │   ├── config/
│   │   │   ├── SecurityConfig.java
│   │   │   └── JwtConfig.java
│   │   ├── controller/
│   │   │   └── AuthController.java
│   │   ├── dto/
│   │   │   ├── LoginRequest.java
│   │   │   ├── LoginResponse.java
│   │   │   └── TokenValidationResponse.java
│   │   ├── service/
│   │   │   ├── AuthService.java
│   │   │   ├── JwtService.java
│   │   │   └── UserService.java
│   │   ├── model/
│   │   │   └── User.java
│   │   ├── filter/
│   │   │   └── JwtAuthenticationFilter.java
│   │   └── exception/
│   │       ├── AuthException.java
│   │       └── GlobalExceptionHandler.java
│   └── resources/
│       ├── application.yml
│       ├── application-local.yml
│       └── users.json
└── test/
    └── java/com/example/auth/
        ├── controller/
        │   └── AuthControllerTest.java
        └── service/
            ├── JwtServiceTest.java
            └── AuthServiceTest.java
```

---

## API Specification

### Base URL
```
http://localhost:8080/api/auth
```

### Endpoints

#### 1. Login (Authenticate)

**POST** `/api/auth/login`

Authenticates a user and returns a JWT token.

**Request Body:**
```json
{
  "username": "string",
  "password": "string"
}
```

**Success Response (200 OK):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

**Error Response (401 Unauthorized):**
```json
{
  "error": "INVALID_CREDENTIALS",
  "message": "Invalid username or password",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

#### 2. Validate Token

**POST** `/api/auth/validate`

Validates a JWT token and returns its claims.

**Request Header:**
```
Authorization: Bearer <token>
```

**Success Response (200 OK):**
```json
{
  "valid": true,
  "username": "testuser",
  "issuedAt": "2024-01-15T10:00:00Z",
  "expiresAt": "2024-01-15T11:00:00Z"
}
```

**Error Response (401 Unauthorized):**
```json
{
  "valid": false,
  "error": "TOKEN_EXPIRED",
  "message": "Token has expired"
}
```

---

#### 3. Refresh Token

**POST** `/api/auth/refresh`

Issues a new token using a valid existing token.

**Request Header:**
```
Authorization: Bearer <token>
```

**Success Response (200 OK):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

---

#### 4. Health Check

**GET** `/api/auth/health`

Returns service health status. No authentication required.

**Success Response (200 OK):**
```json
{
  "status": "UP",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

## JWT Token Specification

### Header
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload (Claims)
```json
{
  "sub": "username",
  "iat": 1705312200,
  "exp": 1705315800,
  "iss": "auth-service",
  "roles": ["USER"]
}
```

### Standard Claims Used

| Claim | Description | Required |
|-------|-------------|----------|
| `sub` | Subject (username) | Yes |
| `iat` | Issued At (Unix timestamp) | Yes |
| `exp` | Expiration Time (Unix timestamp) | Yes |
| `iss` | Issuer ("auth-service") | Yes |
| `roles` | User roles array | Yes |

---

## File-Based Authentication

### User Storage Format (`users.json`)

```json
{
  "users": [
    {
      "username": "admin",
      "password": "$2a$10$...",
      "roles": ["ADMIN", "USER"],
      "enabled": true
    },
    {
      "username": "testuser",
      "password": "$2a$10$...",
      "roles": ["USER"],
      "enabled": true
    }
  ]
}
```

### Password Storage
- Passwords stored using BCrypt hashing
- Cost factor: 10 (default for local testing)
- Plain text passwords never stored

### Default Test Users

| Username | Password | Roles |
|----------|----------|-------|
| admin | admin123 | ADMIN, USER |
| testuser | password | USER |

---

## Configuration

### application.yml

```yaml
server:
  port: 8080

spring:
  application:
    name: auth-service
  profiles:
    active: local

jwt:
  secret: ${JWT_SECRET:local-dev-secret-key-min-256-bits-long-for-hs256}
  expiration: 3600000  # 1 hour in milliseconds
  issuer: auth-service

auth:
  users-file: classpath:users.json
```

### application-local.yml

```yaml
logging:
  level:
    com.example.auth: DEBUG
    org.springframework.security: DEBUG

jwt:
  # Longer expiration for local testing (24 hours)
  expiration: 86400000
```

---

## Security Configuration

### Public Endpoints (No Authentication)
- `POST /api/auth/login`
- `GET /api/auth/health`

### Protected Endpoints (Require Valid JWT)
- `POST /api/auth/validate`
- `POST /api/auth/refresh`

### CORS Configuration (Local Development)
```yaml
cors:
  allowed-origins: "*"
  allowed-methods: GET, POST, PUT, DELETE, OPTIONS
  allowed-headers: "*"
  allow-credentials: false
```

---

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INVALID_CREDENTIALS` | 401 | Username or password incorrect |
| `TOKEN_EXPIRED` | 401 | JWT token has expired |
| `TOKEN_INVALID` | 401 | JWT token malformed or signature invalid |
| `TOKEN_MISSING` | 401 | Authorization header missing or malformed |
| `USER_DISABLED` | 403 | User account is disabled |
| `INTERNAL_ERROR` | 500 | Unexpected server error |

---

## Build & Run

### Prerequisites
- Java 17+
- Gradle 8.x (or use wrapper)

### Commands

```bash
# Build
./gradlew build

# Run
./gradlew bootRun

# Run with local profile
./gradlew bootRun --args='--spring.profiles.active=local'

# Run tests
./gradlew test

# Build executable JAR
./gradlew bootJar
```

---

## Testing

### Unit Tests
- `JwtServiceTest`: Token generation, parsing, validation
- `AuthServiceTest`: Authentication logic, user lookup
- `AuthControllerTest`: Endpoint integration tests

### Manual Testing with cURL

```bash
# Login
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"testuser","password":"password"}'

# Validate token
curl -X POST http://localhost:8080/api/auth/validate \
  -H "Authorization: Bearer <token>"

# Refresh token
curl -X POST http://localhost:8080/api/auth/refresh \
  -H "Authorization: Bearer <token>"

# Health check
curl http://localhost:8080/api/auth/health
```

---

## Dependencies (build.gradle.kts)

```kotlin
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("org.springframework.boot:spring-boot-starter-validation")

    // JWT
    implementation("io.jsonwebtoken:jjwt-api:0.12.5")
    runtimeOnly("io.jsonwebtoken:jjwt-impl:0.12.5")
    runtimeOnly("io.jsonwebtoken:jjwt-jackson:0.12.5")

    // JSON processing
    implementation("com.fasterxml.jackson.core:jackson-databind")

    // Lombok (optional)
    compileOnly("org.projectlombok:lombok")
    annotationProcessor("org.projectlombok:lombok")

    // Testing
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testImplementation("org.springframework.security:spring-security-test")
}
```

---

## Implementation Notes

1. **Secret Key Management**: For local testing, a default secret is embedded. For production, use environment variable `JWT_SECRET` with a secure 256-bit key.

2. **Token Refresh Strategy**: Tokens can be refreshed if they are still valid. Expired tokens cannot be refreshed (user must re-authenticate).

3. **File Watching**: The user file is loaded at startup. For hot-reload during development, consider adding a refresh endpoint or file watcher.

4. **Stateless Design**: No server-side session storage. All state is contained in the JWT.

5. **BCrypt Passwords**: Use Spring's `PasswordEncoder` for consistent hashing.

---

## Future Considerations (Out of Scope)

- Database-backed user storage
- Refresh token rotation
- Token blacklisting/revocation
- OAuth2/OIDC integration
- Rate limiting
- Multi-factor authentication
