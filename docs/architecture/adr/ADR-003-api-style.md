# ADR‑003: API Style — RESTful JSON API
**Status**: Accepted
**Date**: 2026‑05‑29
**Owner**: Architecture & Systems Design

## 1. Context
The NeuByte Fitness App Platform requires a consistent, scalable, and well‑understood API style to support:  

- Web frontend (fitness‑web)
- Mobile clients (future scope)
- External integrations
- Authentication flows via IdP
- Power BI Embedded token generation
- Backend services written in Python

The API must be:

- Easy to consume
- Easy to document
- Easy to test
- Compatible with Azure API Gateway
- Aligned with industry standards
- Suitable for long‑term evolution

Several API styles were evaluated, including REST, GraphQL, gRPC, and RPC‑style endpoints.

## 2. Decision
NeuByte will use a **RESTful JSON API** as the primary API style for all client‑facing and service‑facing endpoints.  

Key characteristics:  

- Resource‑oriented URL structure
- Standard HTTP verbs (GET, POST, PUT, DELETE)
- JSON request/response bodies
- Consistent error handling
- Versioned endpoints (e.g., /api/v1/users)
- Stateless interactions

## 3. Rationale
### 3.1 Industry Standard and Developer Familiarity
REST is widely understood and supported across:  

- Python frameworks
- C# frontend integrations
- API testing tools
- Azure API Management
- Documentation tools (OpenAPI/Swagger)
- This reduces onboarding time and increases maintainability.

### 3.2 Strong Fit for the Platform’s Workflows
The platform’s operations — user management, workout logging, preferences, notifications — map naturally to REST resources:  

- `/users`
- `/workouts`
- `/preferences`
- `/notifications'`

REST provides a clean, predictable structure for these domains.  

### 3.3 Seamless Integration With Azure API Gateway
Azure API Management and API Gateway natively support:

- REST routing
- Rate limiting
- Authentication enforcement
- Logging and telemetry
- Versioning
- Policy injection

REST aligns perfectly with the platform’s cloud architecture.  

### 3.4 JSON as the Universal Data Format
JSON is:  

- Lightweight
- Human‑readable
- Supported by all client platforms
- Ideal for mobile and web performance
- Compatible with Power BI token flows

This ensures consistent serialization across services.

### 3.5 Supports Future Evolution
REST does not prevent:  

- Adding GraphQL for analytics
- Adding gRPC for internal microservices
- Adding WebSockets for real‑time features

REST is the stable foundation; other styles can be layered on later if needed.

## 4. Alternatives Considered
### 4.1 GraphQL
**Pros**:

- Flexible queries
- Reduces over‑fetching

**Cons**:

- Higher complexity
- Requires GraphQL server infrastructure
- Not necessary for current CRUD‑centric workflows
- Adds overhead for simple mobile/web clients

**Reason Rejected**:  
Overkill for the platform’s operational needs.

### 4.2 gRPC
**Pros**:

- High performance
- Strong for service‑to‑service communication

**Cons**:

- Not browser‑friendly
- Requires protobuf tooling
- Poor fit for public APIs
- Adds complexity to Python/C# integration

**Reason Rejected**:  
Better suited for internal microservices, not client‑facing APIs.

### 4.3 RPC‑Style Endpoints
**Pros**:

- Simple to implement

**Cons**:

- Harder to document
- Harder to scale
- Poor alignment with RESTful patterns
- Not future‑proof

**Reason Rejected**:  
Lacks structure and long‑term maintainability.  

## 5. Consequences
**Positive**
- Predictable, resource‑oriented API
- Easy integration with Azure API Gateway
- Strong developer familiarity
- Clean documentation via OpenAPI
- Consistent JSON serialization
- Easy testing and automation

**Negative**
- Less flexible than GraphQL for complex queries
- Requires careful versioning to avoid breaking changes

## 6. Related Documents
**- Architecture‑Philosophy.md**
**- BackendArchitecture.md**
**- ComponentRelationships.md**
**- SystemArchitecture.md**
**- ADR‑002: Authentication Method**

## 7. Version History
| Version | Date | Author | Description |
| --- | --- | --- | --- |
| 1.0 | 2026‑05‑29 | Gordon | Initial creation of ADR‑003: API Style |

