# File System Security Matrix

## Purpose

This document defines file system access requirements for the Fitness Application.

This matrix supplements:

- ADR-014 Security Architecture
- Security-Roles-Matrix.md
- Database-Security-Matrix.md
- Service-Role-Matrix.md

The purpose of this document is to define which security principals may access files, directories, and storage resources.

---

# Security Principles

## Least Privilege

Access to files and directories shall be restricted to the minimum permissions required.

## Separation of Duties

Human users and service identities shall be granted different permissions based on responsibilities.

## Role-Based Access Control

File system permissions shall be assigned through roles rather than individual principals whenever practical.

---

# Permission Legend

| Permission | Description |
|------------|------------|
| R | Read |
| W | Write |
| C | Create |
| M | Modify |
| D | Delete |
| - | No Access |

---

# File System Resources

## Application Configuration

Examples:

- appsettings.json
- appsettings.Development.json
- appsettings.Production.json

Purpose:

- Application configuration
- Connection string configuration
- Feature configuration

---

## Log Directory

Examples:

- Application Logs
- Error Logs
- Diagnostic Logs

Purpose:

- Troubleshooting
- Operational Support
- Monitoring

---

## Audit Export Directory

Examples:

- Audit Reports
- Compliance Exports

Purpose:

- Audit review
- Compliance reporting

---

## Import Directory

Examples:

- Exercise Imports
- Program Imports
- Administrative Data Loads

Purpose:

- Administrative maintenance
- Data loading

---

## Export Directory

Examples:

- Reporting exports
- Administrative exports

Purpose:

- Reporting
- Data extraction

---

# Human Role Access Matrix

| Role | Config | Logs | Audit Exports | Imports | Exports |
|--------|--------|--------|--------|--------|--------|
| FitnessApplicationAdmin | R | R | R | R,W,C,M,D | R |
| FitnessApplicationTester | R | R | - | R,W,C,M,D | R |
| FitnessApplicationAuditor | - | - | R | - | R |
| FitnessApplicationSupport | R | R | - | - | - |
| FitnessApplicationUser | - | - | - | - | - |

---

# Service Role Access Matrix

| Service Role | Config | Logs | Audit Exports | Imports | Exports |
|--------|--------|--------|--------|--------|--------|
| ApplicationService | R | - | - | R | R |
| IdentityService | R | - | - | - | - |
| AuditWriter | - | - | C | - | - |
| LogWriter | - | C,M | - | - | - |

---

# Administrative Responsibilities

## FitnessApplicationAdmin

Responsible for:

- Configuration management
- Data import management
- Export review
- Operational oversight

---

## FitnessApplicationSupport

Responsible for:

- Log review
- Troubleshooting
- Incident investigation

---

## FitnessApplicationAuditor

Responsible for:

- Audit report review
- Compliance verification

---

# Directory Ownership

| Directory | Owner |
|------------|------------|
| Configuration | FitnessApplicationAdmin |
| Logs | LogWriter |
| Audit Exports | AuditWriter |
| Imports | FitnessApplicationAdmin |
| Exports | FitnessApplicationAdmin |

---

# Sensitive Files

The following files may contain sensitive information and require additional protection:

- Production configuration files
- Connection string configuration
- Audit exports
- Administrative exports

These resources should not be accessible to standard application users.

---

# Current Scope

This document applies only to file resources currently identified within the Fitness Application.

Additional file system resources shall be documented when introduced through approved architectural decisions.

---

# References

- ADR-014 Security Architecture
- Security-Roles-Matrix.md
- Database-Security-Matrix.md
- Service-Role-Matrix.md
