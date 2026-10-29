# ADR-007: Centralized Diagnostic Logging Strategy
## Status

Accepted

## Context

**The Fitness Application requires:**

- Auditable error records
- Developer diagnostics
- Operational monitoring
- PMO reporting

**Several approaches were considered:**

- Plain Text
- Distributed ILogger<T> logging
- Multi-provider fan-out logging
- Centralized exception-driven logging

## Decision

Adopt centralized exception-driven logging.

Unhandled exceptions are captured by ErrorHandlerMiddleware and transformed into structured diagnostic records.

Application components are not expected to perform extensive direct logging.

**Diagnostic information is captured from:**

- Plain Text
- Exception Type
- Error Code
- Reference Code
- Metadata
- Correlation Information

and persisted through centralized logging infrastructure.

## Consequences

**Benefits:**

Plain Text
Reduced logging duplication
Consistent diagnostic records
Simplified maintenance
Improved auditability

**Tradeoffs:**

Plain Text
Less ad hoc developer logging
Greater dependency on exception quality
More responsibility in middleware layer
