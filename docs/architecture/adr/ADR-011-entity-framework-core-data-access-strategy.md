# ADR-011: Entity Framework Core Data Access Strategy

**Status:** Accepted
**Date:** 2026-10-29
**Owner:** Architecture & Systems Design

---

# 1. Context

The Fitness platform requires a data access architecture that supports:

- Audit logging
- Fitness domain entities
- Application services
- ASP.NET Core and Blazor integration
- Testability
- Modern .NET development practices
- Future application growth

The project is transitioning from a primarily database-centric design approach toward a modern .NET application architecture.

Historically, the preferred pattern has been:

```text
Application
    ↓
Stored Procedure
    ↓
SQL Server
```

This architecture emphasizes:

- SQL Server as the authoritative source of truth
- Stored procedures for database interaction
- Explicit SQL control
- Strong database governance

As development progressed, the need arose to determine whether the platform should continue using direct SQL access or adopt Entity Framework Core.

---

# 2. Decision

The Fitness platform shall adopt Entity Framework Core as the primary data access technology.

Primary package:

```text
Microsoft.EntityFrameworkCore.SqlServer
```

Data access shall be centered around a single application DbContext:

```csharp
FitnessDbContext
```

The DbContext will represent the application's primary transactional boundary.

---

# 3. Rationale

## 3.1 Modern .NET Alignment

Entity Framework Core is widely used throughout modern .NET applications.

Adopting EF Core supports learning objectives in:

- ASP.NET Core
- Blazor
- Dependency Injection
- LINQ
- Entity Configuration
- Database Migrations
- Repository and Service Patterns

These skills are commonly expected in contemporary .NET development roles.

---

## 3.2 Portfolio Development

One goal of the Fitness platform is to serve as a professional portfolio demonstrating:

- Architecture
- TDD
- Application Development
- Infrastructure Design
- Modern Microsoft Stack Technologies

Entity Framework Core increases the portfolio's relevance to current enterprise development practices.

---

## 3.3 Future Domain Growth

The current data model contains only a limited number of entities.

Future development is expected to include:

- Exercises
- Workouts
- Programs
- Equipment
- User Profiles
- Progress Tracking
- Audit Logging

Entity Framework Core provides strong support for:

- Relationships
- Navigation Properties
- Aggregate Modeling
- Transaction Management

which naturally align with future application requirements.

---

## 3.4 Existing SQL Server Expertise

Adoption of EF Core does not replace SQL Server expertise.

The system shall continue to leverage:

- Database schemas
- Constraints
- Indexing
- Keys
- SQL performance tuning
- T-SQL when appropriate

SQL Server remains the authoritative source of truth.

Entity Framework Core provides an abstraction layer for application development rather than a replacement for sound database design.

---

# 4. Alternatives Considered

## 4.1 Microsoft.Data.SqlClient

### Pros

- Full SQL control
- Maximum transparency
- Minimal abstraction
- Excellent performance
- Familiar to database-centric development

### Cons

- More boilerplate code
- Manual mapping
- Increased maintenance
- Less alignment with modern .NET development practices

### Decision

Rejected as the primary data access strategy but remains available for specialized scenarios.

---

## 4.2 Dapper

### Pros

- Lightweight
- SQL-centric
- Excellent performance
- Reduced mapping effort

### Cons

- Additional dependency
- Limited learning value compared to EF Core
- Less common in enterprise application architectures

### Decision

Rejected as the primary data access strategy.

---

## 4.3 Stored Procedure-Centric Architecture

### Pros

- Strong database governance
- Explicit execution plans
- Familiar architecture

### Cons

- Higher development overhead
- Reduced flexibility
- Less alignment with current .NET practices
- Increased maintenance burden

### Decision

Rejected as the primary application architecture.

Stored procedures may still be used for:

- Reporting
- ETL
- Performance-critical operations
- Administrative processes

---

# 5. DbContext Strategy

The platform shall initially implement a single DbContext:

```csharp
FitnessDbContext
```

This context may include entities from multiple schemas.

Example:

```text
audit.ErrorLog
dbo.Exercise
dbo.Workout
dbo.Program
```

Multiple schemas alone do not justify multiple DbContexts.

---

# 6. Future Architectural Considerations

Additional DbContexts may be introduced if one or more of the following conditions occur:

- Separate databases
- Independent deployment cycles
- Distinct security boundaries
- Bounded contexts
- Independent scaling requirements

Examples:

```text
FitnessDbContext
IdentityDbContext
ReportingDbContext
LoggingDbContext
```

Such separation shall be driven by demonstrated architectural need rather than anticipated future requirements.

---

# 7. Consequences

## Positive

- Alignment with modern .NET development
- Improved portfolio relevance
- Simplified application development
- Strong support for domain modeling
- Integrated migration support
- Consistent application architecture

## Negative

- Higher abstraction than direct SQL access
- Reduced visibility into generated SQL
- Additional learning curve
- Potential for inefficient queries if not monitored

---

# 8. Architectural Principles Preserved

Adoption of Entity Framework Core does not change the following principles:

```text
SQL Server remains the source of truth.
Database constraints remain authoritative.
Database integrity is enforced at the database layer.
Application convenience does not override data governance.
```

EF Core is considered an application development tool rather than a replacement for sound database architecture.

---

# 9. Related Documents

- ADR-001 Database Choice
- ADR-010 Logging Severity and Verbosity Strategy
- SystemArchitecture.md
- DataArchitecture.md
- ApplicationArchitecture.md

---

# 10. Version History

| Version | Date | Author | Description |
|----------|----------|----------|----------|
| 1.0 | 2026-10-29 | Gordon | Initial decision adopting Entity Framework Core as the primary data access strategy. |
