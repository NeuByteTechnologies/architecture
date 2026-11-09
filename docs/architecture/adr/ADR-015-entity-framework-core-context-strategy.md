# ADR-015: Entity Framework Core Context Strategy

**Status:** Accepted
**Date:** 2026-11-08
**Owner:** Architecture & Systems Design

---

# 1. Context

The Fitness platform utilizes:

- ASP.NET Core
- Entity Framework Core
- SQL Server 2025
- Multiple database schemas
- Operational data
- Audit infrastructure
- Future governance and analytics capabilities

As the platform architecture evolved, a need emerged to establish clear guidance regarding:

- DbContext responsibilities
- Schema ownership
- Infrastructure boundaries
- Domain boundaries
- Future extensibility

Early designs assumed that a schema and a DbContext would often align. Further architectural analysis determined that this approach creates unnecessary complexity and does not adequately reflect application ownership boundaries.

A formal strategy is required to govern DbContext design as the platform continues to grow.

---

# 2. Decision

Entity Framework Core DbContexts shall represent application and infrastructure boundaries rather than database schema boundaries.

The platform shall create DbContexts only when a distinct architectural responsibility exists.

The existence of a database schema alone shall not justify creation of a separate DbContext.

New DbContexts shall require:

- Distinct ownership
- Distinct purpose
- Distinct lifecycle
- Distinct architectural responsibility

---

# 3. Architectural Principles

## 3.1 DbContexts Represent Architectural Boundaries

A DbContext is an application architecture construct.

A database schema is a database architecture construct.

These concepts are related but not identical.

The decision to create a DbContext shall be driven primarily by application architecture concerns.

---

## 3.2 Avoid Context Proliferation

New DbContexts introduce:

- Additional configuration
- Additional testing requirements
- Additional dependency injection registrations
- Additional maintenance

DbContexts should not be created without clear architectural justification.

The platform favors fewer well-defined contexts over numerous schema-aligned contexts.

---

## 3.3 Single Responsibility

Each DbContext should have a clearly defined responsibility.

Examples:

```text
Business Operations

Audit and Observability

Governance Analytics
```

DbContexts should avoid mixing unrelated concerns.

---

## 3.4 Infrastructure Isolation

Cross-cutting infrastructure concerns should be isolated from business-domain concerns whenever practical.

Examples include:

- Logging
- Diagnostics
- Observability
- ETL monitoring

Isolation improves maintainability and reduces accidental coupling.

---

# 4. Context Classification

## 4.1 Business Domain Contexts

Business contexts support application functionality.

Examples include:

- User management
- Workout tracking
- Exercise management
- Notification management
- Preferences

Business contexts represent application behavior.

---

## 4.2 Infrastructure Contexts

Infrastructure contexts support platform operations.

Examples include:

- Logging
- Diagnostics
- Audit tracking
- Observability

Infrastructure contexts support platform reliability rather than business functionality.

---

## 4.3 Future Governance Contexts

Governance contexts support:

- Engineering analytics
- SDLC reporting
- Repository governance
- Portfolio reporting

Governance contexts are distinct from both business and infrastructure concerns.

---

# 5. Current Context Strategy

## 5.1 AuditDbContext

### Purpose

AuditDbContext provides persistence for:

- Error logging
- Error catalog management
- Severity metadata
- Verbosity metadata
- ETL execution monitoring

### Supported Schema

```text
audit
```

### Typical Entities

```text
ErrorLog

ErrorCatalog

Severity

Verbosity

EtlRunLog
```

### Architectural Rationale

The audit schema represents an infrastructure concern.

Audit functionality is:

- Cross-cutting
- Append-oriented
- Operationally focused

Separating audit functionality from business functionality improves maintainability and reduces coupling.

---

# 6. Deferred Contexts

## 6.1 GitHubDbContext

### Status

Deferred

### Potential Purpose

Repository governance and SDLC analytics.

Potential objects include:

- Repositories
- Issues
- Labels
- Milestones
- Pull requests
- Workflow executions

### Justification

The github schema represents governance metadata rather than business-domain functionality.

A dedicated context may be introduced once sufficient functionality exists to justify its maintenance.

---

# 7. Rejected Approaches

## 7.1 Context per Schema

### Example

```text
dbo Context

audit Context

star Context

datamart Context

github Context
```

### Rationale

Rejected because schemas do not automatically represent independent architectural responsibilities.

This approach introduces unnecessary complexity and maintenance overhead.

---

## 7.2 Single Global Context

### Example

```text
FitnessContext
```

Containing:

```text
dbo

audit

star

datamart

github
```

### Rationale

Rejected because it creates excessive coupling between:

- Business operations
- Observability
- Analytics
- Governance

The platform favors bounded responsibilities.

---

# 8. DbContext Creation Criteria

A new DbContext should generally satisfy one or more of the following conditions:

### Ownership Boundary

The data is owned by a different architectural concern.

### Lifecycle Boundary

The data is managed through a different operational lifecycle.

### Infrastructure Boundary

The data supports cross-cutting technical capabilities.

### Governance Boundary

The data supports governance, reporting, or analytical objectives.

---

# 9. Relationship to Database Architecture

DbContexts do not determine database architecture.

Database architecture is governed by:

- ADR-012 Database Platform Strategy
- ADR-013 Database Governance Standards

DbContexts provide application access patterns over those structures.

The relationship can be represented as:

```text
SQL Server
    ↓

Schemas
    ↓

Entity Framework Core
    ↓

DbContexts
    ↓

Application Services
```

---

# 10. Future Considerations

Future platform evolution may introduce:

- Repository governance services
- Enterprise reporting services
- Dedicated ETL processing
- Analytical platforms

Additional DbContexts may be introduced when justified by architecture, ownership, or operational concerns.

All future DbContexts should comply with the principles defined in this ADR.

---

# 11. Consequences

## Positive

- Clear architectural boundaries
- Reduced coupling
- Simpler maintenance
- Improved separation of concerns
- Easier testing
- Better support for infrastructure services
- Improved long-term scalability

## Negative

- Additional context management
- Additional dependency injection configuration
- More architectural review required before introducing new contexts

---

# 12. Related Documents

- ADR-011 Entity Framework Core Data Access Strategy
- ADR-012 Migration from Azure MySQL to SQL Server 2025
- ADR-013 Database Governance Standards
- ADR-014 Security Architecture
- ApplicationArchitecture.md
- DataArchitecture.md
- InfrastructureArchitecture.md

---

# 13. Version History

| Version | Date | Author | Description |
|----------|----------|----------|----------|
| 1.0 | 2026-11-08 | Gordon | Established Entity Framework Core DbContext strategy, AuditDbContext architecture, future governance context guidance, and DbContext creation principles. |
