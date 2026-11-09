# ADR-014: Security Architecture

**Status:** Accepted
**Date:** 2026-11-08
**Owner:** Architecture & Systems Design

---

# 1. Context

The Fitness platform contains multiple components requiring security controls including:

- ASP.NET Core Web Applications
- APIs
- Background Services
- Entity Framework Core Data Access
- SQL Server Databases
- GitHub Automation
- ETL Processes
- Reporting Platforms
- Administrative Functions

As the platform evolves, security requirements extend beyond application login functionality and include:

- Authentication
- Authorization
- Database access
- Service account management
- Secret management
- Auditability
- Operational monitoring

Security decisions should be standardized across the platform rather than implemented independently by individual applications or services.

---

# 2. Decision

The Fitness platform shall adopt a layered security model based upon:

- Defense in Depth
- Principle of Least Privilege
- Role-Based Access Control (RBAC)
- Separation of Duties
- Auditable Security Operations
- Secure Secret Management

Security controls shall be enforced consistently across:

- Applications
- APIs
- Databases
- Infrastructure Services
- Automation Platforms

---

# 3. Security Principles

## 3.1 Defense in Depth

Security shall not rely upon a single control.

Multiple layers of protection should exist whenever practical.

Examples:

```text
Authentication
    ↓
Authorization
    ↓
Application Validation
    ↓
Database Permissions
    ↓
Audit Logging
```

Failure of a single security control should not result in unrestricted system access.

---

## 3.2 Principle of Least Privilege

Users, services, and applications shall receive only the permissions required to perform their intended functions.

Access should be granted:

```text
As Little As Possible
For As Short As Possible
```

Permissions should never be granted solely for convenience.

---

## 3.3 Separation of Duties

Administrative responsibilities should be separated whenever practical.

Examples include:

- Application Administration
- Database Administration
- Infrastructure Administration
- Source Control Administration
- Reporting Administration

No single account should require unrestricted access across all platform components unless explicitly justified.

---

## 3.4 Auditability

Security-relevant actions shall be traceable.

Examples include:

- Authentication events
- Authorization failures
- Administrative changes
- Security configuration changes
- Service account activity

Security events should be retained in audit structures where practical.

---

# 4. Authentication Strategy

## 4.1 User Authentication

Interactive users shall authenticate through approved identity providers.

Future implementations may include:

- ASP.NET Identity
- Microsoft Entra ID
- OAuth Providers
- Social Identity Providers

Authentication mechanisms shall support secure credential handling and modern security practices.

---

## 4.2 Multi-Factor Authentication

Administrative and privileged accounts should require Multi-Factor Authentication (MFA) whenever supported by the platform.

Examples:

- Administrative portals
- Deployment platforms
- Source control systems
- Cloud administration consoles

---

## 4.3 Service Authentication

Non-human processes shall authenticate using service identities rather than user accounts.

Examples:

- ETL jobs
- Background services
- Scheduled tasks
- CI/CD pipelines
- Reporting services

---

# 5. Authorization Strategy

## 5.1 Role-Based Access Control (RBAC)

Authorization shall primarily utilize Role-Based Access Control (RBAC).

The model is:

```text
User
    ↓
Role
    ↓
Permission
```

Roles represent responsibilities.

Permissions represent capabilities.

Direct permission assignment should be minimized.

---

## 5.2 Role Design

Roles should reflect business and operational responsibilities.

Examples:

```text
Fitness User

Coach

Administrator

Application Support

Data Steward

Reporting User
```

Roles may be expanded as platform requirements evolve.

---

## 5.3 Administrative Access

Administrative operations should require elevated authorization beyond standard user permissions.

Examples:

- User administration
- Reference data maintenance
- System configuration
- Security configuration

---

# 6. Service Account Strategy

## 6.1 Service Accounts

Automated processes shall utilize dedicated service accounts.

Service accounts should:

- Have defined ownership
- Have documented purpose
- Follow least privilege
- Be auditable

Examples:

```text
Application Runtime

Audit Logging

GitHub ETL

Reporting Services

Deployment Automation
```

---

## 6.2 Shared Accounts

Shared administrative accounts should be avoided whenever practical.

Named ownership is preferred.

---

# 7. Secret Management

## 7.1 Secret Storage

Secrets shall not be hardcoded into source code.

Examples include:

- Database passwords
- API keys
- Access tokens
- Encryption keys
- Service account credentials

---

## 7.2 Approved Storage Locations

Examples include:

```text
User Secrets

Environment Variables

Azure Key Vault

GitHub Secrets

Enterprise Secret Managers
```

The approved mechanism may vary by environment.

---

## 7.3 Source Control Restrictions

Secrets shall never be committed to source control repositories.

Automated scanning should be adopted when practical.

---

# 8. API Security

## 8.1 API Authentication

APIs shall require authenticated access unless explicitly designated as public.

---

## 8.2 API Authorization

Authenticated access does not imply authorization.

Authorization requirements must be validated independently.

---

## 8.3 API Tokens

API keys and access tokens shall:

- Have defined ownership
- Have defined expiration policies where supported
- Follow least privilege principles
- Be securely stored

---

# 9. Database Security

## 9.1 Security Philosophy

Database security is an extension of the overall platform security model.

Database permissions shall follow:

- Least Privilege
- Separation of Duties
- Auditable Access

---

## 9.2 Schema-Level Security

Where practical, permissions should be granted at the schema level rather than directly to individual objects.

Examples:

```text
dbo
audit
star
datamart
github
```

This allows logical grouping of permissions and simplifies administration.

---

## 9.3 Service Account Access

Database access should occur through dedicated service identities.

Examples:

```text
Application Runtime Account

Audit Logging Account

ETL Account

Reporting Account
```

Access shall be limited to required schemas and operations whenever practical.

---

## 9.4 Administrative Database Access

Database administrative permissions shall be restricted to authorized administrators.

Routine application operations shall not require administrative rights.

---

# 10. Audit and Monitoring

## 10.1 Security Event Collection

Security-relevant events should be recorded whenever practical.

Examples:

- Login failures
- Authorization failures
- Administrative actions
- Service account activity

---

## 10.2 Monitoring Objectives

Monitoring should support:

- Incident Investigation
- Root Cause Analysis
- Operational Visibility
- Security Auditing

---

# 11. Future Considerations

The architecture should support future adoption of:

- Microsoft Entra ID
- Managed Identities
- Azure Key Vault
- Centralized Identity Providers
- Enterprise Security Monitoring
- Threat Detection Platforms

These capabilities may be introduced without changing the core security principles defined in this ADR.

---

# 12. Consequences

## Positive

- Consistent security architecture
- Centralized security principles
- Reduced privilege exposure
- Improved auditability
- Easier regulatory alignment
- Better operational visibility
- Stronger separation of duties

## Negative

- Additional administration effort
- Increased configuration complexity
- Additional documentation requirements
- Greater testing requirements

---

# 13. Related Documents

- ADR-010 Logging Severity and Verbosity Strategy
- ADR-011 Entity Framework Core Data Access Strategy
- ADR-012 Migration from Azure MySQL to SQL Server 2025
- ADR-013 Database Governance Standards
- ADR-015 Entity Framework Core Context Strategy
- SystemArchitecture.md
- InfrastructureArchitecture.md

---

# 14. Version History

| Version | Date | Author | Description |
|----------|----------|----------|----------|
| 1.0 | 2026-11-08 | Gordon | Established platform security architecture, RBAC, service account strategy, secret management standards, API security standards, and database security principles. |
