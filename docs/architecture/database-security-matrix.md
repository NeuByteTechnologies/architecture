# Database Security Matrix

## Purpose

This document defines database security boundaries, roles, schema authorization, and service account access within the Fitness Application ecosystem.

This document supports:

- ADR-014 Security Architecture
- RBAC Implementation
- Least Privilege Enforcement
- Separation of Duties
- Database Administration Standards

---

# Security Principles

## Least Privilege

Database principals shall receive only the permissions required to perform assigned responsibilities.

## Schema Based Authorization

Permissions shall be granted primarily at the schema level rather than individual object level whenever practical.

## Separation of Duties

Human users and service identities shall be assigned separate database roles.

## Auditability

Database access should support auditing, monitoring, and compliance requirements.

---

# Database Schemas

| Schema | Purpose |
|----------|----------|
| dbo | Operational application data |
| audit | Audit and compliance records |
| datamart | Reporting and analytical datasets |
| star | Dimensional warehouse structures |
| security | Roles, permissions, and security metadata |
| staging | Temporary ETL processing |
| integration | Import/export integration data |

---

# Database Roles

## Human Roles

| Database Role | Description |
|---------------|-------------|
| FitnessApplicationAdmin | Administrative access to application data |
| FitnessApplicationTester | Testing and validation activities |
| FitnessApplicationAuditor | Audit review and compliance reporting |
| FitnessApplicationSupport | Production support and troubleshooting |
| FitnessApplicationUser | Normal business operations |
| FitnessApplicationReporting | Reporting and analytics |
| FitnessApplicationETL | Data movement and transformation |

---

## Service Roles

| Database Role | Description |
|---------------|-------------|
| ApplicationService | Primary application workload |
| IdentityService | Identity subsystem |
| AuditWriter | Audit persistence |
| LogWriter | Logging persistence |
| ETLService | ETL workload execution |

---

# Schema Authorization Matrix

| Role | dbo | audit | datamart | star |
|--------|--------|--------|--------|--------|
| FitnessApplicationAdmin | RWX | RWX | R | R |
| FitnessApplicationTester | RWX | RWX | R | R |
| FitnessApplicationAuditor | - | R | R | - |
| FitnessApplicationSupport | RWX | R | R | - |
| FitnessApplicationUser | RWX | - | - | - |
| FitnessApplicationReporting | - | R | R | R ||
| FitnessApplicationETL | RWX | RWX | RWX | RWX |
| ApplicationService | RWX | - | - | - |
| IdentityService | RWX | - | - | - |
| AuditWriter | - | WX | - | - |
| LogWriter | - | WX | - | - |
| ETLService | RWX | RWX | RWX | RWX |

Legend:

- R = Read
- W = Write
- X = Execute
- - = No Access

---

# Database Permission Matrix

| Role | Select | Insert | Update | Delete | Execute | View Definition |
|---------|---------|---------|---------|---------|---------|---------|
| FitnessApplicationAdmin | Yes | Yes | Yes | Controlled | Yes | Yes |
| FitnessApplicationTester | Yes | Yes | Yes | No | Yes | Yes |
| FitnessApplicationAuditor | Yes | No | No | No | Yes | No |
| FitnessApplicationSupport | Yes | Yes | Yes | No | Yes | Yes |
| FitnessApplicationUser | Yes | Yes | Yes | No | Yes | No |
| FitnessApplicationReporting | Yes | No | No | No | Yes | No |
| FitnessApplicationETL | Yes | Yes | Yes | Yes | Yes | Yes |
| ApplicationService | Yes | Yes | Yes | No | Yes | No |
| IdentityService | Yes | Yes | Yes | No | Yes | No |
| AuditWriter | No | Yes | No | No | Yes | No |
| LogWriter | No | Yes | No | No | No | No |
| ETLService | Yes | Yes | Yes | Yes | Yes | Yes |

---

# Metadata Access

The following roles require metadata visibility.

| Role | sys | information_schema | View Definition |
|--------|--------|--------|--------|
| FitnessApplicationTester | Yes | Yes | Yes |
| FitnessApplicationAdmin | Yes | Yes | Yes |
| FitnessApplicationSupport | Yes | Yes | Yes |
| FitnessApplicationETL | Yes | Yes | Yes |

---

# Service Account Mapping

| Service Identity | Database Role | Primary Schema |
|------------------|---------------|----------------|
| ApplicationService | ApplicationService | dbo |
| IdentityService | IdentityService | security, dbo |
| AuditDbContext | AuditWriter | audit |
| LoggingService | LogWriter | audit |
| ETLService | ETLService | staging, star, datamart |

---

# DbContext Authorization

| DbContext | Database Role | Schemas |
|------------|------------|------------|
| ApplicationDbContext | ApplicationService | dbo |
| IdentityDbContext | IdentityService | security, dbo |
| AuditDbContext | AuditWriter | audit |
| ReportingDbContext | ETLService | datamart |
| WarehouseDbContext | ETLService | star |

---

# Security Boundaries

## Operational Boundary

Primary transaction processing.

Schemas:

- dbo
- security

## Audit Boundary

Audit and compliance data.

Schemas:

- audit

## Reporting Boundary

Business intelligence and reporting.

Schemas:

- datamart

## Data Warehouse Boundary

Analytical processing.

Schemas:

- star
- staging

---

# Future Enhancements

Potential future additions include:

- Azure Managed Identities
- Row Level Security (RLS)
- Dynamic Data Masking
- Transparent Data Encryption
- Data Classification
- Database Activity Monitoring
- Azure SQL Role Integration

---

# References

- ADR-014 Security Architecture
- Security Roles Matrix
- Identity Architecture
- Data Architecture
- Deployment Architecture
