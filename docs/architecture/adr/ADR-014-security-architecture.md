# ADR-014: Security Architecture

## Status

Accepted

## Date

2026-11-18

---

# Context

The Fitness Application requires a security architecture that supports:

- Role-Based Access Control (RBAC)
- Principle of Least Privilege
- Separation of Duties
- Auditing and Traceability
- Application Security
- Database Security
- Service Security
- File System Security

The solution must support both human users and automated service workloads while remaining maintainable as the application evolves.

The security architecture should establish governance, authorization boundaries, and security principles without prescribing implementation details that are subject to change over time.

---

# Decision

The Fitness Application shall implement a layered security architecture utilizing:

- Role-Based Access Control (RBAC)
- Least Privilege
- Separation of Duties
- Schema-Based Database Authorization
- Dedicated Service Identities
- Defense in Depth

Security shall be enforced at:

- Application Layer
- Service Layer
- Database Layer
- File System Layer

---

# Security Principles

## Principle of Least Privilege

All users, services, and workloads shall receive only the permissions required to perform assigned responsibilities.

Access shall be granted based on business need.

Administrative permissions shall not be assigned to standard users or service workloads without documented justification.

---

## Separation of Duties

Security responsibilities shall be separated to reduce risk and improve accountability.

Examples include:

- Administrative Functions
- Audit Functions
- Support Functions
- Testing Functions
- Application Functions

No single principal should maintain unrestricted control over all security boundaries.

---

## Defense in Depth

Security controls shall exist at multiple layers.

Examples include:

- Authentication
- Authorization
- Database Permissions
- File Permissions
- Auditing
- Logging

Failure of a single security control shall not result in unrestricted access.

---

## Build for Known Requirements

Security controls shall be implemented to satisfy approved business and technical requirements.

Potential future requirements may be documented but shall not drive implementation decisions without an approved need.

Technology, schemas, services, and security boundaries shall be introduced when justified by current requirements.

The architecture shall avoid speculative implementation.

---

# Role-Based Access Control (RBAC)

## Human Roles

Human roles represent responsibilities assigned to individuals.

Examples include:

- FitnessApplicationAdmin
- FitnessApplicationSupport
- FitnessApplicationTester
- FitnessApplicationAuditor
- FitnessApplicationUser

Human roles define business responsibilities rather than technical implementation details.

---

## Service Roles

Service roles represent automated workloads.

Examples include:

- ApplicationService
- IdentityService
- AuditWriter
- LogWriter

Service identities shall be managed independently from human identities.

Service accounts shall not inherit human permissions.

---

## Authorization

Authorization shall be implemented through roles whenever practical.

Direct assignment of permissions to individual users should be minimized.

Roles represent responsibilities.

Permissions represent capabilities.

---

# Identity Architecture

## Dedicated Identities

Each service workload should utilize a dedicated identity.

Examples:

- ApplicationDbContext → ApplicationService
- IdentityDbContext → IdentityService
- AuditDbContext → AuditWriter

This improves:

- Auditing
- Accountability
- Permission Management
- Incident Investigation

---

## Human and Service Separation

Human users and automated services represent distinct security principals.

Human users shall not be used as service accounts.

Service accounts shall not be used interactively by users.

---

# Database Security

## Schema-Based Authorization

Database permissions shall be granted primarily at the schema level.

Examples:

- dbo
- audit

Schema-level security simplifies administration and aligns authorization with logical data ownership.

---

## Object-Level Authorization

Object-level permissions should be treated as exceptions.

Excessive object-level permission requirements may indicate:

- Poor schema boundaries
- Incomplete domain separation
- Security design issues

Schema design should be evaluated before introducing large numbers of object-specific permissions.

---

## Database Roles

Database access shall be assigned through roles.

Database roles should align with:

- Business Responsibilities
- Service Responsibilities
- Security Boundaries

---

## Metadata Access

Metadata visibility shall be granted only when required.

Examples include:

- Testing
- Database Validation
- Support Activities

Access to:

- sys views
- information_schema
- VIEW DEFINITION

shall require documented justification.

---

# Auditing

Security-relevant actions shall be auditable.

Examples include:

- User Administration
- Role Assignment
- Security Configuration Changes
- Data Imports
- Administrative Actions

Audit data shall be logically separated from operational data.

---

# Logging

Logging and auditing serve different purposes.

## Logging

Supports:

- Diagnostics
- Monitoring
- Troubleshooting
- Operations

## Auditing

Supports:

- Accountability
- Compliance
- Traceability
- Investigation

Audit data should not be treated as operational logging data.

---

# File System Security

Sensitive files shall be protected through appropriate security controls.

Examples include:

- Access Restrictions
- Role-Based Permissions
- Data Minimization
- Encryption When Required
- Integrity Validation When Required

Implementation mechanisms shall be determined by the requirements of the specific use case.

---

# Sensitive Data

Sensitive information should be minimized whenever practical.

The preferred approach is:

1. Avoid storing unnecessary sensitive data.
2. Utilize identifiers rather than personally identifiable information where feasible.
3. Restrict access through authorization controls.
4. Apply encryption when justified by business, regulatory, or security requirements.

Data protection requirements must be evaluated based on actual risk and business need.

---

# Security Governance

Security decisions shall be documented through Architecture Decision Records (ADRs).

Supporting implementation documentation shall be maintained separately from this ADR.

Changes to security architecture should be reviewed for impact on:

- Authorization
- Auditing
- Compliance
- Operational Support
- Application Architecture

---

# Supporting Documentation

The following documents implement and extend this ADR:

- Security-Roles-Matrix.md
- Database-Security-Matrix.md
- Service-Role-Matrix.md
- File-System-Security-Matrix.md

These supporting documents define role assignments, permissions, resource access, and implementation details.

---

# Consequences

## Positive

- Consistent security model
- Clear authorization boundaries
- Reduced administrative complexity
- Improved traceability
- Better separation of duties
- Simplified auditing

## Negative

- Additional documentation maintenance
- Increased operational governance
- More roles and permissions to manage

---

# References

- Security-Roles-Matrix.md
- Database-Security-Matrix.md
- Service-Role-Matrix.md
- File-System-Security-Matrix.md
