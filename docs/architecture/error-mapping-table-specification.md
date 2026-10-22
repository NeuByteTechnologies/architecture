# FS-ERROR-MAPPING-TABLE-SPECIFICATION

## 1. Document Overview

### Purpose

This specification defines the Exception Mapping Table for the Fitness Application Error Handling Framework. It establishes standard exception classification, mapping behavior, propagation rules, severity assignment, and cross-layer handling semantics.

### Scope

The specification applies to all application layers including:

- Presentation Layer (UI / API)
- Middleware Layer
- Application Services Layer
- Domain Layer
- Data Access Layer
- External Integration Layer
- Related Artifacts
- Business Requirements
  - BR-ERROR-01
   - BR-ERROR-PIPELINE-02
- Use Cases
  - UC-ERROR-HANDLING-01
  - UC-VALIDATION-02
 - Functional Specifications
  - FS-ERROR-HIERARCHY
  - FS-MIDDLEWARE-FLOW
- Data Specification Fields
  - error.code
  - error.type
  - error.severity
- Test Cases
  - TC-EXCEPTION-MAP-01
  - TC-EXCEPTION-MAP-NEG-01

## 2. Error Handling Objectives

The Exception Mapping Table shall:

 1. Standardize exception classification.
 2. Ensure consistent error responses across layers.
 3.  Prevent leakage of internal implementation details.
 4. Support monitoring, diagnostics, and auditing.
 5. Enable deterministic middleware processing.
  6. Provide consistent user-facing error messages.


## 3. Exception Hierarchy

### 3.1 Logical Hierarchy
---
 BaseApplicationException

 -  ValidationException
 1.  RequiredFieldException
 2.  FormatValidationException
 3.  BusinessRuleValidationException

 -  AuthorizationException
 1.  AuthenticationFailedException
  2. PermissionDeniedException

 -  BusinessException
  1. WorkoutRuleException
  2. NutritionRuleException
  3.  MembershipRuleException

 -  IntegrationException
  1.  ExternalApiException
  2. TimeoutException
  3. ThirdPartyServiceException

 -  DataAccessException
  1.  DatabaseException
  2.  ConcurrencyException
  3.  DataIntegrityException

 -  SystemException
  1. ConfigurationException
  2. InfrastructureException
  3.  UnexpectedException
---

## 4. Severity Classification

| Severity | Description | Operational Impact |
|----------|--------------|----------------------|
| INFO  | Informational processing event | No interruption |
| LOW | Minor recoverable issue | Limited impact |
| MEDIUM | Functional degradation | User action needed |
| HIGH | Business operation failure | Process halted |
| CRITICAL | System-wide failure | Immediate attention required |

## 5. Exception Mapping Table

| Exception Type | Error Code | Severity | HTTP Status | Client Visible | Retry Allowed |
|-----------------|-------------|----------|---------------|---------------|----------------|
| RequiredFieldException | VAL-001 | LOW | 400  | Yes | No |
| FormatValidationException | VAL-002 | LOW | 400 | Yes | No |
| BusinessRuleValidationException | VAL-003 |MEDIUM | 422 | Yes | No |
| AuthenticationFailedException | AUTH-001 | MEDIUM | 401 |Yes | No |
| PermissionDeniedException  | AUTH-002 | MEDIUM |403 | Yes | No |
| WorkoutRuleException | BUS-001 | MEDIUM | 422 | Yes | No |
| NutritionRuleException | BUS-002 | MEDIUM | 422 | Yes | No |
| MembershipRuleException | BUS-003 | MEDIUM | 422 | Yes | No |
| ExternalApiException | INT-001 | HIGH | 502 | No |Yes |
| TimeoutException | INT-002  | HIGH |504 | No | Yes |
| ThirdPartyServiceException | INT-003  | HIGH | 503 | No | Yes |
| DatabaseException | DATA-001 | HIGH  | 500  | No |  Conditional |
| ConcurrencyException | DATA-002 | HIGH | 409 | No | Yes |
| DataIntegrityException | DATA-003 | HIGH | 500 | No | No |
| ConfigurationException | SYS-001 |  CRITICAL | 500 | No | No |
| InfrastructureException | SYS-002 | CRITICAL | 500 | No | Conditional |
| UnexpectedException | SYS-999 | CRITICAL | 500 | No | No

## 6. Mapping Rules

### Rule 1: Explicit Mapping

Every custom exception class shall be mapped to exactly one error.code.

### Rule 2: Severity Assignment

Every mapped exception shall include a defined error.severity.

### Rule 3: Fallback Mapping

Unrecognized exceptions shall be mapped to:

**1     Exception Type: UnexpectedException**

**2     Error Code: SYS-999**

**3     Severity: CRITICAL**

**4     HTTP Status: 500**

### Rule 4: Client Response Protection

Internal stack traces shall never be exposed outside the server boundary.

### Rule 5: Logging Requirement

All HIGH and CRITICAL exceptions shall be logged.

### Rule 6: Observability Requirement

All exception mappings shall generate telemetry events.



## 7. Cross-Layer Propagation Rules

### 7.1 Propagation Principle

Exceptions originate in lower layers and propagate upward through the middleware pipeline.

1     Data Layer

2          │

3          ▼

4     Domain Layer

5          │

6          ▼

7     Application Service Layer

8          │

9          ▼

10     Middleware Layer

11          │

12          ▼

13     API/UI Response

### 7.2 Translation Rules

| Source Layer| Original Exception | Translated To|
|---------------|---------------------|---------------|
| Database | SQL Exception | DatabaseException |
| Third Party API | Network Failure | ExternalApiException |
| Domain Service | Rule Violation | BusinessRuleValidationException |
| Authentication Service | Invalid Credential | AuthenticationFailedException |

### 7.3 Termination Point

The Middleware Layer shall be the final exception handling boundary responsible for:

- Classification
- Mapping
- Logging
- Telemetry generation
- Client response creation

## 8. Error Response Contract

### Standard Structure

1     {

2       "error": {

3         "code": "VAL-001",

4         "type": "ValidationException",

5         "severity": "LOW",

6         "message": "Validation failed",

7         "correlationId": "GUID"

8       }

9     }

### Field Definitions

| Field | Description
| error.code | Unique mapped error identifier|
| error.type | Exception classification |
| error.severity | Operational severity level |
| message | User-safe response message |
| correlationId | Traceability identifier |

## 9. Middleware Processing Semantics

1     Exception Raised

2            │

3            ▼

4     Exception Interceptor

5            │

6            ▼

7     Classify Exception

8            │

9            ▼

10     Lookup Mapping Table

11            │

12            ▼

13     Assign Code & Severity

14            │

15            ├── Log Event

16            ├── Create Telemetry Event

17            └── Build Error Response

18                    │

19                    ▼

20     Return Response

## 10. Validation Rules

The implementation shall reject deployment if:

1. An exception class exists without a mapping.
2. Duplicate error codes exist.
3. Invalid severity values are assigned.
4. Undefined propagation paths are detected.
5. Middleware mapping configuration is incomplete.

## 11. Acceptance Criteria Traceability

| Acceptance Criterion |  Coverage Section |
| Mapping table fully defined | Section 5 |
| All exception classes mapped | Sections 3 and 5 |
| Propagation rules documented | Section 7 |
| Cross-references validated | Section 1 |
| Terminology consistent with Visual Grammar | Entire document |

## 12. Future Diagram Deliverables

The Architecture documentation shall include:

1. Exception Hierarchy Diagram
2. Error Pipeline Sequence Diagram
3. Middleware Exception Processing Flow Diagram
4. Cross-Layer Error Propagation Diagram
5. Error Response Contract Diagram
