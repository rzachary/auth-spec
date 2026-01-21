# JWT Authentication Service

A lightweight JWT-based authentication service built with Java 17+, Spring Boot 3.2.x, and Gradle. Designed for local development and testing with file-based user storage.

## Repository Contents

| File | Description |
|------|-------------|
| `SPEC.md` | Complete API specification, JWT structure, configuration, and implementation details |
| `CLAUDE.md` | Development guidelines, coding standards, and project conventions |
| `README.md` | This file - project overview and quick start guide |

## Technology Stack

- **Language**: Java 17+
- **Framework**: Spring Boot 3.2.x
- **Build Tool**: Gradle 8.x (Kotlin DSL)
- **JWT Library**: jjwt (io.jsonwebtoken) 0.12.x
- **Security**: Spring Security 6.x

## Quick Start

### Prerequisites

- Java 17+
- Gradle 8.x (or use the included wrapper)

### Build & Run

```bash
# Build the project
./gradlew build

# Run the application
./gradlew bootRun

# Run with local profile (debug logging, extended token expiration)
./gradlew bootRun --args='--spring.profiles.active=local'

# Run tests
./gradlew test
```

### Verify Installation

```bash
curl http://localhost:8080/api/auth/health
```

## API Endpoints

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/auth/login` | Authenticate and receive JWT | No |
| POST | `/api/auth/validate` | Validate a JWT token | Yes |
| POST | `/api/auth/refresh` | Refresh an existing token | Yes |
| GET | `/api/auth/health` | Service health check | No |

## Default Test Users

| Username | Password | Roles |
|----------|----------|-------|
| admin | admin123 | ADMIN, USER |
| testuser | password | USER |

## Example Usage

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
```

## Configuration

The application uses the following key configuration:

| Variable | Description | Default |
|----------|-------------|---------|
| `JWT_SECRET` | Signing key (min 256 bits) | Dev default in config |
| `SERVER_PORT` | Application port | 8080 |
| `SPRING_PROFILES_ACTIVE` | Active profile | local |

## Documentation

- **[SPEC.md](SPEC.md)** - Full API specification including request/response formats, JWT token structure, error codes, and configuration details
- **[CLAUDE.md](CLAUDE.md)** - Development guidelines, coding standards, testing instructions, and contribution workflow
