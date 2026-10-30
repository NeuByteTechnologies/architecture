# ADR-013: Data Governance and Audit Standards

**Status:** Accepted
**Date:** 2026-10-30
**Owner:** Architecture & Systems Design

---

# 1. Context

The Fitness platform contains operational data, audit data, diagnostics data, analytical data, and reporting data.

As the platform evolves, consistency in data governance becomes increasingly important.

Without established governance standards, risks include:

- Inconsistent audit capabilities
- Incomplete change tracking
- Difficult root-cause analysis
- Reporting inaccuracies
- Data quality degradation
- ETL complexity
- Inconsistent soft-delete implementations
- Reduced operational observability

A formal governance standard is required to ensure all database objects support auditing, diagnostics, reporting, and future analytics requirements.

---

# 2. Decision

The Fitness platform shall implement standardized data governance and auditing practices across all database schemas.

The database shall support:

- Record lifecycle tracking
- User accountability
- Soft deletion
- Historical analysis
- Reporting consistency
- Diagnostics and operational monitoring
- Data warehouse integration

Governance requirements shall apply to operational, audit, diagnostic, warehouse, and reporting structures where appropriate.

---

# 3. Governance Principles

## 3.1 Data Traceability

All significant business records shall be traceable throughout their lifecycle.

The platform shall preserve information regarding:

- Record creation
- Record modification
- Record deletion
- Record ownership
- Data lineage

---

## 3.2 Database as Source of Truth

SQL Server remains the authoritative source of truth.

Application logic shall not override governance rules enforced at the database level.

Governance controls may be implemented through:

- Constraints
- Stored Procedures
- Views
- Triggers (when justified)
- ETL processes

---

## 3.3 Soft Delete Preference

The platform shall prefer soft deletion over physical deletion for business-critical data.

Soft deletion preserves:

- Historical reporting
- Auditability
- Diagnostics
- Data recovery capabilities

Physical deletion shall occur only when explicitly justified.

---

## 3.4 Reporting Consistency

Reporting consumers should not be required to understand record lifecycle management.

Reporting views shall expose current-state data by default.

Soft-deleted records shall normally be excluded from reporting views unless the reporting purpose explicitly requires historical visibility.

---

# 4. Standard Governance Columns

Unless a documented exception exists, mutable tables shall include the following governance columns.

## Required Columns

```sql
created_date          DATETIME2
created_user_id       INT NULL

last_update_date      DATETIME2
last_update_user_id   INT NULL

deleted_flag          BIT
```

---

## Optional Columns

The following columns may be implemented when additional governance requirements exist.

```sql
deleted_date          DATETIME2 NULL
deleted_user_id       INT NULL

effective_start_date  DATETIME2 NULL
effective_end_date    DATETIME2 NULL
```

These columns are particularly useful for:

- Regulatory tracking
- Historical reporting
- Slowly changing dimensions
- Master data management

---

# 5. Schema Governance Standards

## 5.1 dbo Schema

The dbo schema contains operational OLTP entities.

Examples:

```text
User
Workout
Exercise
Program
Notification
```

Governance columns are required on mutable business entities.

---

## 5.2 audit Schema

The audit schema contains:

- Logging
- Diagnostics
- Auditing
- Telemetry
- Operational monitoring

Examples:

```text
ApplicationLog
ErrorLog
ErrorCode
ErrorSeverity
EventType
ProcedureExecutionLog
```

The audit schema shall provide complete historical visibility whenever practical.

Soft deletion may be omitted for append-only logging structures.

---

## 5.3 dw Schema

The Data Warehouse schema supports:

- Fact tables
- Dimension tables
- Historical analysis
- Power BI reporting

Governance metadata should be preserved when useful for lineage and auditing.

Examples:

```text
FactWorkout
FactApplicationLog
DimUser
DimDate
DimSeverity
DimErrorCode
```

---

## 5.4 datamart Schema

The datamart schema represents reporting-friendly views and projections.

Examples:

```text
vwWorkoutSummary
vwApplicationHealth
vwUserProgress
vwErrorAnalysis
```

Governance columns may be hidden from reporting consumers when not required.

---

# 6. Reporting and View Standards

Reporting views shall generally expose only active records.

Example:

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

Benefits include:

- Consistent reporting behavior
- Reduced dashboard complexity
- Cleaner Power BI models
- Reduced risk of reporting retired records

Exceptions shall be documented.

---

# 7. Reference Data Governance

Reference data is governed data.

Examples:

```text
ErrorCode
ErrorSeverity
NotificationType
WorkoutType
EventType
```

Reference tables may include governance columns even when values change infrequently.

Reference data supports:

- Application behavior
- Diagnostics
- Reporting dimensions
- ETL processes
- Data warehouse structures

---

## 7.1 Application Alignment

When application enums exist for reference data:

```csharp
ErrorSeverity
ErrorCode
```

the database remains authoritative.

Automated integration tests shall validate alignment between:

```text
Application Definitions
          ↔
Database Reference Data
```

The purpose of these tests is to prevent drift between code and governed database values.

---

# 8. Diagnostics and Execution Logging

The platform shall support operational diagnostics through audit and telemetry structures.

Examples include:

```text
ApplicationLog
ErrorLog
SecurityLog
ApiRequestLog
PerformanceLog
ProcedureExecutionLog
JobExecutionLog
```

These structures support:

- Root cause analysis
- Operational monitoring
- Capacity planning
- Performance tuning
- Database health reviews

---

## Example Procedure Execution Tracking

```sql
procedure_execution_log
```

Potential attributes:

```sql
execution_id
procedure_name
execution_start_date
execution_end_date
duration_ms
rows_affected
status
error_code
created_date
```

This information can support:

- Index maintenance decisions
- Query optimization
- Performance analysis
- Operational dashboards
- Reliability monitoring

---

# 9. Data Warehouse and ETL Considerations

The governance model shall support future ETL operations.

Governed data structures should provide sufficient metadata to support:

- Fact creation
- Dimension creation
- Historical tracking
- Data lineage
- Reporting validation

The platform shall favor designs that are compatible with:

```text
OLTP
    ↓
Audit & Diagnostics
    ↓
ETL
    ↓
Data Warehouse
    ↓
Datamart
    ↓
Power BI
```

---

# 10. Consequences

## Positive

- Consistent auditing
- Improved diagnostics
- Easier reporting
- Better Power BI integration
- Enhanced operational visibility
- Stronger governance
- Improved historical traceability
- Reduced reporting complexity

## Negative

- Additional storage requirements
- Additional development effort
- More governance-related testing
- Increased schema discipline requirements

---

# 11. Related Documents

- ADR-001 Database Choice
- ADR-010 Logging Severity and Verbosity Strategy
- ADR-011 Entity Framework Core Data Access Strategy
- DataArchitecture.md
- ReportingArchitecture.md
- SystemArchitecture.md

---

# 12. Version History

| Version | Date | Author | Description |
|----------|------------|----------|-------------|
| 1.0 | 2026-10-30 | Gordon | Initial governance and audit standards definition covering lifecycle tracking, soft deletes, reporting standards, diagnostics, and operational observability. |
