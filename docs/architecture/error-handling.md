# NeuByte Fitness App
## Error Handling Module — Architecture Reference

| Field | Value |
|---|---|
| **Version** | 1.0.0 |
| **Status** | Draft |
| **Date** | October 14, 2026 |
| **Author** | Gordon |
| **Module** | Error Handling |
| **Classification** | Confidential — Internal Architecture |

---

## Table of Contents

1. [Document Purpose & Scope](#1-document-purpose--scope)
2. [Architectural Role & Position](#2-architectural-role--position)
3. [Responsibilities](#3-responsibilities)
4. [Non-Responsibilities](#4-non-responsibilities)
5. [Error Envelope Specification](#5-error-envelope-specification)
6. [Error Code Registry](#6-error-code-registry)
7. [Exception Hierarchy](#7-exception-hierarchy)
8. [Middleware Design](#8-middleware-design)
9. [Logging Strategy](#9-logging-strategy)
10. [DevOps Notification Strategy](#10-devops-notification-strategy)
11. [Validation Integration](#11-validation-integration)
12. [TDD Testing Strategy](#12-tdd-testing-strategy)
13. [Appendix — Architectural Decision Records (ADRs)](#13-appendix--architectural-decision-records-adrs)

---

## 1. Document Purpose & Scope

This document defines the complete architecture of the **Error Handling Module** for the NeuByte Fitness App prior to any implementation. It serves as the **formal contract** between all other application modules and the error system.

### 1.1 Purpose

- Establish the canonical error shape returned by every API endpoint.
- Define the full error code registry and exception class hierarchy.
- Specify middleware design, logging strategy, and DevOps alerting behaviour.
- Provide the TDD test plan that must be executed before implementation begins.

### 1.2 Scope

This document covers:
- Error envelope schema and field specification
- Error code registry across all NeuByte domains
- TypeScript exception class hierarchy
- Global error-handling middleware
- Structured logging and log redaction rules
- DevOps alert routing and SLA definitions
- Zod validation integration
- Complete Test-Driven Development (TDD) plan

### 1.3 Pre-Coding Gate

> **⚠️ Architecture Gate:** No implementation of any error-handling code begins until this document has been reviewed and approved. All P0 tests from Section 12 must be written before any source code is authored.

---

## 2. Architectural Role & Position

The Error Handling Module is a **cross-cutting concern**. Every other NeuByte module — Auth, Workout, Nutrition, Notifications, Analytics, and User Profile — depends on it.

### 2.1 Layer Position

```
┌─────────────────────────────────────────────────────┐
│                 API / Transport Layer                │
│           (HTTP routes, controllers, DTOs)           │
└────────────────────────┬────────────────────────────┘
                         │ throws AppError subclasses
┌────────────────────────▼────────────────────────────┐
│          Global Error Handler Middleware             │
│    (serialises, redacts, logs, alerts, responds)     │
└────────────────────────┬────────────────────────────┘
                         │ uses
┌────────────────────────▼────────────────────────────┐
│              Exception Hierarchy                     │
│   AppError → ValidationError, NotFoundError, etc.   │
└────────────────────────┬────────────────────────────┘
                         │ feeds
┌────────────────────────▼────────────────────────────┐
│           Logging + Alerting Infrastructure          │
│        (structured log sink, PagerDuty, Slack)       │
└─────────────────────────────────────────────────────┘
```

### 2.2 Initialisation Order

The Error Handling Module is **initialised before any other application module boots**. This ensures that even startup errors are captured in the canonical format.

### 2.3 Single Source of Truth

The Error Handling Module is the sole owner of:
- Error envelope shape
- Error code string values
- Severity mappings
- Alert trigger rules

No other module may define its own error shapes or codes outside this module.

---

## 3. Responsibilities

The Error Handling Module is responsible for the following:

1. **Envelope Ownership** — Define and maintain the canonical error envelope schema returned by every API error response.
2. **Error Code Registry** — Own and version the complete registry of error code strings across all domains.
3. **Exception Class Hierarchy** — Provide the `AppError` base class and all typed subclasses used throughout the application.
4. **Global Middleware** — Provide the Express/Fastify global error-handling middleware that intercepts all unhandled errors.
5. **Error Serialisation** — Convert any thrown exception (including third-party errors) into a valid error envelope.
6. **Structured Logging** — Log all errors with appropriate severity levels and structured fields.
7. **DevOps Alerting** — Trigger PagerDuty and Slack notifications for critical and security-relevant errors.
8. **Validation Normalisation** — Integrate with Zod to transform `ZodError` instances into `ValidationError` envelopes.
9. **Utility Functions** — Provide helpers: `wrapAsync`, `throwIf`, `assertDefined`, `mapToAppError`, `redactSensitiveFields`.
10. **Trace ID Propagation** — Attach and propagate UUID v4 correlation IDs for distributed tracing across all error envelopes and log entries.
11. **Sensitive Field Redaction** — Ensure PII and credential fields are stripped from error details before logging or responding.

---

## 4. Non-Responsibilities

The Error Handling Module explicitly does **not** own or perform the following:

1. **Business Logic** — Domain-specific rules belong in domain modules (Workout, Nutrition, etc.).
2. **User-Facing Notifications** — Push notifications or email alerts to end users are owned by the Notifications Module.
3. **Retry Logic** — Retry policies belong to the caller or a dedicated resilience/circuit-breaker module.
4. **Persistent Error Storage** — Long-term error storage in a database is the responsibility of the logging infrastructure, not this module.
5. **HTTP Route Definitions** — This module provides only middleware, not route handlers.
6. **Authentication** — Auth errors are surfaced by this module but originated and owned by the Auth Module.

---

## 5. Error Envelope Specification

Every API error response — regardless of domain, status code, or error type — returns the identical JSON envelope shape.

### 5.1 Schema Definition

```json
{
  "success": false,
  "error": {
    "code": "AUTH_TOKEN_EXPIRED",
    "message": "Your session has expired. Please log in again.",
    "statusCode": 401,
    "traceId": "c1a2b3d4-e5f6-7890-abcd-ef1234567890",
    "timestamp": "2026-10-14T13:02:00.000Z",
    "details": [],
    "meta": {}
  }
}
```

### 5.2 Field Specification

| Field | Type | Required | Description | Example |
|---|---|---|---|---|
| `success` | `boolean` | ✅ | Always `false` for error responses | `false` |
| `error.code` | `string` | ✅ | Machine-readable error code from the registry | `"AUTH_TOKEN_EXPIRED"` |
| `error.message` | `string` | ✅ | Human-readable message; safe to display in UI | `"Your session has expired."` |
| `error.statusCode` | `number` | ✅ | HTTP status code mirroring the response status | `401` |
| `error.traceId` | `string` | ✅ | UUID v4 correlation ID for distributed tracing | `"c1a2b3d4-..."` |
| `error.timestamp` | `string` | ✅ | ISO 8601 UTC timestamp of when the error occurred | `"2026-10-14T13:02:00.000Z"` |
| `error.details` | `ErrorDetail[]` | ✅ | Array of sub-errors (e.g. field-level validation failures). Empty array if none | `[]` |
| `error.meta` | `object` | ✅ | Optional key-value context bag. Never contains PII. Empty object if none | `{}` |

### 5.3 Validation Error Details Shape

When `details` is populated (typically for 422 Validation errors), each entry follows this shape:

```json
"details": [
  {
    "field": "email",
    "code": "VALIDATION_INVALID_FORMAT",
    "message": "Must be a valid email address."
  },
  {
    "field": "password",
    "code": "VALIDATION_TOO_SHORT",
    "message": "Password must be at least 8 characters."
  }
]
```

#### ErrorDetail Field Specification

| Field | Type | Required | Description |
|---|---|---|---|
| `field` | `string` | ✅ | Dot-path to the invalid field (e.g. `"address.postcode"`) |
| `code` | `string` | ✅ | Error code specific to this field violation |
| `message` | `string` | ✅ | Human-readable description of the violation |

### 5.4 Envelope Rules

- The top-level `success` field is **always** `false` for error responses and `true` for success responses.
- `error.message` must **never** expose internal stack traces, SQL queries, or system paths.
- `error.meta` must **never** contain passwords, tokens, PII, or any sensitive field.
- `error.traceId` must match the `X-Trace-Id` response header sent alongside the error.

---

## 6. Error Code Registry

### 6.1 Naming Convention

All error codes follow the pattern:

```
DOMAIN_CATEGORY_SPECIFICS
```

- **All uppercase**, underscore-separated.
- Prefix = owning domain (e.g. `AUTH`, `USER`, `WORKOUT`).
- Codes are string constants defined in `src/errors/registry/errorCodes.ts`.
- No code may be defined outside that file.

**Valid domains:** `AUTH` · `USER` · `WORKOUT` · `NUTRITION` · `NOTIFICATION` · `ANALYTICS` · `SYSTEM` · `VALIDATION` · `NETWORK`

### 6.2 Registry

#### AUTH Domain

| Error Code | HTTP Status | Severity | Description | Retriable? |
|---|---|---|---|---|
| `AUTH_TOKEN_EXPIRED` | 401 | WARN | JWT access token has expired | Yes |
| `AUTH_TOKEN_INVALID` | 401 | WARN | JWT signature is invalid or malformed | No |
| `AUTH_UNAUTHORIZED` | 403 | WARN | Authenticated user lacks required permission | No |
| `AUTH_ACCOUNT_LOCKED` | 403 | ERROR | Account locked after too many failed login attempts | No |
| `AUTH_REFRESH_TOKEN_INVALID` | 401 | WARN | Refresh token is invalid or revoked | No |

#### USER Domain

| Error Code | HTTP Status | Severity | Description | Retriable? |
|---|---|---|---|---|
| `USER_NOT_FOUND` | 404 | INFO | User record does not exist | No |
| `USER_DUPLICATE_EMAIL` | 409 | WARN | Email address is already registered | No |
| `USER_PROFILE_INCOMPLETE` | 422 | INFO | Required profile fields are missing | No |
| `USER_ACCOUNT_DELETED` | 410 | WARN | User account has been deactivated | No |

#### WORKOUT Domain

| Error Code | HTTP Status | Severity | Description | Retriable? |
|---|---|---|---|---|
| `WORKOUT_NOT_FOUND` | 404 | INFO | Workout plan does not exist | No |
| `WORKOUT_INVALID_EXERCISE` | 422 | WARN | Referenced exercise ID does not exist | No |
| `WORKOUT_SCHEDULE_CONFLICT` | 409 | WARN | Overlapping workout already scheduled | No |
| `WORKOUT_MAX_PLANS_REACHED` | 422 | WARN | User has reached maximum allowed workout plans | No |

#### NUTRITION Domain

| Error Code | HTTP Status | Severity | Description | Retriable? |
|---|---|---|---|---|
| `NUTRITION_LOG_NOT_FOUND` | 404 | INFO | Nutrition log entry does not exist | No |
| `NUTRITION_INVALID_MACRO` | 422 | WARN | Macro values exceed physiological limits | No |
| `NUTRITION_DUPLICATE_LOG` | 409 | WARN | Log entry already exists for this date/meal | No |

#### VALIDATION Domain

| Error Code | HTTP Status | Severity | Description | Retriable? |
|---|---|---|---|---|
| `VALIDATION_REQUIRED_FIELD` | 422 | INFO | A required field is missing from the request | No |
| `VALIDATION_INVALID_FORMAT` | 422 | INFO | Field format does not match the expected pattern | No |
| `VALIDATION_TOO_SHORT` | 422 | INFO | Field value is below the minimum length | No |
| `VALIDATION_TOO_LONG` | 422 | INFO | Field value exceeds the maximum length | No |
| `VALIDATION_OUT_OF_RANGE` | 422 | INFO | Numeric value is outside the acceptable range | No |
| `VALIDATION_INVALID_TYPE` | 422 | INFO | Field value is of the wrong data type | No |
| `VALIDATION_INVALID_ENUM` | 422 | INFO | Value is not a member of the allowed enum set | No |

#### SYSTEM Domain

| Error Code | HTTP Status | Severity | Description | Retriable? |
|---|---|---|---|---|
| `SYSTEM_INTERNAL_ERROR` | 500 | CRITICAL | Unhandled internal server error | Yes |
| `SYSTEM_DATABASE_UNAVAILABLE` | 503 | CRITICAL | Database connection failed or timed out | Yes |
| `SYSTEM_RATE_LIMIT_EXCEEDED` | 429 | WARN | Client has exceeded the request rate limit | Yes |
| `SYSTEM_TIMEOUT` | 504 | ERROR | Upstream service or query timed out | Yes |
| `SYSTEM_SERVICE_UNAVAILABLE` | 503 | ERROR | A downstream service dependency is unavailable | Yes |
| `SYSTEM_PAYLOAD_TOO_LARGE` | 413 | WARN | Request body exceeds the maximum allowed size | No |

---

## 7. Exception Hierarchy

All custom exceptions extend the `AppError` base class. No module may throw a raw `Error` object — it must always throw a typed `AppError` subclass.

### 7.1 Hierarchy Diagram

```
Error (native JS/TS)
└── AppError (base — owns statusCode, errorCode, details, meta, traceId, isOperational)
    ├── ValidationError           (422 · VALIDATION_*)
    ├── AuthenticationError       (401 · AUTH_*)
    │   └── TokenExpiredError     (401 · AUTH_TOKEN_EXPIRED)
    ├── AuthorizationError        (403 · AUTH_UNAUTHORIZED)
    ├── NotFoundError             (404 · *_NOT_FOUND)
    ├── ConflictError             (409 · *_DUPLICATE_* / *_CONFLICT_*)
    ├── UnprocessableError        (422 · domain-specific)
    ├── RateLimitError            (429 · SYSTEM_RATE_LIMIT_EXCEEDED)
    ├── InternalError             (500 · SYSTEM_INTERNAL_ERROR)
    └── ServiceUnavailableError   (503 · SYSTEM_DATABASE_UNAVAILABLE)
```

### 7.2 AppError Base Class Specification

```typescript
interface AppErrorParams {
  errorCode: string;
  message: string;
  statusCode: number;
  details?: ErrorDetail[];
  meta?: Record<string, unknown>;
  isOperational?: boolean;  // default: true
  cause?: unknown;          // original error for chaining
}

class AppError extends Error {
  readonly errorCode: string;
  readonly statusCode: number;
  readonly traceId: string;          // UUID v4, generated at construction
  readonly timestamp: string;        // ISO 8601 UTC, generated at construction
  readonly details: ErrorDetail[];
  readonly meta: Record<string, unknown>;
  readonly isOperational: boolean;   // true = expected; false = bug

  constructor(params: AppErrorParams) { /* ... */ }

  toEnvelope(): ErrorEnvelope {
    return {
      success: false,
      error: {
        code: this.errorCode,
        message: this.message,
        statusCode: this.statusCode,
        traceId: this.traceId,
        timestamp: this.timestamp,
        details: this.details,
        meta: this.meta,
      },
    };
  }
}
```

### 7.3 The `isOperational` Flag

The `isOperational` flag is the most important property on `AppError`:

| Value | Meaning | Log Severity | Alert Triggered? |
|---|---|---|---|
| `true` | Expected, handled error (404, 401, 422) | INFO / WARN | No |
| `false` | Unexpected bug or unhandled condition | CRITICAL | Yes — immediately |

- All `AppError` subclasses default `isOperational` to `true`.
- `InternalError` and errors wrapping unknown exceptions set `isOperational` to `false`.
- The global error handler uses this flag to decide alert routing.

---

## 8. Middleware Design

### 8.1 Middleware Chain Order

```
1. TraceId Middleware         → attaches UUID v4 traceId to req.context
2. Request Logger Middleware  → logs incoming request with traceId
3. Route Handlers             → business logic; throws AppError subclasses
4. wrapAsync()                → catches async rejections; forwards to next(err)
5. Global Error Handler       ← THIS MODULE
6. JSON Error Response        → sent to client
```

The Global Error Handler is always the **last** middleware registered.

### 8.2 Global Error Handler — Logic

```typescript
function globalErrorHandler(
  err: unknown,
  req: Request,
  res: Response,
  next: NextFunction
): void {

  let appError: AppError;

  // Step 1: Identify error type and normalise to AppError
  if (err instanceof AppError) {
    appError = err;
  } else if (err instanceof ZodError) {
    appError = mapZodError(err);                  // → ValidationError (422)
  } else if (isKnownThirdPartyError(err)) {
    appError = mapThirdPartyError(err);           // → ServiceUnavailableError, etc.
  } else {
    appError = new InternalError({                // → isOperational = false
      message: "An unexpected error occurred.",
      cause: err,
    });
  }

  // Step 2: Attach traceId from request context if not already set
  const traceId = req.context?.traceId ?? appError.traceId;

  // Step 3: Redact sensitive fields from details
  const safeError = redactSensitiveFields(appError);

  // Step 4: Log structured error
  logger.log(resolveSeverity(safeError), buildLogEntry(safeError, req));

  // Step 5: Trigger DevOps alert if warranted
  if (!safeError.isOperational || safeError.statusCode >= 500) {
    alertingService.trigger(safeError, req);
  }

  // Step 6: Send envelope response
  res.status(safeError.statusCode).json(safeError.toEnvelope());
}
```

### 8.3 Utility Functions

| Function | Signature | Purpose |
|---|---|---|
| `wrapAsync` | `(fn: AsyncHandler) => Handler` | Wraps async route handlers to forward rejections to error middleware |
| `throwIf` | `(condition: boolean, err: AppError) => void` | Throws the error if condition is `true`; no-op if `false` |
| `assertDefined` | `<T>(val: T \| undefined, err: AppError) => T` | Asserts a value is defined; throws if it is `undefined` or `null` |
| `mapToAppError` | `(err: unknown) => AppError` | Converts any unknown error into a typed `AppError` instance |
| `redactSensitiveFields` | `(error: AppError) => AppError` | Returns a new error instance with PII/credential fields stripped from `details` and `meta` |

#### `wrapAsync` Usage Example

```typescript
// Without wrapAsync — async errors are silently swallowed:
router.get("/workout/:id", async (req, res) => { /* ... */ });

// With wrapAsync — async rejections are forwarded to globalErrorHandler:
router.get("/workout/:id", wrapAsync(async (req, res) => {
  const workout = await workoutService.findById(req.params.id);
  throwIf(!workout, new NotFoundError({ errorCode: "WORKOUT_NOT_FOUND" }));
  res.json({ success: true, data: workout });
}));
```

---

## 9. Logging Strategy

### 9.1 Severity Levels

| Level | Condition | Example Errors |
|---|---|---|
| `DEBUG` | Development environment only; verbose diagnostic output | Request body parsing, middleware trace |
| `INFO` | Expected operational errors; normal application flow | `USER_NOT_FOUND`, `VALIDATION_REQUIRED_FIELD` |
| `WARN` | Auth failures, rate limits, business rule conflicts | `AUTH_TOKEN_EXPIRED`, `SYSTEM_RATE_LIMIT_EXCEEDED`, `USER_DUPLICATE_EMAIL` |
| `ERROR` | Recoverable system errors; degraded functionality | `SYSTEM_TIMEOUT`, `SYSTEM_SERVICE_UNAVAILABLE` |
| `CRITICAL` | Non-operational errors; unhandled exceptions; data integrity risk | `SYSTEM_INTERNAL_ERROR`, any `isOperational === false` |

### 9.2 Severity Resolution Rules

```typescript
function resolveSeverity(err: AppError): LogLevel {
  if (!err.isOperational) return "CRITICAL";
  if (err.statusCode >= 500) return "ERROR";
  if (err.statusCode >= 400 && err.statusCode < 500) {
    if (err instanceof AuthenticationError) return "WARN";
    if (err instanceof RateLimitError) return "WARN";
    return "INFO";
  }
  return "INFO";
}
```

### 9.3 Structured Log Schema

Every log entry is a JSON object with the following fields:

```json
{
  "level": "WARN",
  "timestamp": "2026-10-14T13:02:00.000Z",
  "traceId": "c1a2b3d4-e5f6-7890-abcd-ef1234567890",
  "errorCode": "AUTH_TOKEN_EXPIRED",
  "message": "Your session has expired. Please log in again.",
  "statusCode": 401,
  "userId": "usr_abc123",
  "method": "GET",
  "path": "/api/v1/workout/42",
  "durationMs": 34,
  "isOperational": true,
  "stack": null
}
```

> **Note:** The `stack` field is only populated for `CRITICAL` level entries and is **never** included in API responses — only in internal log sinks.

### 9.4 Log Redaction Rules

The following fields must **never** appear in log output:

| Field / Pattern | Action |
|---|---|
| `password`, `passwordHash` | Omit entirely |
| `token`, `accessToken`, `refreshToken` | Omit entirely |
| `Authorization` header value | Omit entirely |
| `cardNumber`, `cvv`, `expiryDate` | Omit entirely |
| Request body size > 10 KB | Truncate to first 10 KB; append `[TRUNCATED]` |
| Stack trace | Internal logs only; never in API response |

---

## 10. DevOps Notification Strategy

### 10.1 Alert Triggers

| Trigger Condition | Severity | Channel | SLA Response Time |
|---|---|---|---|
| `isOperational === false` (unhandled bug) | CRITICAL | PagerDuty + Slack `#incidents` | 15 min |
| HTTP 5xx spike > 5 errors/min | CRITICAL | PagerDuty + Slack `#incidents` | 15 min |
| `SYSTEM_DATABASE_UNAVAILABLE` | CRITICAL | PagerDuty + Slack `#incidents` | 15 min |
| `AUTH_ACCOUNT_LOCKED` > 10 events/min | ERROR | Slack `#security` | 30 min |
| `SYSTEM_TIMEOUT` > 3 consecutive occurrences | ERROR | Slack `#ops` | 1 hour |
| `SYSTEM_RATE_LIMIT_EXCEEDED` sustained > 5 min | WARN | Slack `#ops` | 2 hours |

### 10.2 Alert Payload Structure

```json
{
  "environment": "production",
  "severity": "CRITICAL",
  "errorCode": "SYSTEM_INTERNAL_ERROR",
  "message": "An unexpected error occurred.",
  "traceId": "c1a2b3d4-e5f6-7890-abcd-ef1234567890",
  "timestamp": "2026-10-14T13:02:00.000Z",
  "endpoint": "POST /api/v1/workout",
  "userId": "usr_abc123",
  "isOperational": false
}
```

### 10.3 Environment Routing Rules

| Environment | Console | Slack | PagerDuty | Structured Log Sink |
|---|---|---|---|---|
| Development | ✅ | ❌ | ❌ | ❌ |
| Staging | ✅ | ✅ | ❌ | ✅ |
| Production | ✅ | ✅ | ✅ | ✅ |

---

## 11. Validation Integration

The Error Handling Module owns the Zod→ValidationError normalisation pipeline. No other module may directly catch or transform `ZodError` instances.

### 11.1 Zod Error Mapper

```typescript
import { ZodError } from "zod";
import { ValidationError } from "@/errors/subclasses/ValidationError";

function mapZodError(err: ZodError): ValidationError {
  return new ValidationError({
    message: "Request validation failed.",
    details: err.issues.map((issue) => ({
      field: issue.path.join("."),
      code: `VALIDATION_${issue.code.toUpperCase()}`,
      message: issue.message,
    })),
  });
}
```

### 11.2 Validation Middleware Pattern

Route-level validation middleware passes `ZodError` to the global error handler via `next()`. The global error handler then calls `mapZodError` automatically.

```typescript
import { ZodSchema } from "zod";
import { mapZodError } from "@/errors/utils/mapZodError";

const validateBody = (schema: ZodSchema) =>
  (req: Request, res: Response, next: NextFunction): void => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return next(mapZodError(result.error));
    }
    req.body = result.data;
    next();
  };

// Usage on a route:
router.post(
  "/workout",
  validateBody(CreateWorkoutSchema),
  wrapAsync(workoutController.create)
);
```

### 11.3 Validation Response Example

A request failing validation returns:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_INVALID_FORMAT",
    "message": "Request validation failed.",
    "statusCode": 422,
    "traceId": "c1a2b3d4-e5f6-7890-abcd-ef1234567890",
    "timestamp": "2026-10-14T13:02:00.000Z",
    "details": [
      { "field": "email", "code": "VALIDATION_INVALID_FORMAT", "message": "Invalid email" },
      { "field": "age", "code": "VALIDATION_TOO_SMALL", "message": "Must be at least 13" }
    ],
    "meta": {}
  }
}
```

---

## 12. TDD Testing Strategy

### 12.1 TDD Principles for This Module

- All error classes, utility functions, and middleware **must have tests written before any implementation**.
- Tests define the contract; implementation satisfies the tests.
- No code is merged to `main` without 100% line coverage on error classes and utilities.
- **Test runner:** Jest
- **Integration testing:** Supertest (HTTP middleware integration)
- **All P0 tests must pass before any other NeuByte module begins implementation.**

### 12.2 Test Plan

#### Suite 1: AppError Base Class

| Test Case | Type | Priority |
|---|---|---|
| Constructs with correct `errorCode`, `statusCode`, `traceId`, `timestamp` | Unit | P0 |
| `toEnvelope()` returns a valid `ErrorEnvelope` shape | Unit | P0 |
| `isOperational` defaults to `true` | Unit | P1 |
| `isOperational` can be explicitly set to `false` | Unit | P0 |
| `traceId` is a valid UUID v4 | Unit | P1 |
| `timestamp` is a valid ISO 8601 UTC string | Unit | P1 |
| `details` defaults to an empty array | Unit | P1 |
| `meta` defaults to an empty object | Unit | P1 |

#### Suite 2: Exception Subclasses

| Test Case | Type | Priority |
|---|---|---|
| `ValidationError` produces status `422` and a `VALIDATION_*` code | Unit | P0 |
| `TokenExpiredError` produces status `401` and code `AUTH_TOKEN_EXPIRED` | Unit | P0 |
| `NotFoundError` produces status `404` and correct domain code | Unit | P0 |
| `ConflictError` produces status `409` | Unit | P0 |
| `InternalError` sets `isOperational` to `false` | Unit | P0 |
| `RateLimitError` produces status `429` | Unit | P1 |
| `ServiceUnavailableError` produces status `503` | Unit | P1 |

#### Suite 3: Global Error Handler Middleware

| Test Case | Type | Priority |
|---|---|---|
| `AppError` is serialised to envelope and returned with correct HTTP status | Integration | P0 |
| `ZodError` is converted to a `ValidationError` envelope | Integration | P0 |
| Unknown `Error` is wrapped in `InternalError` with `isOperational: false` | Integration | P0 |
| Response never includes `stack` in production mode | Integration | P0 |
| `traceId` from request context appears in response envelope | Integration | P1 |
| Sensitive fields are redacted from `error.details` | Integration | P0 |
| `X-Trace-Id` response header matches envelope `traceId` | Integration | P1 |
| Middleware calls `next()` only once per error | Unit | P1 |

#### Suite 4: Utility Functions

| Test Case | Type | Priority |
|---|---|---|
| `wrapAsync` forwards async rejection to `next(err)` | Unit | P0 |
| `wrapAsync` does not intercept synchronous errors | Unit | P1 |
| `throwIf(true, err)` throws the provided error | Unit | P0 |
| `throwIf(false, err)` does not throw | Unit | P0 |
| `assertDefined(undefined, err)` throws the provided error | Unit | P0 |
| `assertDefined(value, err)` returns the value unchanged | Unit | P0 |
| `mapToAppError(new Error("x"))` returns an `InternalError` | Unit | P1 |
| `mapToAppError(appError)` returns the error unchanged | Unit | P1 |
| `redactSensitiveFields` removes `password` from details | Unit | P0 |
| `redactSensitiveFields` removes `token` from details | Unit | P0 |
| `redactSensitiveFields` does not remove non-sensitive fields | Unit | P1 |

#### Suite 5: Logging

| Test Case | Type | Priority |
|---|---|---|
| `CRITICAL` errors include `stack` in log output | Unit | P1 |
| `INFO` errors do not include `stack` in log output | Unit | P1 |
| Log entry includes `traceId`, `errorCode`, `statusCode` | Unit | P0 |
| Production mode never exposes `stack` in API response body | Integration | P0 |
| `userId` appears in log if request is authenticated | Unit | P1 |

#### Suite 6: Alert Triggers

| Test Case | Type | Priority |
|---|---|---|
| `isOperational === false` triggers alerting service | Unit | P0 |
| Status code ≥ 500 triggers alerting service | Unit | P0 |
| Status code 4xx does not trigger alerting service | Unit | P1 |
| Alerting service is not called in development environment | Unit | P0 |

### 12.3 Coverage Requirements

| Metric | Threshold | Scope |
|---|---|---|
| Line Coverage | 100% | Error classes and utility functions |
| Branch Coverage | 95% | Global error handler middleware |
| P0 Test Pass Rate | 100% | Required before any other module starts |
| CI Coverage Gate | Blocks merge | Coverage below threshold blocks PR merge |

### 12.4 Test File Structure

```
src/
├── errors/
│   ├── AppError.ts
│   ├── registry/
│   │   └── errorCodes.ts
│   ├── subclasses/
│   │   ├── ValidationError.ts
│   │   ├── AuthenticationError.ts
│   │   ├── TokenExpiredError.ts
│   │   ├── AuthorizationError.ts
│   │   ├── NotFoundError.ts
│   │   ├── ConflictError.ts
│   │   ├── UnprocessableError.ts
│   │   ├── RateLimitError.ts
│   │   ├── InternalError.ts
│   │   └── ServiceUnavailableError.ts
│   ├── middleware/
│   │   └── globalErrorHandler.ts
│   └── utils/
│       ├── wrapAsync.ts
│       ├── throwIf.ts
│       ├── assertDefined.ts
│       ├── mapToAppError.ts
│       ├── mapZodError.ts
│       └── redactSensitiveFields.ts
│
tests/
└── errors/
    ├── AppError.test.ts
    ├── subclasses/
    │   ├── ValidationError.test.ts
    │   ├── AuthenticationError.test.ts
    │   ├── TokenExpiredError.test.ts
    │   ├── NotFoundError.test.ts
    │   ├── ConflictError.test.ts
    │   ├── InternalError.test.ts
    │   └── ServiceUnavailableError.test.ts
    ├── middleware/
    │   └── globalErrorHandler.test.ts
    └── utils/
        ├── wrapAsync.test.ts
        ├── throwIf.test.ts
        ├── assertDefined.test.ts
        ├── mapToAppError.test.ts
        ├── mapZodError.test.ts
        └── redactSensitiveFields.test.ts
```

---

## 13. Appendix — Architectural Decision Records (ADRs)

### ADR-001: Single Error Envelope Shape

- **Decision:** All API errors — regardless of domain or type — return the identical JSON envelope shape.
- **Alternatives Considered:** Domain-specific error shapes (e.g. Auth errors returning a different shape than Workout errors).
- **Rationale:** A single shape reduces frontend complexity, simplifies error monitoring (one schema to query), and enables consistent logging. The `code` field provides all necessary domain specificity.
- **Status:** Accepted

---

### ADR-002: `isOperational` Flag on AppError

- **Decision:** Every `AppError` instance carries an `isOperational` boolean that distinguishes expected errors (`true`) from bugs (`false`).
- **Alternatives Considered:** Separate error class trees for operational and non-operational errors; relying solely on HTTP status codes.
- **Rationale:** Status codes alone cannot distinguish a genuine `500` from an unhandled bug. The `isOperational` flag enables smart alerting — only non-operational errors page on-call engineers.
- **Status:** Accepted

---

### ADR-003: Zod as Primary Validation Library

- **Decision:** Zod is the canonical schema validation library for NeuByte. The Error Handling Module owns the `ZodError → ValidationError` mapper.
- **Alternatives Considered:** Joi (runtime-only), class-validator (decorator-based), custom validators.
- **Rationale:** Zod's TypeScript-first design provides compile-time type inference from schemas, eliminating a class of runtime type errors. Centralising the mapper in the Error Handling Module ensures consistent validation error shapes across all routes.
- **Status:** Accepted

---

### ADR-004: Trace ID Propagation via UUID v4

- **Decision:** Every inbound request is assigned a UUID v4 `traceId` at ingress middleware. This ID propagates through all error envelopes, log entries, and the `X-Trace-Id` response header.
- **Alternatives Considered:** Numeric request IDs; W3C `traceparent` header (deferred for future microservice expansion).
- **Rationale:** UUID v4 provides sufficient entropy for correlation across distributed services with zero coordination overhead. Propagating the ID end-to-end enables support engineers to reconstruct a full error chain from a single ID reported by a user.
- **Status:** Accepted

---

*NeuByte Fitness App — Confidential Internal Document*
*Version 1.0.0 · October 14, 2026 · Error Handling Module Architecture*
