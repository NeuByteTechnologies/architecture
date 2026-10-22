# FS-ERROR-MAPPING-TABLE-SPECIFICATION

| Field | Value |
|---------|---------|
| Version | 1.0.0 |
| Status | Draft |
| Classification | Internal Architecture |
| Domain | Error Handling |
| Owner | Architecture |
| Parent Document | Error Handling Architecture Overview |

---

# 1. Purpose

This specification defines the Exception Mapping Framework used by the Error Handling subsystem.

The mapping framework provides deterministic translation between:

- Thrown exceptions
- Error codes
- HTTP status codes
- Severity levels
- Client response behaviors

The mapping process ensures all errors are handled consistently regardless of originating layer.

---

# 2. Scope

This specification applies to:

- Domain exceptions
- Application exceptions
- Infrastructure exceptions
- Integration exceptions
- Validation exceptions
- Authentication exceptions
- Authorization exceptions

The specification governs all exceptions that may pass through the Error Handler Middleware.

---

# 3. Architectural Goals

The mapping process shall:

- Produce deterministic error responses
- Eliminate ambiguous exception handling
- Standardize severity assignment
- Support observability
- Support troubleshooting
- Prevent implementation detail leakage
- Enable automated documentation generation

---

# 4. Mapping Principles

## Principle 1 - Deterministic Mapping

Every exception shall map to exactly one error code.

An exception shall never map to multiple error codes.

---

## Principle 2 - Single Responsibility

Exception classes define business meaning.

The mapping table defines operational behavior.

---

## Principle 3 - Controlled Disclosure

Internal exception details shall never be exposed outside the application boundary.

---

## Principle 4 - Observability

All mapped exceptions shall be eligible for logging, telemetry, and diagnostics.

---

# 5. Severity Definitions

| Severity | Description | Operational Impact |
|------------|------------|------------|
| INFO | Informational processing event | No interruption |
| LOW | Minor recoverable issue | Limited impact |
| MEDIUM | Degraded business operation | User action required |
| HIGH | Service functionality impaired | Operation failed |
| CRITICAL | System-level failure | Immediate intervention required |

---

# 6. Standard Mapping Table

| Exception Type | Error Code | HTTP Status | Severity | Client Visible | Retry Allowed |
|----------------|------------|-------------|----------|---------------|---------------|
| RequiredFieldException | VAL-001 | 400 | LOW | Yes | No |
| FormatValidationException | VAL-002 | 400 | LOW | Yes | No |
| BusinessRuleValidationException | VAL-003 | 422 | MEDIUM | Yes | No |
| AuthenticationFailedException | AUTH-001 | 401 | MEDIUM | Yes | No |
| PermissionDeniedException | AUTH-002 | 403 | MEDIUM | Yes | No |
| WorkoutRuleException | BUS-001 | 422 | MEDIUM | Yes | No |
| NutritionRuleException | BUS-002 | 422 | MEDIUM | Yes | No |
| MembershipRuleException | BUS-003 | 422 | MEDIUM | Yes | No |
| ExternalApiException | INT-001 | 502 | HIGH | No | Yes |
| TimeoutException | INT-002 | 504 | HIGH | No | Yes |
| ThirdPartyServiceException | INT-003 | 503 | HIGH | Yes | Yes |
| DatabaseException | DATA-001 | 500 | HIGH | No | Conditional |
| ConcurrencyException | DATA-002 | 409 | HIGH | Yes | Yes |
| DataIntegrityException | DATA-003 | 500 | HIGH | No | No |
| ConfigurationException | SYS-001 | 500 | CRITICAL | No | No |
| InfrastructureException | SYS-002 | 500 | CRITICAL | No | Conditional |
| UnexpectedException | SYS-999 | 500 | CRITICAL | No | No |

---

# 7. Default Mapping Rule

If an exception cannot be resolved through explicit mappings, the middleware shall apply the fallback mapping.

| Property | Value |
|------------|------------|
| Exception Type | UnexpectedException |
| Error Code | SYS-999 |
| HTTP Status | 500 |
| Severity | CRITICAL |
| Client Visible | No |
| Retry Allowed | No |

---

# 8. Mapping Resolution Process

```text
Exception Raised
        │
        ▼
Identify Exception Type
        │
        ▼
Lookup Mapping Table
        │
        ├── Match Found
        │        │
        │        ▼
        │   Assign Mapping
        │
        └── No Match
                 │
                 ▼
          Apply Fallback Mapping
                 │
                 ▼
          Return Result
```

---

# 9. Cross-Layer Translation Rules

The originating layer may translate platform-specific exceptions into application-specific exceptions prior to mapping.

| Source Layer | Original Exception | Translated Exception |
|--------------|-------------------|----------------------|
| Database Layer | SqlException | DatabaseException |
| Database Layer | Concurrency Failure | ConcurrencyException |
| Integration Layer | Network Failure | ExternalApiException |
| Integration Layer | Timeout | TimeoutException |
| Domain Layer | Rule Violation | BusinessRuleValidationException |
| Security Layer | Authentication Failure | AuthenticationFailedException |
| Security Layer | Authorization Failure | PermissionDeniedException |

The middleware should only receive application-defined exceptions whenever possible.

---

# 10. Client Visibility Rules

## Visible Errors

Visible errors may expose:

- Error code
- User-safe message
- Correlation identifier

Examples:

- Validation failures
- Authentication failures
- Authorization failures
- Business rule violations

---

## Non-Visible Errors

Non-visible errors shall not expose:

- Stack traces
- Database details
- Infrastructure information
- External dependency information

Examples:

- Database failures
- Configuration failures
- Infrastructure failures
- Unknown exceptions

---

# 11. Retry Classification

## Retry Allowed

Operations may be retried automatically.

Examples:

- Network failures
- External service timeouts
- Concurrency conflicts

---

## Conditional Retry

Retry behavior depends on runtime circumstances.

Examples:

- Infrastructure failures
- Database availability failures

---

## Retry Prohibited

Retries shall not be attempted.

Examples:

- Validation failures
- Authentication failures
- Business rule violations

---

# 12. Validation Rules

The implementation shall fail validation when:

- An exception lacks a mapping
- Duplicate exception mappings exist
- Duplicate error codes are assigned
- Undefined error codes are referenced
- Invalid severity values are assigned
- Invalid HTTP status values are assigned

---

# 13. Compliance Requirements

The following conditions must be true before release:

- Every exception has a mapping
- Every mapping references a registered error code
- All mappings have assigned severities
- All mappings define visibility behavior
- All mappings define retry behavior

---

# 14. Related Documents

## Parent Architecture

- Error Handling Architecture Overview

## Supporting Specifications

- Error Code Registry Specification
- Domain Exception Subclass Specification
- Error Pipeline Logging Strategy
- Correlation ID Strategy

## Governance

- Error Handling Standard
