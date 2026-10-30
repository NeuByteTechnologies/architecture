# ADR-012: Migration from Azure MySQL to SQL Server 2025

**Status:** Accepted
**Date:** 2026-10-29
**Owner:** Architecture & Systems Design
**Supersedes:** ADR-001 Database Choice – Azure MySQL

---

# 1. Context

ADR-001 selected Azure Database for MySQL as the primary relational database platform for the Fitness ecosystem.

Since that decision was made, the project has evolved significantly through:

- Error Handling Framework development
- Audit Logging Framework development
- Adoption of Entity Framework Core
- Expanded SQL Server architectural design
- Portfolio development objectives
- Increased emphasis on enterprise .NET patterns

Additionally, implementation activities identified a growing dependency on:

- SQL Server data types
- T-SQL familiarity
- Audit schema design
- Entity Framework Core SQL Server integration
- Microsoft SQL Server tooling

The assumptions that originally supported Azure MySQL are no longer aligned with the project's current direction.

---

# 2. Decision

The Fitness platform shall use:

```text
SQL Server 2025
```

as the primary relational database platform.

SQL Server becomes the authoritative system of record for:

- Application data
- Audit logs
- Reference data
- Fitness domain entities
- Reporting sources
- Future analytical workloads

ADR-001 is superseded by this decision.

---

# 3. Rationale

## 3.1 Existing Professional Experience

The primary project owner possesses extensive experience with:

- SQL Server
- T-SQL
- Database administration
- ETL development
- Power BI
- Reporting solutions
- Data warehouse design

Leveraging existing expertise allows development effort to focus on:

```text
.NET
Blazor
Entity Framework Core
Application Architecture
Test-Driven Development
```

rather than simultaneously learning an unfamiliar database platform.

---

## 3.2 Audit Logging Architecture

Recent architectural decisions established:

```text
audit.ErrorLog
audit schema ownership
error classification
verbosity tracking
correlation tracking
```

The logging framework was designed around SQL Server capabilities and conventions.

Examples include:

- audit schema
- tinyint enum mappings
- T-SQL familiarity
- SQL-based operational reporting

SQL Server aligns naturally with the logging architecture.

---

## 3.3 Entity Framework Core Alignment

ADR-011 adopted:

```text
Microsoft.EntityFrameworkCore.SqlServer
```

as the application's primary persistence strategy.

SQL Server provides:

- First-class EF Core support
- Mature tooling
- Long-term Microsoft ecosystem alignment
- Extensive documentation
- Community support

This reduces architectural complexity and minimizes technology fragmentation.

---

## 3.4 Portfolio Objectives

The platform is intended to demonstrate competency in:

- ASP.NET Core
- Blazor
- Entity Framework Core
- SQL Server
- Azure
- Software Architecture
- Test-Driven Development

The combination of:

```text
Blazor
ASP.NET Core
Entity Framework Core
SQL Server
```

represents a widely adopted enterprise stack that provides stronger portfolio value than maintaining MySQL solely for historical reasons.

---

## 3.5 Single Source of Truth Philosophy

A recurring architectural principle throughout the project is:

```text
Single Source of Truth
```

This philosophy influences decisions involving:

- Database ownership
- Audit records
- Reporting
- Analytics
- Data governance

SQL Server aligns well with this principle due to the project's existing familiarity with:

- Constraints
- Referential integrity
- Data governance
- Reporting integrations
- Operational monitoring

---

# 4. Alternatives Considered

## 4.1 Azure Database for MySQL

### Pros

- Managed Azure service
- Good cloud integration
- Lower operational overhead
- Strong open-source ecosystem

### Cons

- Lower alignment with existing experience
- Additional learning curve
- Reduced benefit for current project goals
- Less integration with SQL Server expertise

### Decision

Rejected.

The original rationale remains valid but no longer represents the best fit for the evolved architecture.

---

## 4.2 PostgreSQL

### Pros

- Advanced SQL features
- Strong open-source community
- Excellent performance

### Cons

- Additional learning curve
- Not aligned with current development goals
- Reduced leverage of existing expertise

### Decision

Rejected.

---

## 4.3 Continued Polyglot Database Strategy

### Pros

- Exposure to multiple platforms
- Broader technical experience

### Cons

- Increased complexity
- Slower development velocity
- Reduced architectural consistency

### Decision

Rejected.

The project should optimize for depth of implementation rather than breadth of database technologies.

---

# 5. Architectural Implications

The following technologies are standardized for the platform:

## Database Platform

```text
SQL Server 2025
```

## Data Access

```text
Entity Framework Core
Microsoft.EntityFrameworkCore.SqlServer
```

## Database Connectivity

```text
Microsoft.Data.SqlClient
```

when direct SQL access is required.

## Query Language

```text
T-SQL
```

for administrative, operational, reporting, and specialized data operations.

---

# 6. Migration Strategy

Future database objects shall target SQL Server conventions.

Examples include:

```text
Schemas
Stored Procedures
Views
Functions
Indexes
Constraints
Audit Tables
```

Existing MySQL-specific assumptions shall be retired as components are updated.

No effort shall be made to maintain database-provider neutrality unless a future requirement explicitly demands it.

---

# 7. Consequences

## Positive

- Leverages existing expertise
- Supports EF Core adoption
- Accelerates development
- Improves architectural consistency
- Simplifies operational design
- Strengthens portfolio relevance
- Reduces technology switching costs

## Negative

- Reduced exposure to MySQL
- Increased alignment with Microsoft ecosystem
- Less database portability

---

# 8. Architectural Principles Preserved

The migration to SQL Server does not change the following principles:

```text
Database remains the authoritative source of truth.
Data integrity is enforced in the database.
Application code complements database governance.
Auditability remains a first-class concern.
Reporting and operational analytics remain supported.
```

---

# 9. Related Documents

- ADR-001 Database Choice – Azure MySQL (Superseded)
- ADR-010 Logging Severity and Verbosity Strategy
- ADR-011 Entity Framework Core Data Access Strategy
- DataArchitecture.md
- ApplicationArchitecture.md
- SystemArchitecture.md

---

# 10. Version History

| Version | Date | Author | Description |
|----------|----------|----------|----------|
| 1.0 | 2026-10-29 | Gordon | Superseded ADR-001 and adopted SQL Server 2025 as the primary relational database platform. |
