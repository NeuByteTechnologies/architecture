# Data Architecture  
NeuByte Fitness App Platform  
Version: 1.0  
Last Updated: 2026‑05‑20

---

## Overview
The NeuByte Fitness App Platform uses a dual‑schema data architecture designed to support both **transactional operations** and **analytical reporting**.  
This document describes how data is structured, how it flows through the system, and how the platform integrates with external analytical services such as Power BI.

The data architecture consists of:
- **Azure MySQL Database** (primary data store)
- **dbo schema** (OLTP transactional layer)
- **star schema** (DW analytical layer)
- **ETL/ELT pipelines** (movement from OLTP → DW)
- **Power BI Service** (analytics and embedded reporting)
- **Backend repositories** (data access abstraction)

This document provides a high‑level architectural view and does not define individual tables or fields.  
Those details are documented in the **Data Dictionary**.

---

## Data Architecture Goals
- Maintain transactional integrity for all user‑facing operations  
- Support analytical reporting and trend analysis  
- Enable Power BI Embedded for in‑app reporting  
- Separate OLTP and analytical workloads  
- Provide scalable, query‑optimized structures  
- Ensure clean data lineage from source → transform → analytics  

---

## Logical Data Architecture

### **1. OLTP Layer — dbo Schema**
The **dbo** schema contains normalized tables that support the operational needs of the platform.

Characteristics:
- Highly normalized  
- Optimized for frequent reads/writes  
- Used directly by backend services  
- Stores user profiles, preferences, workouts, notifications, and program data  

Examples of data domains:
- User & Authentication  
- Workout Logging  
- Weight Tracking  
- Exercise Programs  
- Notifications  
- Preferences  

This schema is the **source of truth** for all application transactions.

---

### **2. Data Warehouse Layer — star Schema**
The **star** schema contains denormalized tables optimized for analytics and reporting.

Characteristics:
- Fact tables store measurable events  
- Dimension tables store descriptive attributes  
- Optimized for aggregations and Power BI queries  
- Supports trend indicators and historical analysis  

Example structures:
- **FactWorkouts**  
- **FactWeightTracking**  
- **FactNotifications**  
- **DimUser**  
- **DimDate**  
- **DimExercise**  

This schema is consumed by the **Power BI Service** to generate embedded reports.

---

## Physical Data Architecture

### **Azure MySQL Database**
The platform uses a single Azure MySQL instance with two logical schemas:

| Schema | Purpose | Workload |
|--------|---------|----------|
| **dbo** | OLTP transactional operations | High‑frequency reads/writes |
| **star** | Analytical reporting | Aggregations, Power BI queries |

Additional physical considerations:
- Indexing strategy optimized for OLTP in dbo  
- Star schema optimized for scan‑heavy analytical workloads  
- Backup and restore policies managed at the database level  
- Data retention policies defined per domain (e.g., workout history)  

---

## ETL / ELT Pipeline

### **Purpose**
Move data from the OLTP schema to the DW schema for analytics and reporting.

### **Flow**
dbo (OLTP) → ETL/ELT → star (DW) → Power BI Service → Embedded Reports


### **Process Description**
- **Extract:** Data is read from dbo tables  
- **Transform:**  
  - Data cleansing  
  - Aggregation  
  - Dimensional modeling  
  - Trend indicator calculations  
- **Load:** Data is written into star schema fact and dimension tables  

### **Frequency**
- Scheduled batch (nightly or hourly)  
- Optional incremental loads for high‑value metrics  

### **Data Quality**
- Validation rules applied during transformation  
- Referential integrity enforced in DW  
- Slowly Changing Dimensions (SCD) applied where needed  

---

## Integration Points

### **Backend Services → dbo Schema**
All application operations (login, preferences, workouts, notifications) interact with the dbo schema through repository classes.

### **DW Schema → Power BI Service**
The star schema provides:
- Dataset models  
- Aggregated metrics  
- Trend indicators  
- Historical analysis  

Power BI Service refreshes datasets on a scheduled basis.

### **Power BI Embedded → NeuByte App**
The backend requests:
- Embed tokens  
- Report metadata  
- Secure access to Power BI reports  

The frontend renders embedded reports inside the app UI.

---

## Data Flow Diagrams

### **Level‑0 Data Flow**
User → NeuByte Platform → Azure MySQL → Power BI Service

### **Level‑1 Data Flow**
Controllers → Services → Repositories → dbo → ETL → star → Power BI → Embedded Reports

### **Level‑2 Data Flow (Example: Workout Logging)**

WorkoutLoggingController
→ WorkoutLoggingService
→ WorkoutRepository
→ dbo.WorkoutLog
→ ETL
→ FactWorkouts
→ Power BI Dataset
→ Embedded Report


---

## Diagram Reference
**Figure 1 — NeuByte Data Architecture Diagram**  
`/architecture/diagrams/data-architecture.svg`

A PNG export is also available for reviewer readability.

---

## Assumptions & Constraints
- OLTP and DW schemas reside in the same Azure MySQL instance  
- ETL processes do not impact transactional performance  
- Power BI consumes only DW data, not OLTP  
- No cross‑schema writes occur from the application  
- Data Dictionary defines all table‑level details  

---

## Revision History
| Version | Date       | Description |
|---------|------------|-------------|
| 1.0     | 2026‑05‑20 | Initial creation of Data Architecture Document |

