# Service Role Matrix

## Purpose

This document defines service identities, workload permissions, and resource access requirements for the Fitness Application.

This document supplements:

- ADR-014 Security Architecture
- Security-Roles-Matrix.md
- Database-Security-Matrix.md

The purpose of this document is to define what automated system components may access and which permissions are required.

---

# Security Principles

## Least Privilege

Service identities shall receive only the permissions required to perform their assigned responsibilities.

## Separation of Duties

Service identities shall not share credentials or permissions with human users.

## Dedicated Identity

Each service shall have its own security principal whenever practical.

---

# Service Roles

| Service Role | Purpose |
|--------------|----------|
| ApplicationService | Primary application workload |
| IdentityService | Identity and authentication management |
| AuditWriter | Audit event persistence |
| LogWriter | Operational logging persistence |

---

# Service Responsibilities

## ApplicationService

### Responsibilities

- Application business logic
- CRUD operations
- User workflows
- Exercise management
- Program management
- Workout management

### Database Access

| Schema | Permissions |
|----------|----------|
| dbo | SELECT, INSERT, UPDATE, EXECUTE |

### File Access

| Resource | Permission |
|----------|----------|
| Application Configuration | Read |

### Notes

This is the primary runtime identity used by ApplicationDbContext.

---

## IdentityService

### Responsibilities

- User management
- Role assignment
- Authentication support
- Authorization support

### Database Access

| Schema | Permissions |
|----------|----------|
| dbo | SELECT, INSERT, UPDATE, EXECUTE |

### File Access

| Resource | Permission |
|----------|----------|
| Application Configuration | Read |

### Notes

Used by IdentityDbContext.

---

## AuditWriter

### Responsibilities

- Record audit events
- Record security events
- Record administrative actions
- Preserve audit trail

### Database Access

| Schema | Permissions |
|----------|----------|
| audit | INSERT, EXECUTE |

### File Access

| Resource | Permission |
|----------|----------|
| None | None |

### Notes

Used by AuditDbContext.

Audit records should not be updated or deleted through this role.

---

## LogWriter

### Responsibilities

- Application logging
- Exception logging
- Diagnostic logging

### Database Access

| Schema | Permissions |
|----------|----------|
| audit | INSERT |

### File Access

| Resource | Permission |
|----------|----------|
| Log Directory | Create, Append |

### Notes

Used when logging is persisted to database and/or file storage.

---

# DbContext Mapping

| DbContext | Service Role |
|------------|------------|
| ApplicationDbContext | ApplicationService |
| IdentityDbContext | IdentityService |
| AuditDbContext | AuditWriter |

---

# Resource Access Summary

| Service Role | Database | Configuration Files | Log Files |
|----------|----------|----------|----------|
| ApplicationService | Read / Write | Read | No Access |
| IdentityService | Read / Write | Read | No Access |
| AuditWriter | Write | No Access | No Access |
| LogWriter | Write | No Access | Create / Append |

---

# Prohibited Capabilities

The following capabilities are intentionally restricted from service identities unless a future architectural decision requires otherwise:

- ALTER DATABASE
- CREATE SCHEMA
- DROP TABLE
- ALTER TABLE
- Security Administration
- Role Administration
- User Administration
- Permission Grants

These responsibilities remain administrative functions.

---

# Future Review

Additional service roles shall only be introduced when supported by a documented architectural requirement.

Examples may include:

- ReportingService
- NotificationService
- BackgroundProcessingService

Such roles shall not be created until corresponding application capabilities exist.

---

# References

- ADR-014 Security Architecture
- Security-Roles-Matrix.md
- Database-Security-Matrix.md
