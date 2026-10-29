# ADR-009: Diagnostic Information Consumer Model

This is the one that came from your stakeholder statement.

## Decision

Diagnostic records are designed to support multiple consumer groups.

### Auditor

**Consumes:**

- Plain Text
- Timestamp
- UserId
- CorrelationId
- ReferenceCode
- Severity

### Developer

**Consumes:**

- Plain Text
- Metadata
- Source
- ExceptionType
- StackTrace

### DevOps

**Consumes:**

- Plain Text
- Provider Health
- Infrastructure Failures
- Failover Events

### PMO

**Consumes:**

- Plain Text
- Error Trends
- Severity Distribution
- Error Frequency
- Module Analysis

  **Consequence**

Diagnostic schema must support all consumer groups without requiring duplicate logging mechanisms.
