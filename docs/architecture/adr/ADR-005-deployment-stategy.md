# ADR‑005: Deployment Strategy
**Status**: Accepted
**Date**: 2026‑05‑29
**Owner**: Architecture & Systems Design

## 1. Context
The NeuByte Fitness App Platform requires a deployment strategy that is:  

- Cloud‑native
- Secure
- Scalable
- Cost‑efficient
- Easy to maintain
- Aligned with Azure certification goals
- Consistent with the platform’s architecture (API Gateway → Services → Database → Analytics)

The deployment approach must support:

- Python backend (fitness‑api)
- C# web frontend (fitness‑web)
- Azure MySQL database
- Power BI Embedded integration
- ETL pipelines
- Future mobile clients

The strategy must also reflect portfolio‑appropriate simplicity — avoiding enterprise‑scale complexity while still demonstrating professional‑grade architecture.

## 2. Decision
NeuByte will use a container‑based deployment strategy on Azure, with the following components:  

- Azure App Service for hosting the Python API
- Azure App Service for hosting the C# web frontend
- Azure API Management (APIM) as the API Gateway
- Azure MySQL Flexible Server as the database
- Azure Storage for static assets and logs
- Azure Key Vault for secrets
- Azure Monitor / Application Insights for telemetry

This approach provides a clean, cloud‑native deployment model without introducing unnecessary operational overhead.  

## 3. Rationale
### 3.1 Alignment With Azure Certification and Professional Goals
This deployment model reinforces:  

- Azure App Service
- Azure API Management
- Azure MySQL
- Azure Key Vault
- Azure Monitor
- Azure networking and identity

These are core components of Azure certification paths (AZ‑900, AZ‑204, AZ‑305), making the architecture directly supportive of your professional development.  

### 3.2 Cloud‑Native, Managed Infrastructure
Azure App Service provides:

- Automatic scaling
- Zero‑downtime deployments
- Built‑in HTTPS
- Managed runtime environments
- Simplified CI/CD integration

This avoids the operational burden of:

- Managing VMs
- Patching OS images
- Handling container orchestration manually

### 3.3 Seamless Integration With Power BI Embedded
Power BI Embedded requires:  

- Azure AD / Entra ID
- Azure‑hosted datasets
- Azure‑based token generation

Deploying the API and frontend inside Azure ensures:

- Low‑latency token generation
- Secure identity flows
- Consistent networking

### 3.4 Predictable, Portfolio‑Appropriate Complexity
This deployment strategy:

- Demonstrates real cloud architecture
- Avoids enterprise‑scale Kubernetes complexity
- Avoids unnecessary microservices
- Keeps the architecture clean and understandable

It shows senior‑level thinking without overwhelming the portfolio.

### 3.5 Supports Future Evolution
This model can evolve into:

- Azure Container Apps
- Azure Kubernetes Service (AKS)
- Multi‑region deployments
- CI/CD pipelines with GitHub Actions

But none of these are required today.

## 4. Alternatives Considered
### 4.1 Azure Kubernetes Service (AKS)
**Pros**:

- Maximum scalability
- Full microservice orchestration

**Cons**:

- High operational overhead
- Requires cluster management
- Overkill for current project size

**Reason Rejected**:  
Too complex for a portfolio‑scale platform.

### 4.2 VM‑Based Deployment
**Pros**:

- Full control
- Flexible configuration

**Cons**:

- Requires OS patching
- Requires manual scaling
- Higher security risk
- More DevOps overhead

**Reason Rejected**:  
Not cloud‑native and increases maintenance burden.

### 4.3 Serverless (Azure Functions)
**Pros**:

- Highly scalable
- Pay‑per‑execution

**Cons**:

- Cold starts
- Stateless constraints
- Poor fit for REST API with multiple domains

**Reason Rejected**:  
Not ideal for a multi‑domain API with consistent traffic patterns.

### 4.4 On‑Prem or Shared Hosting
**Pros**:

- Low cost
- Simple setup

**Cons**:

- No integration with Power BI Embedded
- No Azure identity integration
- No cloud scalability
- Not aligned with certification goals

**Reason Rejected**:  
Breaks analytics integration and professional alignment.

## 5. Consequences
**Positive**
- Clean, cloud‑native deployment
- Strong alignment with Azure certifications
- Low operational overhead
- Easy CI/CD integration
- Predictable scaling
- Secure identity and secret management
- Supports future evolution

**Negative**
- Vendor lock‑in to Azure
- App Service scaling is less flexible than AKS
- Requires careful cost monitoring

## 6. Related Documents
Architecture‑Philosophy.md
SystemArchitecture.md
BackendArchitecture.md
ComponentRelationships.md
DataArchitecture.md
ADR‑001: Database Choice
ADR‑002: Authentication Method
ADR‑003: API Style
ADR‑004: Service Boundaries

## 7. Version History
| Version | Date | Author | Description |
| --- | --- | --- | --- |
| 1.0 | 2026‑05‑29 | Gordon | Initial creation of ADR‑005: Deployment Strategy |

