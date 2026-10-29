# ADR-008: Logging Persistence Hierarchy

This is actually separate.

## Decision

Logging persistence follows a reliability hierarchy.

## Plain Text

**Tier 1**
SQL Database
 
**Tier 2**
File Logging
 
**Tier 3**
Windows Event Viewer

Each tier exists as a failover mechanism for the previous tier.

Logging providers do not duplicate records under normal operation.

## Rationale

**Avoid:**

## Plain Text
- Storage duplication
- Synchronization problems
- Multiple systems of record

while preserving diagnostic persistence during infrastructure failures.
