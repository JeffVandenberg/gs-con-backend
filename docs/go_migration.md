# Migrating from PHP to Go: Architecture and Implementation Guide

## Introduction

This document outlines the changes needed to migrate the Gaming Convention Web Application from PHP 8.x to Go. The migration would involve changes to the architecture, service layer components, database connectivity, and database migration strategies.

## Architecture Changes

### Current PHP Architecture

The current architecture is based on PHP 8.x with options like Laravel, Symfony, or Slim frameworks. These frameworks provide:
- MVC pattern implementation
- Routing
- Middleware support
- ORM integration (Doctrine, Eloquent)
- Authentication and authorization
- Dependency injection

### Proposed Go Architecture

Go has different architectural patterns and frameworks compared to PHP. Here's how the architecture would change:

#### Web Framework

**Options:**
1. **Gin** - A lightweight web framework with excellent performance and middleware support
2. **Echo** - High performance, minimalist framework with good middleware support
3. **Chi** - Lightweight, idiomatic router with middleware support
4. **Fiber** - Express-inspired web framework built on top of Fasthttp

**Recommendation:** Gin or Echo would be the most suitable for this application due to their performance, middleware support, and active communities.

#### Application Structure

The Go application would follow a clean architecture approach with the following layers:

1. **Handlers/Controllers Layer** - HTTP request handling, input validation, response formatting
2. **Service Layer** - Business logic implementation
3. **Repository Layer** - Data access and persistence
4. **Domain Layer** - Core business entities and interfaces

```
/
├── cmd/                  # Application entry points
│   └── api/              # API server
├── internal/             # Private application code
│   ├── domain/           # Domain models and interfaces
│   ├── handlers/         # HTTP handlers
│   ├── middleware/       # HTTP middleware
│   ├── services/         # Business logic
│   └── repositories/     # Data access
├── pkg/                  # Public libraries
│   ├── auth/             # Authentication utilities
│   ├── config/           # Configuration management
│   └── validator/        # Input validation
├── migrations/           # Database migrations
└── configs/              # Configuration files
```

#### Dependency Injection

Unlike PHP frameworks that often have built-in DI containers, Go typically uses manual dependency injection or lightweight DI libraries:

**Options:**
1. **Manual DI** - Explicitly pass dependencies through constructors
2. **Wire** - Compile-time dependency injection by Google
3. **Dig** - Runtime dependency injection by Uber

**Recommendation:** Use manual DI for simplicity or Wire for larger applications with many dependencies.

## Common Service Layer Components

The service layer in Go would include the following components:

### Core Services

1. **UserService**
   ```go
   type UserService interface {
       GetByID(ctx context.Context, id string) (*domain.User, error)
       Create(ctx context.Context, user *domain.User) error
       Update(ctx context.Context, user *domain.User) error
       Delete(ctx context.Context, id string) error
       Authenticate(ctx context.Context, email, password string) (*domain.User, error)
       SocialLogin(ctx context.Context, provider string, token string) (*domain.User, error)
   }
   ```

2. **ConventionService**
   ```go
   type ConventionService interface {
       GetByID(ctx context.Context, id string) (*domain.Convention, error)
       List(ctx context.Context, filter *domain.ConventionFilter) ([]*domain.Convention, error)
       Create(ctx context.Context, convention *domain.Convention) error
       Update(ctx context.Context, convention *domain.Convention) error
       Delete(ctx context.Context, id string) error
   }
   ```

3. **BadgeService**
   ```go
   type BadgeService interface {
       GetByID(ctx context.Context, id string) (*domain.Badge, error)
       GetUserBadges(ctx context.Context, userID string) ([]*domain.Badge, error)
       Purchase(ctx context.Context, badge *domain.Badge) error
       Transfer(ctx context.Context, badgeID, newUserID string) error
       CheckIn(ctx context.Context, badgeID string) error
   }
   ```

4. **EventService**
   ```go
   type EventService interface {
       GetByID(ctx context.Context, id string) (*domain.Event, error)
       List(ctx context.Context, filter *domain.EventFilter) ([]*domain.Event, error)
       Create(ctx context.Context, event *domain.Event) error
       Update(ctx context.Context, event *domain.Event) error
       Delete(ctx context.Context, id string) error
       Register(ctx context.Context, eventID, badgeID string) error
       CancelRegistration(ctx context.Context, eventID, badgeID string) error
   }
   ```

### Supporting Services

1. **AuthService**
   ```go
   type AuthService interface {
       GenerateToken(ctx context.Context, user *domain.User) (string, error)
       ValidateToken(ctx context.Context, token string) (*domain.Claims, error)
       RefreshToken(ctx context.Context, refreshToken string) (string, error)
   }
   ```

2. **ValidationService**
   ```go
   type ValidationService interface {
       ValidateStruct(s interface{}) error
       ValidateField(field interface{}, tag string) error
   }
   ```

3. **AuditService**
   ```go
   type AuditService interface {
       LogAction(ctx context.Context, entityType, entityID, action string, oldValue, newValue interface{}) error
       GetLogs(ctx context.Context, filter *domain.AuditLogFilter) ([]*domain.AuditLog, error)
   }
   ```

### Implementation Pattern

Each service would be implemented with:
1. An interface defining the contract
2. A concrete implementation
3. Dependencies injected via constructor

Example:
```go
type badgeService struct {
    badgeRepo    repository.BadgeRepository
    userRepo     repository.UserRepository
    auditService AuditService
    validator    ValidationService
}

func NewBadgeService(
    badgeRepo repository.BadgeRepository,
    userRepo repository.UserRepository,
    auditService AuditService,
    validator ValidationService,
) BadgeService {
    return &badgeService{
        badgeRepo:    badgeRepo,
        userRepo:     userRepo,
        auditService: auditService,
        validator:    validator,
    }
}

func (s *badgeService) GetByID(ctx context.Context, id string) (*domain.Badge, error) {
    // Implementation
}
```

## Database Connectivity

### Current PHP Approach

The current approach uses PHP ORMs like Doctrine or Eloquent, which provide:
- Object-relational mapping
- Query building
- Relationship management
- Migrations

### Go Database Connectivity Options

Go has different approaches to database connectivity:

#### Low-level Libraries

1. **database/sql** - Standard library package for database operations
   - Provides a generic interface around SQL databases
   - Requires a database driver (e.g., lib/pq for PostgreSQL)
   - Manual mapping between database rows and Go structs

2. **sqlx** - Extension of database/sql
   - Simplifies mapping between rows and structs
   - Provides convenience functions for common operations
   - Still requires manual relationship management

#### ORMs and Query Builders

1. **GORM** - Full-featured ORM for Go
   - Supports relationships, migrations, hooks
   - Auto-migrations (though not recommended for production)
   - Callbacks and hooks

2. **SQLBoiler** - Type-safe SQL query builder
   - Generates code based on your database schema
   - Type-safe queries with compile-time checking
   - Excellent performance

3. **Ent** - Entity framework by Facebook
   - Schema-as-code approach
   - Type-safe API
   - Code generation for queries

#### Recommendation

For this complex application with many relationships:

1. **Primary Option: Ent**
   - Schema-as-code approach fits well with Go's type system
   - Handles complex relationships well
   - Code generation provides type safety
   - Good migration support

2. **Alternative: GORM**
   - More familiar to developers coming from ORM backgrounds
   - Easier learning curve
   - Good documentation and community support

Example with Ent:
```go
// Define schema
type Badge struct {
    ent.Schema
}

func (Badge) Fields() []ent.Field {
    return []ent.Field{
        field.UUID("id", uuid.UUID{}),
        field.String("badge_number").Unique(),
        field.Time("purchase_date"),
        field.Float("price"),
        field.String("status"),
        field.Bool("checked_in"),
        field.Time("check_in_time").Optional().Nillable(),
        field.JSON("additional_fields", map[string]interface{}{}),
    }
}

func (Badge) Edges() []ent.Edge {
    return []ent.Edge{
        edge.From("user", User.Type).Ref("badges").Unique().Required(),
        edge.From("convention", Convention.Type).Ref("badges").Unique().Required(),
        edge.From("tier", BadgeTier.Type).Ref("badges").Unique().Required(),
        edge.From("previous_user", User.Type).Ref("transferred_badges").Unique().Optional(),
        edge.To("event_registrations", EventRegistration.Type),
        edge.To("event_hosts", EventHost.Type),
        edge.To("volunteer_shifts", VolunteerShift.Type),
        edge.To("game_checkouts", GameCheckout.Type),
    }
}

// Usage in repository
func (r *badgeRepository) GetByID(ctx context.Context, id string) (*domain.Badge, error) {
    badge, err := r.client.Badge.Query().
        Where(badge.ID(id)).
        WithUser().
        WithConvention().
        WithTier().
        First(ctx)
    
    if err != nil {
        if ent.IsNotFound(err) {
            return nil, domain.ErrBadgeNotFound
        }
        return nil, err
    }
    
    return mapEntToDomain(badge), nil
}
```

## Database Migration Strategy

### Current PHP Approach

The current approach likely uses migration tools from PHP frameworks:
- Laravel Migrations
- Doctrine Migrations
- Phinx

### Go Migration Options

1. **golang-migrate**
   - CLI and library for database migrations
   - Supports multiple database types
   - Version-controlled migrations
   - Up/down migration support

2. **Atlas**
   - Schema migration tool with declarative syntax
   - Supports multiple database types
   - Can generate migrations from schema changes

3. **goose**
   - Simple migration tool for Go
   - SQL or Go-based migrations
   - Timestamp-based versioning

4. **Ent Migrations**
   - If using Ent, it provides its own migration system
   - Can generate migrations from schema changes
   - Supports auto-migrations (dev only)

#### Recommendation

**Primary Option: golang-migrate with SQL migrations**
- Industry standard approach
- Clear, explicit SQL migrations
- Good for complex schema with many constraints
- Works well with CI/CD pipelines

Example migration files:

```sql
-- 20230601120000_create_users_table.up.sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255),
    first_name VARCHAR(255) NOT NULL,
    last_name VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- 20230601120000_create_users_table.down.sql
DROP TABLE IF EXISTS users;
```

Migration workflow:
1. Create migration files (up/down)
2. Run migrations as part of deployment
3. Version control migration files alongside code

## Authentication and Authorization

### Current PHP Approach

The current approach uses:
- JWT for tokens
- OAuth 2.0 for social login
- Role-based access control

### Go Implementation

1. **JWT Authentication**
   - Use `golang-jwt/jwt` package
   - Implement token generation, validation, and refresh

2. **OAuth 2.0**
   - Use `golang.org/x/oauth2` for social login
   - Implement provider-specific handlers

3. **Authorization**
   - Implement middleware for role-based access control
   - Use context to pass user information

Example JWT implementation:
```go
import "github.com/golang-jwt/jwt/v5"

type Claims struct {
    UserID string `json:"user_id"`
    Email  string `json:"email"`
    Role   string `json:"role"`
    jwt.RegisteredClaims
}

func (s *authService) GenerateToken(ctx context.Context, user *domain.User) (string, error) {
    expirationTime := time.Now().Add(24 * time.Hour)
    claims := &Claims{
        UserID: user.ID,
        Email:  user.Email,
        Role:   user.Role,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(expirationTime),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
            Issuer:    "gaming-convention-api",
        },
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString([]byte(s.secretKey))
}
```

## API Documentation

### Current PHP Approach

The current approach uses:
- OpenAPI (Swagger)
- PHP annotations or separate YAML/JSON files

### Go Implementation

1. **Swaggo**
   - Generate OpenAPI documentation from Go comments
   - Integrates with Gin, Echo, and other frameworks

2. **go-swagger**
   - More comprehensive OpenAPI tooling
   - Can generate server stubs and client code

Example with Swaggo:
```go
// @Summary Get badge by ID
// @Description Get a badge by its ID
// @Tags badges
// @Accept json
// @Produce json
// @Param id path string true "Badge ID"
// @Success 200 {object} domain.Badge
// @Failure 404 {object} ErrorResponse
// @Failure 500 {object} ErrorResponse
// @Router /badges/{id} [get]
func (h *badgeHandler) GetByID(c *gin.Context) {
    // Implementation
}
```

## Performance Considerations

Go offers several performance advantages over PHP:

1. **Compiled Language**
   - Go compiles to native machine code
   - No interpretation overhead
   - Faster startup time

2. **Concurrency**
   - Goroutines are lightweight and efficient
   - Can handle many concurrent requests
   - Built-in synchronization primitives

3. **Memory Management**
   - Efficient garbage collection
   - Lower memory footprint

Example of concurrent processing:
```go
func (s *eventService) GetEventStats(ctx context.Context, conventionID string) (*domain.EventStats, error) {
    var wg sync.WaitGroup
    var mu sync.Mutex
    stats := &domain.EventStats{}
    errCh := make(chan error, 3)
    
    wg.Add(3)
    
    go func() {
        defer wg.Done()
        count, err := s.eventRepo.CountByConvention(ctx, conventionID)
        if err != nil {
            errCh <- err
            return
        }
        mu.Lock()
        stats.TotalEvents = count
        mu.Unlock()
    }()
    
    go func() {
        defer wg.Done()
        count, err := s.eventRepo.CountRegistrations(ctx, conventionID)
        if err != nil {
            errCh <- err
            return
        }
        mu.Lock()
        stats.TotalRegistrations = count
        mu.Unlock()
    }()
    
    go func() {
        defer wg.Done()
        popular, err := s.eventRepo.GetMostPopular(ctx, conventionID, 5)
        if err != nil {
            errCh <- err
            return
        }
        mu.Lock()
        stats.PopularEvents = popular
        mu.Unlock()
    }()
    
    wg.Wait()
    close(errCh)
    
    if len(errCh) > 0 {
        return nil, <-errCh
    }
    
    return stats, nil
}
```

## Migration Strategy

A phased approach to migration is recommended:

1. **Phase 1: Core Infrastructure**
   - Set up Go project structure
   - Implement database connectivity
   - Create base domain models
   - Implement authentication

2. **Phase 2: API Endpoints**
   - Implement handlers for core entities
   - Set up middleware (logging, auth, etc.)
   - Create service layer for business logic

3. **Phase 3: Advanced Features**
   - Implement complex business logic
   - Add reporting and analytics
   - Integrate with external services

4. **Phase 4: Testing and Optimization**
   - Comprehensive testing
   - Performance optimization
   - Documentation

## Conclusion

Migrating from PHP to Go for the Gaming Convention Web Application would involve significant changes to the architecture, service layer, and database connectivity approach. However, the benefits of Go's performance, type safety, and concurrency model make it a compelling choice for a complex application like this.

The recommended approach uses:
- Gin or Echo as the web framework
- Clean architecture with clear separation of concerns
- Ent for database connectivity and schema management
- golang-migrate for database migrations
- JWT for authentication

This migration would result in a more performant, type-safe, and maintainable application that can better handle the complex requirements of a gaming convention management system.