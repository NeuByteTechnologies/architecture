# NeuByte Repository Map
*System‑Level Architecture Overview*  

This document defines the repository structure for the NeuByte FitnessApp platform.  
It describes the purpose, boundaries, and relationships of each repository in the system.  

## 1. Purpose of This Document
The Repo Map provides:

- A high‑level architectural view of all repositories
- Clear domain boundaries
- Ownership and responsibility for each repo
- How repos interact and depend on each other
- A reference for onboarding, governance, and future expansion

This is a **design artifact**, not a governance policy.

## 2. Repository Overview (Top‑Level Map)
Code
NeuByte Platform
│
├── architecture
│     System design, diagrams, schema docs, repo map
│
├── standards
│     Repo governance, naming conventions, coding standards
│
├── fitness-api
│     Backend API implementation, migrations, DDL, business logic
│
├── fitness-web
│     Frontend web application (future C#/Blazor or JS)
│
├── data-engineering
│     Pipelines, ETL, analytics models, warehouse schemas
│
├── help-product
│     Jekyll-based help center, user guides, FAQs
│
├── resume
│     YAML-driven résumé system, Jekyll components
│
├── portfolio
│     Legacy documentation, static HTML pages
│
├── router
│     Shared assets, GitHub Pages root, global CSS/JS
│
└── .github
      Issue templates, workflows, org-wide automation  
This is the canonical structure of the NeuByte ecosystem.  

## 3. Repository Descriptions
### 3.1 architecture
**Purpose**:    
System design, diagrams, schema documentation, repo map, data models, API architecture.  

**Contains**:  

- System context diagrams
- Container diagrams
- ERDs
- MySQL Schema Design Document
- Repo Map (this document)
- Data flow diagrams
- Integration architecture

**Does NOT contain**:  

- Code
- Migrations
- Build scripts

### 3.2 standards
**Purpose**:  
Defines rules and conventions for all repos.  

**Contains**:  

- Repo governance
- Branching strategy
- Naming conventions
- Documentation standards
- Diagram grammar
- API standards
- Security standards
- Does NOT contain:
- Architecture diagrams
- System design decisions

### 3.3 fitness-api
**Purpose**:  
Backend implementation for the FitnessApp.

**Contains**:

- MySQL DDL
- Migrations
- API endpoints
- Business logic
- DTOs
- Unit tests
- CI/CD workflows
- Does NOT contain:
- Architecture docs
- Frontend code
- Portfolio content

### 3.4 fitness-web
**Purpose**:    
Frontend application for the FitnessApp.  

**Contains**:  

- UI components
- Pages
- CSS/JS/Blazor code
- API client
- Authentication flows

**Does NOT contain**:  

- Backend logic
- Database code

### 3.5 data-engineering
**Purpose**:    
Analytics, pipelines, and data transformations.  

**Contains**:  

- ETL/ELT pipelines
- Data warehouse schema
- Aggregations
- Reporting models
- Scheduled jobs

**Does NOT contain:**

- Transactional schema
- API code

### 3.6 help-product
**Purpose**:  
Public help center for the FitnessApp.

**Contains**:

- Jekyll site
- Help articles
- FAQ content
- Category structure

**Does NOT contain**:

- API documentation
- Architecture diagrams

### 3.7 resume
**Purpose**:  
- Gordon Neuls YAML‑driven résumé system.

**Contains**:

- YAML data files
- Jekyll includes
- Résumé templates
 - Skills, jobs, education components

### 3.8 portfolio
**Purpose**:    
- Legacy documentation and static HTML pages.

**Contains**:  
- Use cases  
- Static HTML  
- Early portfolio content  

**Eventually**:    
This repo becomes archival as the Architecture + Resume repos take over.  

### 3.9 router
**Purpose**:    
Shared GitHub Pages root for all public sites.  

**Contains**:  

- Global CSS
- Global JS
- Shared assets
- Redirect rules
- CNAME files

**Does NOT contain**:

- App code
- API code

### 3.10 .github
**Purpose**:  
- Org‑wide automation and templates.

**Contains**:

- Issue templates
- PR templates
- Actions workflows
- Security policies

## 4. Cross‑Repository Relationships
**Architecture → fitness-api**  
- Links to DDL
- Links to API design
- Links to schema diagrams
- Architecture → standards
- Architecture references standards for naming and conventions

**fitness-api → data-engineering**
- API emits events
- Data engineering consumes logs and workout data

**fitness-web → fitness-api**
= Frontend calls backend endpoints

**help-product → router**
- Help center is published through router Pages

**resume → router**
- Résumé site is published through router Pages

## 5. Domain Boundaries
- This section defines what each repo owns.  

**User Domain**
- API owns user logic
- Architecture owns user model
- Standards owns naming conventions

**Workout Domain**
- API owns workout logic
- Architecture owns schema
- Data engineering owns analytics

**Notification Domain**
- API owns delivery
- Architecture owns event model
- Help-product documents user-facing behavior

## 6. Design Rationale
- Why this structure works:

✔ Clear separation of concerns
Each repo has a single responsibility.

✔ Architecture is implementation‑agnostic
You can rewrite the API without touching architecture docs.

✔ Standards are policy, not design
No mixing of rules with architecture.

✔ Router centralizes public hosting
No duplication of CSS/JS across repos.

✔ API repo stays clean
No diagrams, no docs, no portfolio content.

✔ Recruiter‑friendly
Shows you understand enterprise‑scale repo organization.