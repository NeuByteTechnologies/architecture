# ADR‑001: Database Choice — Azure MySQL
**Status**: Accepted
**Date**: 2026‑05‑29
**Owner**: Architecture & Systems Design

## 1. Context
The NeuByte Fitness App Platform requires a cloud‑ready, secure, and scalable relational database to support:  

- User authentication and MFA workflows
- Workout logging and program tracking
- Notification preferences
- Transactional OLTP workloads
- Analytical reporting via Power BI Embedded
- ETL pipelines from OLTP → DW (star schema)
- The database must integrate cleanly with:
- Python backend (fitness‑api)
- C# frontend (fitness‑web)
- Azure‑based analytics (Power BI Service / Fabric)
- Azure identity and networking

Multiple hosting models were evaluated, including shared hosting, VM‑hosted MySQL, on‑prem SQL Server/MySQL, and cloud‑managed databases across Azure, AWS, and GCP.

## 2. Decision
NeuByte will use Azure Database for MySQL – Flexible Server as the primary relational database for both OLTP and DW workloads.  

Schemas:  

- **dbo** — OLTP transactional schema
- **star** — analytical schema consumed by Power BI

This database is the system of record for all application data.

## 3. Rationale
Azure MySQL was selected based on three primary drivers:

### 3.1 Alignment With Certification and Professional Goals (Reason A & E)
The NeuByte platform is intentionally designed to reinforce:

- Azure certification preparation
- Azure architectural fluency
- Cloud‑native design patterns
- Professional branding as a cloud‑aware systems analyst

Using Azure MySQL ensures the portfolio demonstrates:

- Azure networking
- Azure identity integration
- Azure managed services
- Azure analytics (Power BI Embedded)
- Azure DevOps / CI/CD alignment

This strengthens both the architecture story and the candidate’s professional positioning.

### 3.2 Required Integration With Power BI Embedded (Reason C)
Power BI Embedded requires:

- Azure Fabric datasets
- Azure AD / Entra ID authentication
- Azure‑hosted data sources
- Azure‑based embed token generation

Using a non‑Azure database (GoDaddy, VM‑hosted MySQL, AWS RDS, etc.) would:

- Break dataset refresh pipelines
- Complicate authentication
- Increase latency
- Add custom networking and security layers
- Reduce reliability of embedded analytics

Azure MySQL ensures seamless integration with the analytics architecture.

### 3.3 Cloud‑Managed Reliability and Reduced Operational Overhead
Azure MySQL provides:

- Automated backups
- High availability
- Patch management
- Scaling options
- VNet integration
- Monitoring and telemetry
-
This eliminates the operational burden of:

- Managing a VM
- Maintaining OS patches
- Securing a self‑hosted database
- Handling failover manually

## 4. Alternatives Considered
### 4.1 Shared Hosted MySQL (GoDaddy, Bluehost, HostGator)
**Pros**:

- Very low cost
- Easy to provision

**Cons**:

- No VNet integration
- Weak security model
- No private networking
- No HA or scaling
- Not suitable for Power BI Embedded
- Not enterprise‑grade

**Reason Rejected:**  
Fails security, scalability, and analytics integration requirements.

### 4.2 Dedicated VM Running MySQL
**Pros**:

- Full control
- Flexible configuration

**Cons**:

- Requires OS patching
- Requires MySQL maintenance
- Requires manual backups
- No built‑in HA
- Higher operational overhead
- More complex networking

**Reason Rejected:**
Adds unnecessary DevOps burden and breaks the cloud‑native architecture.

### 4.3 On‑Prem SQL Server or MySQL
**Pros**:

- Full control
- Potentially lower long‑term cost

**Cons**:

- No cloud elasticity
- No native integration with Power BI Embedded
- Requires VPN or hybrid networking
- High operational overhead

**Reason Rejected**:  
Not compatible with the cloud‑native design or analytics requirements.

### 4.4 AWS RDS / GCP Cloud SQL
**Pros**:

- Mature managed database services
- Strong performance

**Cons**:

- Breaks Power BI Embedded integration
- Requires cross‑cloud networking
- Increases latency
- Complicates identity and security
- Misaligned with Azure certification goals

**Reason Rejected**:  
Fails analytics integration and professional alignment requirements.

### 4.5 Azure SQL Database
**Pros**:

- Deep Azure integration
- Strong T‑SQL support

**Cons**:

- Higher cost
- Overkill for current OLTP needs
- Less aligned with MySQL‑based prototypes

**Reason Rejected**:  
Cost and complexity outweigh benefits for this platform.

## 5. Consequences
**Positive**
- Seamless integration with Power BI Embedded
- Strong alignment with Azure certifications
- Reinforces cloud‑native architecture
- Reduced operational overhead
- Predictable scaling
- Mature MySQL ecosystem
- Clean integration with Python and C#

**Negative**
- Vendor lock‑in to Azure
- Requires careful indexing for performance
- Lacks some advanced PostgreSQL features

## 6. Related Documents
- DataArchitecture.md
- BackendArchitecture.md
- ComponentRelationships.md
-  SystemArchitecture.md
- Architecture‑Philosophy.md

## 7. Version History
| Version | Date | Author | Description |
| --- | --- | --- | --- |
| 1.0 | 2026‑05‑29 | Gordon | Updated ADR‑001 to reflect certification goals, Power BI integration, and professional alignment. |

