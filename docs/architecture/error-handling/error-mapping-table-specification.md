# FS-ERROR-MAPPING-TABLE-SPECIFICATION

## 1. Document Overview

### Purpose

This specification defines the Exception Mapping Framework for the Fitness Application Error Handling Architecture. It establishes standard exception classification, error condition mapping, severity assignment, propagation rules, response behavior, telemetry requirements, and auditing semantics.

### Scope

This specification applies to all application layers:

- Presentation Layer (UI / API)
- Middleware Layer
- Application Services Layer
- Domain Layer
- Data Access Layer
- External Integration Layer

### Related Artifacts

#### Business Requirements

- BR-ERROR-01
- BR-ERROR-PIPELINE-02

#### Use Cases

- UC-ERROR-HANDLING-01
- UC-VALIDATION-02

#### Functional Specifications

- FS-ERROR-HIERARCHY
- FS-MIDDLEWARE-FLOW

#### Data Specification Fields

- error.code
- error.type
- error.severity

#### Test Cases

- TC-EXCEPTION-MAP-01
- TC-EXCEPTION-MAP-NEG-01

---

# 2. Error Handling Objectives

The Exception Mapping Framework shall:

- Standardize exception classification.
- Standardize error condition identification.
- Ensure consistent client responses.
- Prevent leakage of internal implementation details.
- Support application observability.
- Support telemetry and auditing.
- Enable deterministic middleware processing.
- Provide consistent error codes across applications.
- Support future enterprise-wide governance and monitoring initiatives.

---

# 3. Exception Hierarchy

## 3.1 Logical Hierarchy

```text
BaseApplicationException
│
├── ValidationException
│   ├── RequiredFieldException
│   ├── FormatValidationException
│   └── BusinessRuleValidationException
│
├── AuthorizationException
│   ├── AuthenticationFailedException
│   └── PermissionDeniedException
│
├── BusinessException
│   ├── WorkoutRuleException
│   ├── NutritionRuleException
│   └── MembershipRuleException
│
├── IntegrationException
│   ├── ExternalApiException
│   ├── TimeoutException
│   └── ThirdPartyServiceException
│
├── DataAccessException
│   ├── DatabaseException
│   ├── ConcurrencyException
│   └── DataIntegrityException
│
└── SystemException
    ├── ConfigurationException
    ├── InfrastructureException
    └── UnexpectedException
```

---

# 4. Severity Classification

## 4.1 Severity Definitions

| Severity | Description | Operational Impact |
|-----------|-------------|-------------------|
| INFO | Informational event | No interruption |
| LOW | Minor recoverable issue | Limited impact |
| MEDIUM | Functional degradation | User action required |
| HIGH | Business operation failure | Process halted |
| CRITICAL | Application or infrastructure failure | Immediate attention required |

## 4.2 Severity Enumeration

```csharp
public enum ErrorSeverity : byte
{
    Info = 0,
    Low = 1,
    Medium = 2,
    High = 3,
    Critical = 4
}
```

---

# 5. Exception Mapping Table

## 5.1 Validation Exceptions

| Exception Type | Error Code | Severity | HTTP Status | Client Visible | Retry Allowed |
|----------------|------------|----------|-------------|----------------|---------------|
| RequiredFieldException | VAL-001 | LOW | 400 | Yes | No |
| FormatValidationException | VAL-002 | LOW | 400 | Yes | No |
| BusinessRuleValidationException | VAL-003 | MEDIUM | 422 | Yes | No |

---

## 5.2 Authentication and Authorization Exceptions

| Exception Type | Error Code | Severity | HTTP Status | Client Visible | Retry Allowed |
|----------------|------------|----------|-------------|----------------|---------------|
| AuthenticationFailedException | AUTH-001 | MEDIUM | 401 | Yes | No |
| PermissionDeniedException | AUTH-002 | MEDIUM | 403 | Yes | No |

---

## 5.3 Business Exceptions

| Exception Type | Error Code | Severity | HTTP Status | Client Visible | Retry Allowed |
|----------------|------------|----------|-------------|----------------|---------------|
| WorkoutRuleException | BUS-001 | MEDIUM | 422 | Yes | No |
| NutritionRuleException | BUS-002 | MEDIUM | 422 | Yes | No |
| MembershipRuleException | BUS-003 | MEDIUM | 422 | Yes | No |

---

## 5.4 Integration Exceptions

| Exception Type | Error Code | Severity | HTTP Status | Client Visible | Retry Allowed |
|----------------|------------|----------|-------------|----------------|---------------|
| ExternalApiException | INT-001 | HIGH | 502 | No | Yes |
| TimeoutException | INT-002 | HIGH | 504 | No | Yes |
| ThirdPartyServiceException | INT-003 | HIGH | 503 | Yes |

---

## 5.5 Database Exceptions

### Database Operation Failures

DatabaseException represents failures during database operations, connectivity, execution, transaction processing, and SQL Server engine interactions.

| Exception Type | Error Code | Severity | HTTP Status | Client Visible | Retry Allowed |
|----------------|------------|----------|-------------|----------------|---------------|
| DatabaseException | DATA-000 | HIGH | 500 | No | Conditional |
| DatabaseException | DATA-001 Connection Failure | CRITICAL | 500 | No | Yes |
| DatabaseException | DATA-002 Timeout | HIGH | 504 | No | Yes |
| DatabaseException | DATA-003 Deadlock | MEDIUM | 409 | No | Yes |
| DatabaseException | DATA-004 Arithmetic Overflow | HIGH | 500 | No | No |
| DatabaseException | DATA-005 Constraint Violation | MEDIUM | 422 | No | No |

---

## 5.6 Concurrency Exceptions

| Exception Type | Error Code | Severity | HTTP Status | Client Visible | Retry Allowed |
|----------------|------------|----------|-------------|----------------|---------------|
| ConcurrencyException | DATA-050 | HIGH | 409 | Yes | Yes |

---

## 5.7 Data Integrity Exceptions

### Data Integrity Failures

DataIntegrityException represents invalid, inconsistent, corrupted, or non-compliant business data that may not originate from a database engine failure.

Examples include:

- Data governance violations
- Referential inconsistencies
- Aggregate calculation mismatches
- Reconciliation failures
- Application-level data corruption

| Exception Type | Error Code | Severity | HTTP Status | Client Visible | Retry Allowed |
|----------------|------------|----------|-------------|----------------|---------------|
| DataIntegrityException | DATA-100 Data Integrity Failure | HIGH | 500 | No | No |
| DataIntegrityException | DATA-101 Referential Integrity Violation | HIGH | 500 | No | No |
| DataIntegrityException | DATA-102 Aggregate Consistency Failure | HIGH | 500 | No | No |
| DataIntegrityException | DATA-103 Governance Rule Violation | LOW | 422 | Yes | No |
| DataIntegrityException | DATA-104 Data Reconciliation Failure | HIGH | 500 | No | No |

---

## 5.8 System Exceptions

| Exception Type | Error Code | Severity | HTTP Status | Client Visible | Retry Allowed |
|----------------|------------|----------|-------------|----------------|---------------|
| ConfigurationException | SYS-001 | CRITICAL | 500 | No | No |
| InfrastructureException | SYS-002 | CRITICAL | 500 | No | Conditional |
| UnexpectedException | SYS-999 | CRITICAL | 500 | No | No |

---

# 6. Mapping Rules

## Rule 1: Explicit Mapping

Every custom exception class shall be mapped to one or more defined error codes representing specific error conditions.

---

## Rule 2: Severity Assignment

Every mapped error code shall define a severity classification.

---

## Rule 3: Unique Error Codes

Each error code shall uniquely identify exactly one exception condition.

Duplicate error codes are prohibited.

---

## Rule 4: Fallback Mapping

Unrecognized exceptions shall be mapped as:

```text
Exception Type : UnexpectedException
Error Code     : SYS-999
Severity       : CRITICAL
HTTP Status    : 500
```

---

## Rule 5: Client Response Protection

Internal implementation details including:

- Stack traces
- Connection strings
- SQL statements
- Infrastructure details

shall never be exposed outside the server boundary.

---

## Rule 6: Logging Requirement

All HIGH and CRITICAL severity conditions shall generate log entries.

Logging implementation details and logging levels are outside the scope of the Exception Mapping Framework.

---

## Rule 7: Observability Requirement

All exception mappings shall generate telemetry events.

Telemetry data shall include:

- Error Code
- Exception Type
- Severity
- Correlation Identifier
- Timestamp

---

# 7. Cross-Layer Propagation Rules

## 7.1 Propagation Principle

Exceptions originate in lower layers and propagate upward through the middleware pipeline.

```text
Data Layer
    │
    ▼
Domain Layer
    │
    ▼
Application Service Layer
    │
    ▼
Middleware Layer
    │
    ▼
API/UI Response
```

---

## 7.2 Translation Rules

| Source Layer | Original Condition | Translated Exception |
|--------------|-------------------|----------------------|
| Database Layer | SQL Connectivity Failure | DatabaseException |
| Database Layer | Deadlock | DatabaseException |
| Database Layer | Constraint Violation | DatabaseException |
| Third-Party API | Network Failure | ExternalApiException |
| Domain Service | Business Rule Violation | BusinessRuleValidationException |
| Authentication Service | Invalid Credentials | AuthenticationFailedException |
| Application Service | Data Consistency Failure | DataIntegrityException |
| Governance Service | Policy Violation | DataIntegrityException |

---

## 7.3 Termination Point

The Middleware Layer shall be the final exception handling boundary responsible for:

- Exception classification
- Error code assignment
- Severity assignment
- Logging
- Telemetry generation
- Response creation

---

# 8. Error Response Contract

## Standard Structure

```json
{
  "error": {
    "code": "DATA-001",
    "type": "DatabaseException",
    "severity": "CRITICAL",
    "message": "Database connectivity failure.",
    "correlationId": "GUID"
  }
}
```

---

## Field Definitions

| Field | Description |
|---------|------------|
| error.code | Unique mapped error code |
| error.type | Exception classification |
| error.severity | Assigned severity level |
| message | Client-safe error message |
| correlationId | End-to-end trace identifier |

---

# 9. Middleware Processing Semantics

```text
Exception Raised
      │
      ▼
Exception Interceptor
      │
      ▼
Classify Exception
      │
      ▼
Lookup Error Mapping
      │
      ▼
Assign Error Code
Assign Severity
      │
      ├── Log Event
      ├── Create Telemetry Event
      └── Build Error Response
                │
                ▼
         Return Response
```

---

# 10. Validation Rules

Deployment shall be rejected when:

- An exception class exists without a defined mapping.
- Duplicate error codes exist.
- Invalid severity values exist.
- Undefined propagation paths exist.
- Middleware mapping configuration is incomplete.
- Reserved error code ranges are violated.

---

# 11. Acceptance Criteria Traceability

| Acceptance Criterion | Coverage Section |
|----------------------|-----------------|
| Exception hierarchy defined | Section 3 |
| Severity model defined | Section 4 |
| Exception mappings defined | Section 5 |
| Propagation rules documented | Section 7 |
| Error response contract defined | Section 8 |
| Middleware processing defined | Section 9 |
| Validation rules defined | Section 10 |

---

# 12. Future Deliverables

The architecture documentation shall include:

- Exception Hierarchy Diagram
- Exception Mapping Diagram
- Error Code Catalog Specification
- Error Pipeline Sequence Diagram
- Middleware Processing Flow Diagram
- Cross-Layer Error Propagation Diagram
- Error Response Contract Diagram
- Telemetry and Observability Specification
