# ADR-006: Centralized Exception Handling Middleware
## Status

Accepted

## Context

The application requires a consistent mechanism for handling exceptions across all application layers.

Without centralized handling:

- Controllers and endpoints contain repetitive try-catch blocks.
- HTTP status code mappings become inconsistent.
- Error response structures vary by implementation.
- Logging behavior becomes fragmented.
- Correlation identifiers are not consistently returned.

The library already provides:

- ErrorCodes
- ErrorCatalog
- BaseApplicationException hierarchy
- TemplateFormatter
- ErrorEnvelope
- LoggerEnvelope

A mechanism was needed to coordinate these components.

## Decision

Implement a centralized ErrorHandlerMiddleware component responsible for:

1. Capturing all unhandled exceptions.
2. Normalizing exceptions into BaseApplicationException.
3. Resolving metadata from ErrorCatalog.
4. Creating a LoggerEnvelope.
5. Creating a client-facing ErrorEnvelope.
6. Setting the appropriate HTTP status code.
7. Returning a JSON response.

## Design

### Exception Flow

**Plain Text**
Application Code
↓
Exception Thrown
↓
ErrorHandlerMiddleware
↓
NormalizeException()
↓
ErrorCatalog Lookup
↓
BuildLogRecord()
↓
BuildErrorEnvelope()
↓
JSON Response

### Normalization Strategy

Application exceptions are passed through unchanged.

Unknown framework and runtime exceptions are wrapped as:

C#
---
UnexpectedException
---

allowing all downstream processing to operate against:

C#
---
BaseApplicationException
 ---

Logging Strategy

Middleware creates a LoggerEnvelope containing:

CorrelationId
Timestamp
ErrorCode
ReferenceCode
Severity
ExceptionType
Message
Source
UserId
Metadata

The middleware performs construction internally rather than introducing additional logging factories.

Response Strategy

Responses are returned using:

C#
---
ErrorEnvelope
---

containing:

C#
---
ReferenceCode
Message
Severity
TraceId
---

Only client-safe information is exposed.

## Alternatives Considered

### Per Controller Try/Catch

Rejected.

Reasons:

Repetitive code.
Inconsistent status code handling.
Difficult to maintain.

### Exception Filters

Rejected.

Reasons:

Middleware provides broader coverage.
Works consistently for all requests.
Easier integration with logging.

### Dedicated LogRecordFactory

Rejected.

Reasons:

Middleware is currently the sole consumer.
Introduced additional abstraction without demonstrated need.
Construction logic remains small and cohesive.

## Consequences

### Positive
Consistent HTTP responses.
Consistent logging behavior.
Single exception processing pipeline.
Reduced duplication.
Easier testing.

### Negative

Middleware must understand ErrorCatalog.
Additional helper methods exist within middleware.

These were considered acceptable tradeoffs because ErrorCatalog is the established source of truth for error metadata.

### Validation

Behavior tests verify:

Next delegate execution.
Validation exception returns HTTP 400.
Unexpected exception returns HTTP 500.
JSON response generation.
Trace identifier generation.
Support reference code generation.
