# ADR-010 Diagnostic Severity and Verbosity Strategy
## Status

Accepted

## Context

The FitnessApp logging architecture requires classification of diagnostic events while supporting multiple stakeholders:

- Plain Text
- Developer
- DevOps
- Auditor
- PMO

During design discussions, the distinction between:

- Plain Text
- Severity
- LogLevel
- Verbosity

became unclear.

Multiple approaches were considered:

- Severity only
- Severity + LogLevel
- Severity + Verbosity
- Severity + LogLevel + Verbosity

The framework is currently focused on structured exception logging persisted to SQL Server through the audit.ErrorLog repository.

## Decision

The logging architecture will utilize:

### Severity

Represents business impact and error classification.

C#
public enum Severity : byte
{
Information = 0,
Warning = 1,
Error = 2,
Critical = 3
}

**Severity indicates:**

- Plain Text
- How serious was the business event?

**Severity is stored in:**

- SQL
- Severity tinyint NOT NULL
- Verbosity

Represents the diagnostic depth captured for the log record.

Inspired by SQL Server Integration Services (SSIS) logging levels.

C#
public enum LogVerbosity : byte
{
Basic = 0,
Performance = 1,
Verbose = 2
}

**Verbosity indicates:**

- How much diagnostic information was captured?

**Verbosity is stored in:**

- SQL
- Verbosity tinyint NOT NULL
- LogLevel

Microsoft.Extensions.Logging.LogLevel will not be persisted in the ErrorLog table.

**Reasoning:**

- ErrorLog is an exception repository rather than a general-purpose application log.

Most exception records already convey importance through Severity.

Persisting LogLevel would duplicate information and increase complexity without providing significant reporting value.

## Verbosity Mapping
### Basic

**Intended for:**

- Auditor
- PMO
- Support

**Captured information:**

- CorrelationId
- TimestampUtc
- ReferenceCode
- ErrorCode
- Severity
- Message
- UserId
 
### Performance

**Intended for:**

- DevOps
- Operations
- Performance Analysis

**Captured information:**

- Basic Information
- ExceptionType
- Source
- Metadata
 
### Verbose

**Intended for:**

- Developer
- Root Cause Analysis

**Captured information:**

- Performance Information
- StackTrace
- Inner Exception Details
- Complete Diagnostic Context
- Database Design
- SQL
- Severity tinyint NOT NULL,
- Verbosity tinyint NOT NULL

**Enum values are persisted as:**

C# byte enum
↓
SQL tinyint

to maintain semantic alignment between application code and database storage.

Consequences

**Benefits:**

- Clear separation of concerns
- Severity represents impact
- Verbosity represents diagnostic depth
- SSIS-style terminology familiar to SQL professionals
- Supports Auditor, DevOps, Developer, and PMO use cases
- Supports future reporting and analytics

**Tradeoffs:**

- Additional column in ErrorLog
- More configuration choices
- Requires documentation of verbosity behavior
