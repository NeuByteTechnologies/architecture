# Security Roles Matrix

## Purpose

This document defines the security principals, permissions, and resource access requirements for the Fitness Application.

This matrix supports ADR-014 Security Architecture and serves as the authoritative reference for:

- Role-Based Access Control (RBAC)
- Database Security
- Schema Authorization
- Service Identities
- File System Security
- Operational Support Access
- DevOps and Deployment Security

## Security Principles

### Least Privilege

All principals shall receive only the permissions necessary to perform assigned responsibilities.

### Separation of Duties

Human users and service identities shall be separated.

### Schema-Based Authorization

Database permissions should be granted at the schema level where practical.

### Defense in Depth

Authorization should be enforced at:

- Application Layer
- Service Layer
- Database Layer
- File System Layer

---

# Human Roles

| Role | Purpose | Database Permissions | Schemas |
|--------|--------|--------|--------|
| FitnessApplicationAdmin | Application administration and operational management | SELECT, INSERT, UPDATE, EXECUTE | All |
| FitnessApplicationTester | Validation, integration testing, and verification | SELECT, INSERT, EXECUTE, VIEW DEFINITION | audit, dbo, sys, information_schema |
| FitnessApplicationAuditor | Audit review and compliance reporting | SELECT, EXECUTE | audit, datamart |
| FitnessApplicationSupport | Production support and troubleshooting | SELECT, INSERT, UPDATE, EXECUTE | All |
| FitnessApplicationUser | Normal application usage | SELECT, INSERT, UPDATE, EXECUTE | dbo |
| FitnessApplicationReporting | Reporting and analytics | SELECT, EXECUTE | datamart |
| FitnessApplicationETL | Data movement and transformation operations | SELECT, INSERT, UPDATE, EXECUTE | audit, dbo, star, datamart |

---

# Service Roles

| Service Role | Purpose | Database Permissions | Schemas |
|--------|--------|--------|--------|
| ApplicationService | Primary application workload identity | SELECT, INSERT, UPDATE, EXECUTE | dbo |
| IdentityService | Identity and membership management | SELECT, INSERT, UPDATE, EXECUTE | dbo |
| AuditWriter | Audit event persistence | INSERT, EXECUTE | audit |
| LogWriter | Logging persistence | INSERT | audit |
| ETLService | Data warehouse and ETL processing | SELECT, INSERT, UPDATE, EXECUTE | star, datamart |

---

# Active Directory Role Mapping

| Active Directory Group | Application Role |
|--------|--------|
| FitnessApplicationAdmins | FitnessApplicationAdmin |
| FitnessApplicationTesters | FitnessApplicationTester |
| FitnessApplicationAuditors | FitnessApplicationAuditor |
| FitnessApplicationSupport | FitnessApplicationSupport |
| FitnessApplicationReporting | FitnessApplicationReporting |
| FitnessApplicationUsers | FitnessApplicationUser |
| FitnessApplicationETL | FitnessApplicationETL |

---

# File System Permissions

## Configuration Files

| Role | Permission |
|--------|--------|
| FitnessApplicationAdmin | Read, Write |
| ApplicationService | Read |
| IdentityService | Read |

### Examples

- appsettings.json
- environment configuration
- deployment configuration

---

## Log Files

| Role | Permission |
|--------|--------|
| FitnessApplicationSupport | Read |
| FitnessApplicationAdmin | Read |
| LogWriter | Create, Append |

### Examples

- application logs
- diagnostic logs
- exception logs

---

## Audit Exports

| Role | Permission |
|--------|--------|
| FitnessApplicationAuditor | Read |
| FitnessApplicationAdmin | Read |
| AuditWriter | Create |

### Examples

- audit extracts
- compliance exports

---

## Data Import Directory

| Role | Permission |
|--------|--------|
| FitnessApplicationAdmin | Read, Write |
| FitnessApplicationTester | Read, Write |
| ETLService | Read, Write |

### Examples

- exercise catalogs
- program imports
- configuration imports

---

## Data Export Directory

| Role | Permission |
|--------|--------|
| FitnessApplicationReporting | Read |
| FitnessApplicationAdmin | Read |
| ETLService | Create |

### Examples

- reporting exports
- data extracts
- business intelligence exports

---

# Database Security Boundaries

## dbo Schema

Primary application tables and operational data.

Authorized Roles:

- FitnessApplicationAdmin
- FitnessApplicationSupport
- FitnessApplicationUser
- FitnessApplicationTester
- ApplicationService
- IdentityService

---

## audit Schema

Audit and security events.

Authorized Roles:

- FitnessApplicationAuditor
- FitnessApplicationAdmin
- FitnessApplicationTester
- AuditWriter

---

## datamart Schema

Reporting and analytical datasets.

Authorized Roles:

- FitnessApplicationReporting
- FitnessApplicationAuditor
- FitnessApplicationETL
- ETLService

---

## star Schema

Star-schema warehouse structures.

Authorized Roles:

- FitnessApplicationETL
- ETLService

---

## sys and information_schema

Metadata visibility required for testing and validation activities.

Authorized Roles:

- FitnessApplicationTester

---

# DbContext Security Mapping

| DbContext | Service Identity | Primary Schema |
|--------|--------|--------|
| ApplicationDbContext | ApplicationService | dbo |
| IdentityDbContext | IdentityService | dbo |
| AuditDbContext | AuditWriter | audit |
| ReportingDbContext | ETLService | datamart |
| WarehouseDbContext | ETLService | star |

---

# Future Considerations

Future revisions may
