# FS-CORRELATION-ID-STRATEGY

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

This specification defines the Correlation ID Strategy used throughout the platform.

The strategy provides end-to-end traceability across:

- HTTP Requests
- Application Services
- Domain Services
- Infrastructure Components
- External Integrations
- Logging Pipelines
- Telemetry Systems
- Error Responses

The Correlation ID enables operational teams to reconstruct the complete lifecycle of a request.

---

# 2. Scope

This specification applies to:

- APIs
- Middleware
- Background Services
- Scheduled Jobs
- Integration Services
- Telemetry Pipelines
- Logging Infrastructure

All platform components shall participate in correlation propagation.

---

# 3. Architectural Goals

The Correlation ID framework shall:

- Support end-to-end request tracing
- Improve diagnostics
- Simplify troubleshooting
- Support auditing
- Support observability
- Support incident investigation
- Enable cross-system traceability

---

# 4. Core Concepts

## Correlation ID

A Correlation ID uniquely identifies a logical request as it moves through the platform.

The identifier remains constant throughout the request lifecycle.

Example:

```text
7c84d060-2d95-4c38-b6d6-1fdc4bde9592
```

---

## Request Chain

A request chain consists of all operations performed while servicing a request.

Example:

```text
HTTP Request
      │
      ▼
API Layer
      │
      ▼
Application Service
      │
      ▼
Domain Service
      │
      ▼
Repository
      │
      ▼
Database
```

All operations within the chain share the same Correlation ID.

---

# 5. Identifier Standard

## Format

The Correlation ID shall be a UUID Version 4.

Example:

```text
7c84d060-2d95-4c38-b6d6-1fdc4bde9592
```

---

## Rules

### Rule 1

The identifier must be globally unique.

### Rule 2

The identifier shall not contain business information.

### Rule 3

The identifier shall not contain user information.

### Rule 4

The identifier shall remain immutable throughout request execution.

---

# 6. Generation Strategy

## Incoming Request Contains Correlation ID

If a valid Correlation ID is supplied by an upstream system:

```text
Use Existing Correlation ID
```

---

## Incoming Request Does Not Contain Correlation ID

The middleware shall generate a new UUID v4.

```text
Generate New Correlation ID
```

---

## Invalid Correlation ID

If an invalid value is supplied:

```text
Generate New Correlation ID
```

The invalid identifier shall not be propagated.

---

# 7. Request Lifecycle

## Inbound Request

```text
Request Received
        │
        ▼
Inspect Header
        │
        ├── Exists
        │      │
        │      ▼
        │  Validate
        │
        └── Missing
               │
               ▼
           Generate
```

---

## Application Processing

The Correlation ID shall be stored within request context.

Examples:

- Middleware Context
- Request Context
- Service Context
- Execution Context

---

## Response Processing

The same Correlation ID shall be included in:

- Response Headers
- Error Responses
- Telemetry Events
- Log Entries

---

# 8. Transport Headers

## Inbound Header

```http
X-Correlation-Id
```

Example:

```http
X-Correlation-Id: 7c84d060-2d95-4c38-b6d6-1fdc4bde9592
```

---

## Outbound Header

```http
X-Correlation-Id
```

Example:

```http
X-Correlation-Id: 7c84d060-2d95-4c38-b6d6-1fdc4bde9592
```

---

# 9. Error Handling Integration

The Error Handling subsystem shall include the Correlation ID in all error responses.

Example:

```json
{
  "error": {
    "code": "VAL-001",
    "message": "Required field missing",
    "correlationId": "7c84d060-2d95-4c38-b6d6-1fdc4bde9592"
  }
}
```

The Correlation ID enables users and support personnel to reference specific failures.

---

# 10. Logging Integration

All error logs shall include the Correlation ID.

Example:

```json
{
  "level": "ERROR",
  "errorCode": "INT-001",
  "correlationId": "7c84d060-2d95-4c38-b6d6-1fdc4bde9592"
}
```

The Logging Strategy specification defines the complete logging schema.

---

# 11. Telemetry Integration

All telemetry events shall include the Correlation ID.

Examples:

- Request Telemetry
- Exception Telemetry
- Dependency Telemetry
- Audit Events

The identifier enables event correlation across monitoring platforms.

---

# 12. External Integration Requirements

The Correlation ID shall be propagated to downstream services whenever supported.

Examples:

```text
API Gateway
   │
   ▼
Fitness API
   │
   ▼
Notification API
   │
   ▼
Email Provider
```

The same Correlation ID shall be maintained whenever possible.

---

# 13. Background Processing Requirements

Background tasks shall generate a Correlation ID if one is not available.

Examples:

- Scheduled Jobs
- Queue Consumers
- Event Handlers
- Batch Processes

---

# 14. Security Requirements

Correlation IDs shall not:

- Contain usernames
- Contain email addresses
- Contain account identifiers
- Contain credentials
- Contain protected information

Correlation IDs are diagnostic identifiers only.

---

# 15. Validation Rules

The implementation shall fail validation when:

- Correlation IDs are missing from error responses
- Correlation IDs are missing from logs
- Invalid UUID formats are accepted
- Correlation IDs are modified during execution
- Correlation IDs are not propagated to supported downstream services

---

# 16. Compliance Requirements

Before release:

- Correlation generation shall be validated.
- Correlation propagation shall be validated.
- Error responses shall include correlation identifiers.
- Telemetry events shall include correlation identifiers.
- Structured logs shall include correlation identifiers.

---

# 17. Sequence Flow

```text
Request Received
        │
        ▼
Generate/Validate Correlation ID
        │
        ▼
Store In Request Context
        │
        ▼
Application Processing
        │
        ├──► Logging
        │
        ├──► Telemetry
        │
        ├──► Integration Calls
        │
        └──► Error Handling
                   │
                   ▼
             Response Sent
```

---

# 18. Cross References

| Dependency | Purpose |
|------------|------------|
| Error Handling Architecture Overview | Subsystem architecture |
| Error Mapping Table Specification | Error response generation |
| Error Pipeline Logging Strategy | Log correlation |
| Error Code Registry Specification | Error identification |

---

# 19. Related Documents

## Parent Architecture

- Error Handling Architecture Overview

## Supporting Specifications

- Error Mapping Table Specification
- Error Pipeline Logging Strategy
- Error Code Registry Specification

## Governance

- Error Handling Standard
- Observability Standard
- Security Standard
