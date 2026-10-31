# ADR-013: Database Governance Standards

**Status:** Accepted
**Date:** 2026-10-31
**Owner:** Architecture & Systems Design

---

# 1. Context

The Fitness platform utilizes a single SQL Server database containing multiple schemas with distinct architectural responsibilities.

These schemas support:

- Operational transaction processing
- Diagnostics and observability
- Data warehousing
- Reporting and analytics

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

Database governance standards should be enforced through build validation, deployment validation, and continuous integration pipelines.

---

## 3.4 Reporting Consistency

Reporting consumers should not require awareness of:

- Soft delete implementation
- Data warehouse structure
- Lifecycle management implementation details

Reporting structures shall expose consistent business semantics.

---

# 4. Schema Governance Standards

## 4.1 dbo Schema Governance

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

## 4.2 audit Schema Governance

### Purpose

The audit schema contains diagnostics, telemetry, auditing, and observability structures.

Examples:

- ApplicationLog
- ErrorLog
- SecurityLog
- PerformanceLog
- ApiRequestLog
- ProcedureExecutionLog
- JobExecutionLog

### Governance Model

Audit structures are generally append-only.

Historical visibility is preferred over lifecycle management.

### Typical Governance Attributes

Depending upon table purpose:

```sql
created_date
correlation_id
reference_code
event_type_id
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

## 4.3 star Schema Governance

### Purpose

The star schema supports analytical workloads and data warehousing.

Examples:

- FactWorkout
- FactApplicationLog
- FactApiRequest
- DimUser
- DimDate
- DimErrorCode
- DimSeverity

### Governance Model

Governance focuses on lineage, historical accuracy, and ETL traceability rather than operational auditing.

---

### Fact Table Standards

Fact tables should support lineage and ETL traceability.

Typical governance attributes:

```sql
load_date
etl_batch_id
source_system
```

Additional lineage columns may be added as warehouse requirements evolve.

---

### Dimension Table Standards

Dimensions supporting historical analysis should implement Slowly Changing Dimension patterns when required.

Typical governance attributes:

```sql
effective_start_date
effective_end_date
current_flag
etl_batch_id
```

### Governance Objectives

The star schema supports:

- Historical reporting
- Trend analysis
- Slowly changing dimensions
- Data lineage
- Analytical consistency
- Power BI analytics

---

## 4.4 datamart Schema Governance

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

# 5. Reference Data Governance

Reference data is governed data.

Examples:

- ErrorCode
- ErrorSeverity
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
- ErrorSeverity

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
ADR013_dbo_GovernanceValidation.sql
ADR013_AuditValidation.sql
ADR013_DimensionValidation.sql
ADR013_FactValidation.sql
ADR013_DatamartValidation.sql
ADR013_ReferenceDataValidation.sql
```

Validation scripts should return violations and fail automated builds when governance standards are not satisfied.

Governance standards should be treated as executable architectural rules.

---

# 7. Data Warehouse and ETL Considerations

The governance model shall support future ETL operations.

Governed structures should provide sufficient metadata to support:

- Fact creation
- Dimension creation
- Historical tracking
- Data lineage
- Reporting validation

The platform architecture shall favor:

```text
OLTP (dbo)
      ↓
Audit & Diagnostics (audit)
      ↓
ETL
      ↓
Data Warehouse (star)
      ↓
Datamart
      ↓
Power BI
```

---

# 8. Consequences

## Positive

- Consistent governance strategy
- Clear schema responsibilities
- Improved diagnostics
- Better historical reporting
- Stronger data lineage
- Easier CI/CD validation
- Better Power BI integration
- Reduced reporting ambiguity

## Negative

- Additional governance documentation
- Additional validation requirements
- Increased testing effort
- Increased schema discipline requirements

---

# 9. Related Documents

- ADR-001 Database Choice
- ADR-010 Logging Severity and Verbosity Strategy
- ADR-011 Entity Framework Core Data Access Strategy
- ADR-012 Database Architecture
- DataArchitecture.md
- ReportingArchitecture.md
- SystemArchitecture.md

---

# 10. Version History

| Version | Date | Author | Description |
|----------|------------|----------|-------------|
| 2.0 | 2026-10-31 | Gordon | Revised governance strategy from universal governance columns to schema-specific governance standards supporting operational, audit, warehouse, and reporting responsibilities. |
