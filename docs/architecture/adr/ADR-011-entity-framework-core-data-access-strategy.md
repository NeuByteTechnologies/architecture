# ADR-011: Entity Framework Core Data Access Strategy

**Status:** Accepted
**Date:** 2026-10-29
**Owner:** Architecture & Systems Design

---

# 1. Context

The Fitness platform requires a data access architecture capable of supporting:

- Fitness domain entities
- Application services
- Audit logging
- Reference data management
- ASP.NET Core and Blazor integration
- Testability
- Reporting and analytics
- ETL processes
- Future application growth

Historically, development has followed a database-centric architecture:

```text
Application
    ↓
Stored Procedure
    ↓
SQL Server
```

This approach emphasizes:

- SQL Server as the authoritative source of truth
- Stored procedures for business operations
- Explicit query and transaction control
- Strong database governance
- Reference data management within the database
- Reporting and ETL support

As the Fitness platform evolved, a formal decision was required regarding adoption of Entity Framework Core while preserving the database governance principles established through prior development experience.

---

# 2. Decision

The Fitness platform shall adopt Entity Framework Core as the primary data access framework.

Primary package:

```text
Microsoft.EntityFrameworkCore.SqlServer
```

The primary application data boundary shall be represented through:

```text
FitnessDbContext
```

Entity Framework Core shall be used as the standard mechanism for:

- Entity mapping
- Dependency injection integration
- Transaction coordination
- Change tracking
- Migrations
- Unit testing support
- Repository implementations

The adoption of Entity Framework Core does not require exclusive use of entity-to-table CRUD operations.

A hybrid data access strategy shall be used where Entity Framework Core may interact with:

- Tables
- Views
- Stored Procedures

based upon the architectural requirements of the operation being performed.

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
- Repository Patterns
- Service Layer Design

These skills are commonly expected within current .NET development environments.

---

## 3.2 Portfolio Development

The Fitness platform serves as a professional portfolio demonstrating:

- Architecture
- TDD
- Application Development
- Infrastructure Design
- Modern Microsoft Stack Technologies

Entity Framework Core increases portfolio relevance while maintaining flexibility to utilize advanced SQL Server capabilities when appropriate.

---

## 3.3 Future Domain Growth

Future development is expected to include:

- Exercises
- Workouts
- Programs
- Equipment
- User Profiles
- Progress Tracking
- Notifications
- Audit Logging
- Diagnostics

Entity Framework Core provides strong support for:

- Relationships
- Navigation Properties
- Aggregate Modeling
- Transaction Management

which align naturally with anticipated growth of the domain model.

---

## 3.4 Existing SQL Server Expertise

Adoption of EF Core does not replace SQL Server expertise.

The platform shall continue to leverage:

- Schemas
- Constraints
- Primary Keys
- Foreign Keys
- Indexes
- Views
- Stored Procedures
- Query optimization
- Execution plan analysis
- T-SQL

SQL Server remains the authoritative source of truth.

Entity Framework Core is considered an application development framework rather than a replacement for sound database architecture.

---

## 3.5 Hybrid Data Access Strategy

The platform shall utilize a hybrid approach combining Entity Framework Core and database-native capabilities.

Entity Framework Core shall be preferred for:

- Simple CRUD operations
- Standard entity persistence
- Relationship management
- Common application queries
- Transaction coordination

Stored Procedures shall be considered appropriate for:

- Multi-table business operations
- Audit logging
- Complex validation workflows
- Reference data resolution
- Reporting operations
- ETL support
- Administrative functions
- Performance-sensitive operations

Views shall be considered appropriate for:

- Reporting
- Analytics
- Read-only projections
- ETL staging access
- Complex reusable query definitions

Selection of implementation technology shall be based upon:

- Governance requirements
- Maintainability
- Performance characteristics
- Architectural clarity
- Long-term supportability

---

# 4. Reference Data Governance

The platform includes managed reference data tables such as:

- Error Codes
- Error Severity
- Event Types
- Notification Types
- Workout Types
- User Types

Reference data is considered governed data rather than application constants.

Reference data tables may include governance fields such as:

```sql
create_date
last_update_date
deleted_flag
```

These tables may be used to support:

- Reporting dimensions
- ETL processes
- Operational validation
- Master data management

When application enums represent reference data:

- Integration tests shall validate alignment between code and database values.
- Database governance remains authoritative.
- Drift between application definitions and database reference data shall be detected through automated testing.

---

# 5. Alternatives Considered

## 5.1 Microsoft.Data.SqlClient

### Pros

- Full SQL control
- Maximum transparency
- Minimal abstraction
- Excellent performance
- Familiar database-centric model

### Cons

- More boilerplate code
- Manual mapping
- Higher maintenance cost
- Less alignment with modern .NET practices

### Decision

Rejected as the primary data access strategy but remains available when EF Core abstractions are not appropriate.

---

## 5.2 Dapper

### Pros

- Lightweight
- SQL-centric
- Excellent performance
- Reduced mapping effort

### Cons

- Additional dependency
- Reduced educational value compared to EF Core
- Less alignment with desired portfolio objectives

### Decision

Rejected as the primary data access strategy.

---

## 5.3 Stored Procedure-Centric Architecture

### Pros

- Strong database governance
- Explicit execution plans
- Centralized validation
- Consistent interaction patterns
- Excellent support for reporting and ETL

### Cons

- Higher development overhead
- Increased deployment coordination
- Reduced flexibility for simple CRUD scenarios
- Larger maintenance burden

### Decision

Rejected as the exclusive application architecture.

Stored procedures remain an approved architectural pattern where appropriate.

---

# 6. DbContext Strategy

The platform shall initially implement a single DbContext.

```csharp
FitnessDbContext
```

The DbContext may contain entities from multiple schemas.

Examples:

```text
audit.ErrorLog
audit.ApplicationLog
dbo.Exercise
dbo.Workout
dbo.Program
dbo.User
```

Multiple schemas alone do not justify multiple DbContexts.

---

# 7. Future Architectural Considerations

Additional DbContexts may be introduced when justified by:

- Separate databases
- Independent deployment cycles
- Security boundaries
- Distinct bounded contexts
- Independent scaling requirements
- Separate operational ownership

Examples:

```text
FitnessDbContext
IdentityDbContext
ReportingDbContext
LoggingDbContext
```

Such separation shall be driven by demonstrated architectural need rather than anticipated future requirements.

-
