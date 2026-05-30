# System Context Diagram  
NeuByte Fitness App Platform  
Version: 1.0  
Last Updated: 2026‑05‑20

---

## Overview
The System Context Diagram defines the **highest‑level view** of the NeuByte Fitness App Platform.  
It shows the system as a single black‑box application and illustrates how external actors and external systems interact with it.  
This is a Level‑0 diagram following enterprise architecture standards (C4 Model: System Context).

The purpose of this document is to clearly communicate:
- What is **inside** the system boundary  
- What is **outside** the system boundary  
- How the system interacts with users and external services  
- Which integrations the platform depends on  

This diagram does **not** describe internal components, APIs, schemas, or data structures.  
Those are covered in the Backend Architecture, API Contracts, and Data Architecture documents.

---

## System Boundary
The NeuByte Fitness App Platform is represented as a **single system**:

**NeuByte Fitness App Platform (Hosted on Azure)**  
Includes:
- API‑driven backend  
- Power BI Embedded interface for in‑app reporting  
- Authentication, preferences, workout logging, notifications, and program management logic  

Internal components are intentionally abstracted at this level.

---

## External Actors
### **User (Mobile / Web)**
The primary human actor who interacts with the system through:
- Mobile app  
- Web application  

User interactions include:
- Login  
- Updating preferences  
- Viewing notifications  
- Accessing embedded reports  
- Logging workouts  
- Viewing program details  

---

## External Systems
### **Authentication Provider (MFA)**
Provides secure multi‑factor authentication services.  
The NeuByte platform delegates identity verification to this provider.

### **Azure APIs**
Represents Azure‑hosted services consumed by the platform, including:
- Cloud infrastructure services  
- Messaging  
- Storage  
- Auxiliary Azure components  

### **Azure MySQL Database**
Primary transactional data store for the platform.  
Stores all OLTP data (dbo schema) and provides data to the analytical layer (star schema).  
Internal schemas are not shown at this level.

### **Power BI Service (Azure Fabric)**
External analytics system that:
- Consumes data models from the DW schema  
- Provides embedded reports to the NeuByte app via Power BI Embedded  
- Issues embed tokens for secure report rendering  

---

## Data Flows (High‑Level)
- **User → NeuByte Platform**  
  Login, preferences, workout logging, notifications, report access  

- **NeuByte Platform → Authentication Provider**  
  MFA verification, token validation  

- **NeuByte Platform ↔ Azure APIs**  
  Cloud service interactions  

- **NeuByte Platform ↔ Azure MySQL Database**  
  Read/write transactional data  

- **Azure MySQL Database → Power BI Service**  
  Dataset / report data model  

- **NeuByte Platform → Power BI Service**  
  Embed tokens / report access  

---

## Diagram
The following diagram represents the complete System Context for the NeuByte Fitness App Platform:

**Figure 1 — NeuByte System Context Diagram (Level‑0)**  
`/architecture/diagrams/system-context.svg`

(Exported PNG version also available for reviewers.)

---

## Assumptions & Constraints
- The platform relies on external MFA for identity verification.  
- Power BI Embedded is integrated into the application UI but the **Power BI Service** remains an external system.  
- Azure MySQL is treated as a single external system; internal schemas are documented separately.  
- No internal components or APIs are shown at this level.  
- This diagram follows C4 Model conventions for System Context.

---

## Revision History
| Version | Date       | Description |
|---------|------------|-------------|
| 1.0     | 2026‑05‑20 | Initial creation of System Context Document |

