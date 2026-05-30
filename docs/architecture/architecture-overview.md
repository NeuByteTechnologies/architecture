---
layout: default
title: Architecture Overview
description: High‑level overview of the NeuByte Fitness App system architecture, documentation structure, and cross‑repo relationships.
---

# Architecture Overview  
*NeuByte Fitness App — System Architecture Summary*

This document provides a high‑level overview of the NeuByte Fitness App architecture.  
It explains the system’s purpose, major components, data flows, and how the architecture documentation is organized across the repository.

The goal of this overview is to give readers a clear understanding of **how the system works**, **how it is structured**, and **where to find deeper technical details**.

---

# 1. System Purpose

The NeuByte Fitness App is a full‑stack fitness‑tracking platform designed to demonstrate NeuByte’s end‑to‑end execution of the Software Development Life Cycle (SDLC).  
The system enables users to:

- Authenticate securely using CAPTCHA, credentials, and MFA  
- Select and manage exercise programs  
- Log workouts with detailed performance metrics  
- Track body weight and personal progress  
- View dashboards, insights, and reports  
- Receive system notifications  
- Access help and support content  

The architecture emphasizes clarity, modularity, scalability, and maintainability.

---

# 2. High‑Level Architecture

The system is composed of three major subsystems:

### **1. Frontend (fitness-app repo)**
- Static web application (HTML/CSS/JS or future C#/Blazor)
- Implements UI workflows, navigation, and user interactions
- Communicates with the API via HTTPS
- Hosted via GitHub Pages

### **2. Backend API (fitness-app-api repo)**
- Stateless REST API
- Handles authentication, program management, workout logging, notifications, and weight tracking
- Implements business logic and validation
- Exposes structured DTOs and API Contracts
- Uses a cloud-hosted database for persistence

### **3. Architecture Documentation (architecture repo)**
- System context and architecture diagrams  
- Data architecture and metadata definitions  
- ADRs (Architecture Decision Records)  
- NFRs (Non‑Functional Requirements)  
- Use cases and business requirements  
- Glossary (authoritative domain vocabulary)  
- Standards and design principles  

These three repos form the complete system.

---

# 3. Cross‑Repo Architecture Map

+-------------------------+        +---------------------------+
|     fitness-app         | <----> |     fitness-app-api       |
|  (Frontend/UI Layer)    |  HTTPS |   (Backend REST API)      |
+-------------------------+        +---------------------------+
^                               |
|                               |
|                               v
|                   +---------------------------+
|                   |   Cloud Database Layer    |
|                   +---------------------------+
|
v
+-------------------------------------------------------------+
|                 architecture (this repo)                    |
|  System Architecture • Data Models • ADRs • NFRs • Glossary |
+-------------------------------------------------------------+


The architecture repo documents the system; it does **not** contain code.

---

# 4. Major Architectural Components

### **4.1 Authentication & Security**
- CAPTCHA → Credentials → MFA  
- Session tokens (HTTP‑only, secure)  
- Password hashing  
- MFA via TOTP or Email OTP  
- Account lockout and rate limiting  

### **4.2 Program & Workout Engine**
- Program metadata  
- Daily workout structure  
- Exercise definitions  
- Workout logging  
- Historical performance  

### **4.3 Weight Tracking**
- Daily weight entries  
- Trend calculations  
- Insights and summaries  

### **4.4 Notifications**
- Unread/read state  
- Notification types  
- Notification center  

### **4.5 Dashboard**
- Composite data load  
- Recent activity  
- Progress summaries  
- Trends and insights  

### **4.6 Reporting**
- Weekly, monthly, annual summaries  
- Power BI integration (conceptual)  

---

# 5. Documentation Structure

The architecture repo is organized into the following sections:

### **5.1 System Architecture**
- `system-context.md`  
- `system-architecture.md`  
- `backend-architecture.md`  
- `component-relationships.md`  
- Architecture diagrams (SVG)

### **5.2 Data Architecture**
- `data-architecture.md`  
- Metadata definitions  
- ERD diagrams  

### **5.3 Architecture Decisions**
- `adr/ADR-001-database-choice.md`  
- `adr/ADR-002-authentication-method.md`  
- `adr/ADR-003-api-style.md`  
- `adr/ADR-004-service-boundaries.md`  
- `adr/ADR-005-deployment-strategy.md`  

### **5.4 Requirements**
- Business Requirements (BRD)  
- Use Cases  
- NFRs (`nfrs.md`)  

### **5.5 Glossary**
- `glossary.md` — authoritative domain vocabulary  

### **5.6 Standards**
- UI standards  
- Accessibility standards  
- Coding and documentation conventions  

---

# 6. Key System Flows

The following sequence diagrams describe runtime behavior:

- Login + MFA Flow  
- Dashboard Composite Load  
- Program Start → Workout → Completion  
- Workout Logging Flow  

(See `/docs/diagrams/` for SVGs.)

---

# 7. How to Navigate This Documentation

If you are new to the system, follow this recommended path:

1. **Start with System Context**  
2. Review **System Architecture**  
3. Explore **Data Architecture**  
4. Read the **ADRs** to understand design decisions  
5. Review **NFRs** for system‑wide constraints  
6. Use the **Glossary** to understand domain terms  
7. Dive into **Use Cases** and **BRD** for functional detail  

This layered approach mirrors real‑world architecture review processes.

---

# 8. Related Repositories

| Repository | Purpose |
|-----------|---------|
| **fitness-app** | Frontend UI, workflows, navigation, and user experience |
| **fitness-app-api** | Backend REST API, business logic, DTOs, and persistence |
| **architecture** | System documentation, diagrams, ADRs, NFRs, glossary |

---

# 9. Summary

This Architecture Overview provides the high‑level understanding needed to navigate the NeuByte Fitness App system.  
It connects the dots between:

- business requirements  
- architectural decisions  
- system components  
- data structures  
- runtime flows  
- cross‑repo relationships  

Use this document as your starting point for exploring the full architecture.