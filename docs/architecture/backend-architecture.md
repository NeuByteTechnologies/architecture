# Backend Architecture Breakdown for the Architecture Document
## 1. Overview  

The NeuByte Fitness App backend follows a layered architecture pattern derived from MVC but extended for enterprise scalability and maintainability.
Each layer has a distinct responsibility, ensuring separation of concerns and clear data flow from the user interface to the database.

---
Controller → Service → Repository → Database
---

## 2. Layer Descriptions
### Controller Layer (Presentation / API Layer)
- **Purpose**: Handles HTTP requests and responses.
- **Responsibility**:
 -  Receives requests from the frontend or mobile app.
 - Validates input and delegates business logic to the Service layer.
 - Returns standardized API responses (JSON).
- Artifacts: Your API Contracts (e.g., API-User.md, API-WorkoutLogging.md, etc.) define these endpoints.
- Example Components:
 - AuthController
 - UserController
 - WorkoutLoggingController
 - NotificationController

### Service Layer (Business Logic Layer)
- **Purpose**: Implements the core business rules and workflows.
- **Responsibility**:
 - Encapsulates logic independent of HTTP or database concerns.
 - Coordinates multiple repositories or external systems.
 - Ensures transactional integrity and validation.
- Example Components:
 - AuthService (handles MFA, token generation)
 - UserService (manages profiles, preferences)
 - WorkoutService (handles workout logging and analytics)
 -NotificationService (routes messages and preferences)

### Data Access Layer (Repository Layer)
- **Purpose**: Abstracts database operations.
- **Responsibility**:
 - Performs CRUD operations on the Azure MySQL database.
 - Maps domain models to relational tables.
 - Provides reusable data access methods for services.
- **Example Components:**
 - UserRepository
 - WorkoutRepository
 - ProgramRepository
 - NotificationRepository

### Database Layer
- **Purpose**: Persistent data storage.
- **Technology**: Azure MySQL Database
- **Structure**:
 - dbo schema for OLTP (application transactions)
 - star schema for DW (analytics and Power BI integration)
- **Responsibility**:
 -Stores normalized transactional data.
 - Provides views or ETL outputs for analytical consumption.

### External Systems
- **Authentication Provider (MFA)**: Handles secure multi‑factor authentication.
- **Azure APIs:** Provides cloud services and integrations.
- **Power BI Service (Azure Fabric)**: Consumes analytical data from the DW schema.

## 3. Data Flow Summary
**1. User Request** → Controller
**2. Controller**→ Service (business logic execution)
**3. Service** → Repository (data access)
**4. Repository** → Azure MySQL (read/write)
**5. Service** → External Systems (MFA, Power BI, Azure APIs)
**6. Response** → Controller → User

## 4. Design Principles
- **Separation of Concerns**: Each layer focuses on one responsibility.
- **Dependency Inversion**: Controllers depend on abstractions (interfaces) of services, not concrete implementations.
- **Scalability**: Each layer can be extended or replaced independently.
- **Testability**: Services and repositories can be unit‑tested without HTTP or database dependencies.
- **Maintainability**: Clear boundaries make future enhancements predictable.

## 5. Diagram Reference
Use the NeuByte Backend Architecture Diagram (SVG) as the visual reference for this section.
It should appear under the heading:

**Figure X: NeuByte Backend Architecture**