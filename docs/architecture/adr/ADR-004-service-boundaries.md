# ADR‑004: Service Boundaries
**Status**: Accepted
**Date**: 2026‑05‑29
**Owner**: Architecture & Systems Design

## 1. Context
The NeuByte Fitness App Platform uses a layered architecture consisting of:

- Controller Layer
- Service Layer
- Repository Layer
- Database Layer

The **Service Layer** is the core of the platform’s business logic.
It must be clearly defined to:  

- Prevent logic leakage into controllers
- Prevent data access leakage into controllers or other services
- Maintain separation of concerns
- Support testability
- Enable future scaling
- Ensure predictable data flow
- Align with Azure‑based authentication and analytics integrations

The platform includes several functional domains:

- Authentication
- User Management
- Profile Management
- Workout Logging
- Notifications
- Program Management

Each domain requires clear service boundaries to avoid coupling and ensure long‑term maintainability.

## 2. Decision
NeuByte will implement strict, domain‑aligned service boundaries, where each service owns a single functional domain and is the only layer allowed to perform business logic for that domain.

The primary services are:

**- Authentication Service**
**- User Management Service**
**- Profile Service**
**- Workout Service**
**- Notification Service**
**- Program Service**

Each service:

- Owns its domain’s business rules
- Coordinates with repositories
- Interacts with external systems (IdP, Power BI, Azure APIs)
- Exposes domain‑specific operations to controllers
- Does not access other services’ repositories directly

## 3. Rationale
### 3.1 Enforces Separation of Concerns
Service boundaries ensure:  

- Controllers remain thin
- Repositories remain data‑focused
- Business logic is centralized
- No cross‑layer shortcuts occur

This aligns with the architecture philosophy and BackendArchitecture.md.  

### 3.2 Supports Scalability and Future Microservice Evolution
Clear boundaries allow:  

- Independent scaling of high‑traffic domains
- Future extraction into microservices (if needed)
- Clean API Gateway routing
- Predictable domain ownership
- Workout logging, for example, may scale differently than notifications.

### 3.3 Improves Testability
Each service can be:  

- Unit tested independently
- Mocked in isolation
- Validated without database dependencies

This reduces regression risk and improves CI/CD reliability.  

### 3.4 Aligns With Azure Integration Requirements
Azure‑based components (IdP, Power BI, Azure APIs) require:  

- Predictable service entry points
- Clear authentication flows
- Consistent token handling
- Clean separation between operational and analytical logic

Service boundaries ensure these integrations remain stable.

### 3.5 Prevents Domain Coupling
Without strict boundaries, systems drift into:

- Shared logic
- Cross‑service dependencies
- Hidden coupling
- Hard‑to‑maintain workflows

Boundaries keep each domain clean and independent.

## 4. Alternatives Considered
4.1 Monolithic Service (Single Service Layer)
**Pros**:

- Simple to implement
- Fewer files

**Cons**:

- Logic becomes tangled
- Harder to scale
- Harder to test
- Harder to evolve
- Poor domain separation

**Reason Rejected**:  
Does not support long‑term maintainability or cloud‑native evolution.  

### 4.2 Microservices From Day One
**Pros**:

- Maximum isolation
- Independent deployment

**Cons**:

- High operational overhead
- Requires service mesh, distributed tracing, etc.
- Overkill for current project size
- Slows development

**Reason Rejected**:  
Premature complexity for a portfolio‑scale platform.  

### 4.3 Repository‑Driven Boundaries
**Pros**:

- Simple mapping to database tables

**Cons**:

- Encourages anemic services
- Pushes business logic into controllers
- Does not reflect real domain boundaries

**Reason Rejected**:  

## 5. Consequences
**Positive**
- Clean domain ownership
- Predictable data flow
- Easier onboarding
- Strong testability
- Supports future microservice evolution
- Aligns with Azure‑based integrations
- Reduces long‑term maintenance cost

**Negative**
- Requires discipline to maintain boundaries
- Some cross‑domain workflows require orchestration logic

## 6. Related Documents
**- Architecture‑Philosophy.md**
**- BackendArchitecture.md**
**- ComponentRelationships.md**
**- SystemArchitecture.md**
**- ADR‑002: Authentication Method**
**- ADR‑003: API Style**

## 7. Version History
| Version | Date | Author | Description |
| --- | --- | --- | --- |
| 1.0 | 2026‑05‑29 | Gordon | Initial creation of ADR‑004: Service Boundaries |