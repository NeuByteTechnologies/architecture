# ADR-013: Database Governance Standards

**Status:** Accepted
**Date:** 2026-11-08
**Owner:** Architecture & Systems Design

---

# 1. Context

The Fitness platform utilizes a single SQL Server database containing multiple schemas with distinct architectural responsibilities.

These schemas support:

- Operational transaction processing
- Diagnostics and observability
- Data warehousing
- Reporting and analytics
- Governance and engineering metrics

A single governance model applied uniformly across all schemas introduces unnecessary constraints and does not reflect the differing responsibilities of operational, audit, warehouse, and reporting structures.

The platform therefore requires a unified governance strategy that establishes common governance principles while allowing schema-specific implementations.

---

# 2. Decision

The Fitness platform shall implement governance standards based upon schema responsibility.

The governance strategy shall support:

- Data traceability
- Auditability
- Operational monitoring
- Historical analysis
- Data lineage
- Reporting consistency
- Data warehouse integration

Governance requirements shall vary by schema purpose rather than enforcing identical governance columns across all database objects.

---

# 3. Governance Principles

## 3.1 Data Traceability

Significant business data shall be traceable throughout its lifecycle.

Traceability may include:

- Record creation
- Record modification
- Record deletion
- Record ownership
- Historical versioning
- ETL lineage
- Warehouse lineage

The implementation mechanism may vary by schema.

---

## 3.2 Database as Source of Truth

SQL Server remains the authoritative source of truth.

Governance controls may be implemented through:

- Constraints
- Views
- Stored Procedures
- Triggers
- ETL Processes
- Validation Tests

Application logic shall not circumvent governance controls.

---

## 3.3 Explicit Governance

Governance requirements shall be documented and validated through automated testing whenever practical.

Database governance standards should be enforced through:

- Build validation
- Deployment validation
- Continuous integration pipelines

---

## 3.4 Reporting Consistency

Reporting consumers should not require awareness of:

- Soft delete implementation
- Data warehouse structure
- Lifecycle management implementation details

Reporting structures shall expose consistent business semantics.

---

## 3.5 DbContext Boundary Governance

Entity Framework Core DbContexts shall be aligned to ownership, security, and operational responsibilities rather than solely to physical database boundaries.

A SQL Server database may contain multiple schemas that support different operational responsibilities and security requirements.

Separate DbContexts provide:

- Stronger separation of concerns
- Reduced accidental coupling
- Principle of least privilege
- Independent security models
- Independent connection strings
- Future scalability of infrastructure services

The existence of a schema does not automatically require a dedicated DbContext.

A separate DbContext should only be introduced when a schema represents a distinct operational, security, or ownership boundary.

---

# 4. Schema Governance Standards

## 4.1 dbo (Operational OLTP) Schema Governance

### Purpose

The dbo schema contains operational OLTP entities representing current application state.

Examples:

- User
- Workout
- Exercise
- Program
- Notification

### Required Governance Columns

Mutable business entities shall normally contain:

```sql
created_date          DATETIME2
created_user_id       INT NULL
last_update_date      DATETIME2
last_update_user_id   INT NULL
deleted_flag          BIT
```

### Optional Governance Columns

The following columns may be implemented when additional auditability is required:

```sql
deleted_date          DATETIME2 NULL
deleted_user_id       INT NULL
```

### Governance Objectives

The dbo schema supports:

- Record lifecycle tracking
- User accountability
- Operational auditing
- Soft deletion
- Data quality management

---

## 4.2 audit (Diagnostics & Observability) Schema Governance

### Purpose

The audit schema contains diagnostics, telemetry, auditing, and observability structures.

Examples:

- ErrorLog
- ErrorCatalog
- Severity
- Verbosity
- EtlRunLog

### Governance Model

Audit structures are generally append-only.

Historical visibility is preferred over lifecycle management.

### Typical Governance Attributes

Depending upon table purpose:

```sql
created_date
correlation_id
reference_code
severity_id
source_system
```

### Soft Delete Guidance

Append-only logging structures should not implement soft deletion unless explicitly justified.

### Governance Objectives

The audit schema supports:

- Diagnostics
- Root cause analysis
- Operational monitoring
- System observability
- Capacity planning
- Reliability analysis

---

## 4.2.1 AuditDbContext

The audit schema shall be accessed through a dedicated AuditDbContext.

The AuditDbContext represents an infrastructure concern and supports:

- Logging
- Diagnostics
- Observability
- ETL monitoring

Typical entities include:

- ErrorLog
- ErrorCatalog
- Severity
- Verbosity
- EtlRunLog

Business domain entities shall not be exposed through AuditDbContext.

AuditDbContext may utilize a dedicated connection string and a dedicated SQL login restricted to the audit schema.

---

## 4.3 star (Data Warehouse) Schema Governance

### Purpose

The star schema supports analytical workloads and data warehousing.

Examples:

- FactWorkoutLog
- FactUserWeight
- FactUserExerciseLog
- FactUserExerciseSetLog
- DimUser
- DimDate
- DimExercise
- DimErrorCode

### Governance Model

Governance focuses on lineage, historical accuracy, and ETL traceability rather than operational auditing.

### Fact Table Standards

Fact tables should support lineage and ETL traceability.

Typical governance attributes:

```sql
load_date
etl_batch_id
source_system
```

Additional lineage columns may be added as warehouse requirements evolve.

### Dimension Table Standards

Dimensions supporting historical analysis should implement Slowly Changing Dimension patterns when required.

Typical governance attributes:

```sql
effective_start_date
effective_end_date
is_current_record
etl_batch_id
```

### Unknown Member Standards

Dimensions should contain an Unknown member record where practical.

Typical implementation:

```text
Surrogate Key = 0
Business Key = Unknown
```

This supports warehouse integrity when source values are missing or invalid.

### Governance Objectives

The star schema supports:

- Historical reporting
- Trend analysis
- Slowly changing dimensions
- Data lineage
- Analytical consistency
- Power BI analytics

---

## 4.4 datamart (Reporting & Analytics) Schema Governance

### Purpose

The datamart schema exposes reporting-friendly structures for downstream consumers.

Examples:

- vwWorkoutSummary
- vwUserProgress
- vwApplicationHealth
- vwErrorAnalysis

### Governance Model

Datamart objects typically consist of views and reporting projections.

Governance focuses on reporting correctness rather than ownership of audit columns.

### Reporting Standards

Views should:

- Present current-state business information
- Exclude soft-deleted records when appropriate
- Simplify reporting consumption
- Hide implementation details
- Support stable reporting semantics

### Example

```sql
CREATE VIEW datamart.vwExercise
AS
SELECT
    exercise_id,
    exercise_name,
    exercise_type_id
FROM dbo.exercise
WHERE deleted_flag = 0;
```

### Governance Objectives

The datamart schema supports:

- Reporting consistency
- Reduced dashboard complexity
- Simplified Power BI models
- Stable reporting semantics

---

## 4.5 Schema Security Boundaries

Database schemas should support separation of security concerns.

Where practical, SQL Server roles and service accounts should be granted permissions only to the schemas required by their operational responsibilities.

| Schema | Typical Access |
|----------|----------|
| dbo | Application Runtime |
| audit | Logging and Monitoring Services |
| github | Governance and ETL Services |
| star | ETL Services |
| datamart | Reporting and Analytics Consumers |

The architecture favors least-privilege access through schema-level permissions.

DbContext boundaries should reinforce schema security boundaries.

### Guiding Principle

```text
New Schema
    ≠
New DbContext

New Security Boundary
    =
Candidate DbContext
```

---

# 5. Reference Data Governance

Reference data is governed data.

Examples:

- ErrorCode
- Severity
- WorkoutType
- NotificationType
- EventType

Reference tables support:

- Application behavior
- Reporting dimensions
- Diagnostics
- ETL processing
- Warehouse structures

Reference data should follow the same governance standards as operational data when mutable.

---

## 5.1 Application Alignment

Where application enumerations exist, the database remains authoritative.

Examples:

- ErrorCode
- Severity

Automated tests shall validate alignment between:

```text
Application Definitions
          ↔
Database Reference Data
```

The purpose of these tests is to prevent drift between application code and governed database values.

---

# 6. Governance Validation

Governance standards should be validated through automated database testing and CI/CD pipelines.

Examples include:

```text
ADR013_Dbo_GovernanceValidation.sql
ADR013_AuditValidation.sql
ADR013_DimensionValidation.sql
ADR013_FactValidation.sql
ADR013_DatamartValidation.sql
ADR013_ReferenceDataValidation.sql
```

Additional validation should include:

```text
ADR013_DataTypeValidation.sql
ADR013_UnknownMemberValidation.sql
ADR013_CrossSchemaAlignmentValidation.sql
```

Validation scripts should return violations and fail automated builds when governance standards are not satisfied.

Governance standards should be treated as executable architectural rules.

---

# 7. Data Warehouse and ETL Considerations

The governance model shall support future ETL operations.

Governed structures should provide sufficient metadata to support:

- Fact creation
- Dimension creation
