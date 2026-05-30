# ADR-002: Authentication Method

| Field            | Value                                      |
|------------------|--------------------------------------------|
| **ADR Number**   | ADR-002                                    |
| **Title**        | Authentication Method                      |
| **Status**       | Accepted                                   |
| **Date**         | 2026-05-29                                 |
| **Author(s)**    | NeuByte Architecture Team                  |
| **Reviewed By**  | Engineering Lead, Security Lead            |
| **Supersedes**   | N/A                                        |
| **Superseded By**| N/A                                        |
| **Related ADRs** | ADR-001 (Cloud Platform Selection)         |

---

## Table of Contents

1. [Context](#1-context)
2. [Decision](#2-decision)
3. [Rationale](#3-rationale)
4. [Alternatives Considered](#4-alternatives-considered)
5. [Consequences](#5-consequences)
6. [Implementation Notes](#6-implementation-notes)
7. [Compliance & Security Considerations](#7-compliance--security-considerations)
8. [Version History](#8-version-history)

---

## 1. Context

NeuByte is a portfolio-focused analytics platform built on Azure, serving enterprise clients who require secure, auditable, and scalable access to embedded business intelligence dashboards. The platform integrates **Power BI Embedded** to deliver white-labeled reporting experiences and connects to multiple organizational data sources across tenant boundaries.

As the platform scales, several authentication pressures have emerged:

- **Multi-tenancy**: Each portfolio client operates under its own organizational identity boundary. Authentication must enforce strict tenant isolation without degrading the user experience.
- **Power BI Embedded requirements**: Embedding Power BI content requires a validated Azure identity context. Token generation for embedded content must be tied to an authenticated, authorized principal — either a delegated user token or a service principal backed by Microsoft Entra ID.
- **Regulatory compliance**: Enterprise clients operate in regulated sectors (finance, healthcare, professional services) where MFA enforcement, audit logging, and identity federation are contractual or statutory requirements.
- **Identity federation**: Many client organizations manage their own Identity Providers (IdPs) — including Okta, Azure Active Directory (Entra ID), PingIdentity, and ADFS — and expect SSO to work transparently with their existing credential stores.
- **Developer and service-to-service access**: Background jobs, data refresh pipelines, and scheduled report generation require non-interactive authentication that does not depend on a human user session.
- **Portfolio architecture alignment**: NeuByte's architecture favors Azure-native services to minimize operational overhead, capitalize on platform SLAs, and leverage built-in compliance controls.

A clear, consistent authentication decision is required to govern all access patterns: end-user interactive login, IdP-federated SSO, Power BI Embedded token issuance, and automated service-to-service calls.

---

## 2. Decision

**NeuByte will adopt Microsoft Entra ID (formerly Azure Active Directory) as its Identity Platform, with Multi-Factor Authentication (MFA) enforced through Conditional Access Policies, federated SSO via SAML 2.0 / OIDC for external IdPs, and service principal authentication for Power BI Embedded and automated workloads.**

The authentication architecture consists of four interlocking layers:

### 2.1 Interactive User Authentication — Microsoft Entra ID + MFA

All interactive user sessions are authenticated through **Microsoft Entra ID** using **OpenID Connect (OIDC)** over **OAuth 2.0**. MFA is enforced universally via **Entra ID Conditional Access Policies**, requiring a second factor (Microsoft Authenticator, TOTP, or FIDO2 hardware key) for all users, with no permanent exemptions.

Conditional Access policies are scoped by:

- **User risk level** — step-up authentication required for risky sign-ins.
- **Device compliance** — Intune-enrolled or compliant devices receive reduced friction; unmanaged devices trigger additional verification.
- **Network location** — named trusted networks may reduce prompt frequency; all external access enforces MFA unconditionally.
- **Application sensitivity** — administrative console access triggers a separate, more stringent Conditional Access policy.

### 2.2 Federated SSO — External IdP Integration

Client organizations with existing enterprise IdPs are onboarded via **SAML 2.0** or **OIDC federation**, configured as **external identity providers** in Entra ID using **Microsoft Entra External ID** (B2B federation). This allows client users to authenticate against their own corporate IdP (Okta, ADFS, PingIdentity, Google Workspace, etc.) while Entra ID remains the authoritative token issuer for all NeuByte services.

Federation configuration per tenant includes:

- Metadata exchange and certificate pinning at onboarding.
- Attribute mapping from IdP claims to NeuByte role claims (`neubyte_role`, `tenant_id`, `portfolio_scope`).
- MFA satisfaction acknowledgment — client-asserted MFA is accepted if the IdP returns an `amr` claim containing `mfa`; otherwise, Entra Conditional Access enforces a step-up challenge.

### 2.3 Power BI Embedded — Service Principal Authentication

Power BI Embedded content (dashboards, reports, datasets) is served using the **"App Owns Data"** embedding model. Token generation for embedded reports is handled by a **Microsoft Entra registered application** authenticating as a **Service Principal** using a **client certificate** (preferred) or client secret (fallback, with mandatory 90-day rotation).

The embedding flow is as follows:

User authenticates via Entra ID (OIDC)
│
▼
NeuByte API validates user session and resolves tenant + portfolio scope
│
▼
Backend calls Power BI REST API using Service Principal token
(GenerateTokenRequest with portfolio-scoped RLS identity)
│
▼
Embed token returned to frontend — scoped to specific report + RLS role
│
▼
Power BI JavaScript SDK renders report within NeuByte UI


Row-Level Security (RLS) rules are applied at the dataset level, parameterized by `tenant_id` and `portfolio_scope` claims resolved from the authenticated user session. No user-level credentials are ever passed to the Power BI service; all dataset access flows through the service principal with RLS filtering as the isolation boundary.

### 2.4 Service-to-Service Authentication — Managed Identities

Internal Azure services (Azure Functions, App Service, Azure Data Factory pipelines, Logic Apps) authenticate to one another and to Azure-managed resources (Key Vault, Storage, SQL, Service Bus) using **System-Assigned Managed Identities**. No credentials, connection strings, or secrets are stored in application configuration or code for inter-service communication.

Where Managed Identities are not available (third-party integrations, external APIs), credentials are stored in **Azure Key Vault** and retrieved at runtime via Managed Identity, with access audited through Key Vault diagnostic logs.

---

## 3. Rationale

### 3.1 Entra ID as the Identity Platform

Microsoft Entra ID is the natural identity foundation for a portfolio built on Azure. It provides:

- **Native integration** with all Azure services used by NeuByte (App Service, Azure Functions, Azure SQL, Power BI, Key Vault, Monitor).
- **Mature federation support** for SAML 2.0 and OIDC, eliminating the need to build or maintain custom federation brokers.
- **Conditional Access** as a declarative, policy-driven MFA enforcement engine that is auditable, version-controlled, and enforced at the platform level rather than within application code.
- **Token standardization** — all Entra-issued tokens are JWT-based, allowing consistent validation across NeuByte services without per-integration token parsing logic.
- **Compliance posture** — Entra ID holds certifications relevant to NeuByte's enterprise clients: ISO 27001, SOC 2 Type II, FedRAMP Moderate, GDPR, and HIPAA BAA eligibility.

### 3.2 MFA via Conditional Access (Not Application-Level Enforcement)

Enforcing MFA through Entra ID Conditional Access — rather than within NeuByte application code — provides three key advantages:

1. **Consistency**: MFA cannot be bypassed by accessing a different NeuByte service endpoint. Policy applies to the token issuance layer, not the application layer.
2. **Auditability**: All MFA events are logged in Entra ID's sign-in logs, creating a single audit trail independent of application logging.
3. **Maintainability**: MFA policy updates (new factor types, emergency exclusions, risk adjustments) are applied centrally without code deployments.

### 3.3 Service Principal for Power BI Embedded ("App Owns Data")

The "App Owns Data" model was chosen over "User Owns Data" for Power BI Embedded because:

- It eliminates the requirement for every NeuByte end user to hold a Power BI Pro or Premium Per User license.
- It allows NeuByte to control and audit all Power BI API calls through a single, monitored service identity.
- RLS at the dataset level provides portfolio-grade data isolation that is easier to audit and test than workspace-level user permission assignments.
- Service principal token lifetimes are configurable independently of user session lifetimes, allowing embed tokens to be scoped with appropriate expiry for the UX context.

### 3.4 Managed Identities for Internal Services

Managed Identities eliminate the credential management burden for internal Azure service communication. Secret rotation, expiry monitoring, and credential storage are handled by the Azure platform. This reduces the attack surface for credential theft and removes a class of operational incidents (expired secrets causing production outages).

---

## 4. Alternatives Considered

### 4.1 Custom Identity Server (e.g., IdentityServer / Duende)

**Description**: Deploy a self-hosted OpenID Connect / OAuth 2.0 authorization server (e.g., Duende IdentityServer) within the NeuByte infrastructure.

**Pros**:
- Full control over token structure, lifetimes, and issuance logic.
- No dependency on Microsoft's pricing or availability.
- Can be customized to support highly bespoke claim schemas.

**Cons**:
- High operational overhead: certificate management, high-availability deployment, patching, and incident response become the NeuByte team's responsibility.
- Federation with external enterprise IdPs requires significant custom development.
- Does not integrate natively with Power BI Embedded — embed token generation still requires an Entra-registered service principal, making a custom IdP redundant for this use case.
- Certification and compliance attestation must be independently achieved rather than inherited from Microsoft's compliance posture.

**Decision**: Rejected. The operational cost and integration complexity outweigh the benefits of ownership for NeuByte's current scale and team size.

---

### 4.2 Auth0 / Okta Customer Identity Cloud

**Description**: Use a third-party identity-as-a-service provider (Auth0, Okta CIC) as the primary IdP for NeuByte.

**Pros**:
- Excellent developer experience and SDKs.
- Strong built-in MFA and federation support.
- Vendor-managed infrastructure.

**Cons**:
- Adds a third-party dependency in the critical authentication path; outages at Auth0/Okta directly impact NeuByte sign-in availability.
- Power BI Embedded still requires Entra ID service principal registration, meaning two identity systems must be maintained in parallel.
- Additional licensing cost per monthly active user, which scales unfavorably for enterprise multi-tenant deployments.
- Audit logs and compliance reports are split across two identity systems, complicating incident investigation.

**Decision**: Rejected. The requirement for an Entra-registered service principal for Power BI Embedded means Entra ID cannot be fully replaced; maintaining two identity systems in parallel introduces unnecessary complexity.

---

### 4.3 Azure AD B2C (Business-to-Consumer)

**Description**: Use Azure AD B2C as the identity foundation, leveraging its consumer-identity features for client onboarding.

**Pros**:
- Designed for external-facing, multi-tenant consumer applications.
- Supports custom user journeys and branded login experiences.
- Supports social identity providers (Google, Facebook, Apple).

**Cons**:
- B2C is optimized for consumer identity, not enterprise federation. Enterprise SAML federation setup in B2C is more complex and limited than in Entra External ID.
- Conditional Access integration is available but less mature than in Entra ID P2.
- Power BI Embedded service principal registration requires a standard Entra ID tenant, not a B2C tenant.
- Portfolio clients expect enterprise-grade SSO experiences, not consumer-oriented sign-up flows.

**Decision**: Rejected. Entra External ID (B2B federation model) better fits the enterprise multi-tenant profile of NeuByte's client base and aligns with the existing Entra ID investment.

---

### 4.4 Application-Level MFA (Custom TOTP Integration)

**Description**: Implement MFA directly within NeuByte's application layer using a TOTP library and custom enrollment flow, independent of the identity platform.

**Pros**:
- Full control over the MFA user experience.
- No external dependency for MFA enforcement.

**Cons**:
- MFA enforcement is application-specific — a bypass at the network or API layer would circumvent protection entirely.
- TOTP secret storage and enrollment become security-critical responsibilities of the NeuByte team.
- Does not satisfy enterprise client requirements for IdP-side MFA enforcement and `amr` claim assertion.
- Audit trail is application-log dependent rather than platform-level, reducing auditability for compliance purposes.

**Decision**: Rejected. Platform-level MFA enforcement via Entra Conditional Access provides stronger guarantees with less implementation risk.

---

## 5. Consequences

### 5.1 Positive Consequences

- **Unified audit trail**: All authentication events — interactive, federated, and service-to-service — are logged in Entra ID sign-in and audit logs, providing a single pane of glass for security monitoring and compliance reporting.
- **Reduced credential surface**: Managed Identities and service principal certificates eliminate static secrets from application code and configuration for the majority of internal communication patterns.
- **Scalable federation**: New enterprise clients can be onboarded via IdP federation without changes to NeuByte application code — federation is a configuration operation in Entra.
- **Power BI alignment**: Service principal authentication with RLS satisfies Microsoft's recommended production architecture for Power BI Embedded, ensuring supportability and access to future Power BI platform capabilities.
- **Inherited compliance**: NeuByte inherits Entra ID's compliance certifications for the authentication layer, simplifying enterprise security reviews and client due diligence questionnaires.
- **MFA resilience**: Conditional Access policies survive application deployments and code changes — MFA enforcement is not a feature that can be accidentally removed in a code change.

### 5.2 Negative Consequences / Trade-offs

- **Entra ID dependency**: NeuByte's authentication availability is coupled to Microsoft Entra ID availability. Service disruptions at Microsoft affect user sign-in capability. Mitigation: token caching with appropriate lifetimes reduces re-authentication frequency; Entra ID's SLA (99.99% for premium tier) is acceptable for the platform's availability targets.
- **Service principal secret management**: Even with certificate-preferred authentication, the service principal registration requires lifecycle management (certificate renewal, emergency rotation procedures). Mitigation: certificates are stored in Key Vault with automated renewal alerts and documented rotation runbooks.
- **Federation onboarding complexity**: Each new enterprise client requiring IdP federation requires a metadata exchange and attribute mapping configuration step. This is an operational process, not a self-service flow. Mitigation: a standardized onboarding checklist and Terraform-based federation configuration template will be maintained.
- **Power BI Embedded capacity cost**: The "App Owns Data" model requires an Azure Power BI Embedded capacity (A-SKU or P-SKU) for production workloads. This is a fixed infrastructure cost that must be factored into portfolio client pricing. Mitigation: capacity can be paused during off-hours for cost optimization.
- **Entra ID P2 licensing requirement**: Conditional Access risk-based policies require Entra ID P2 licenses. This is an additional per-user cost for NeuByte's internal team and admin accounts. Mitigation: P2 licenses are scoped to admin and privileged accounts only; standard end users authenticate through federated IdPs and require only Entra External ID entitlements.

### 5.3 Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Entra ID regional outage | Low | High | Token caching; graceful degradation messaging |
| Service principal certificate expiry causing embed failures | Medium | High | Key Vault expiry alerts; automated renewal pipeline |
| Client IdP federation misconfiguration at onboarding | Medium | Medium | Standardized onboarding runbook; pre-production federation testing environment |
| RLS misconfiguration exposing cross-tenant data | Low | Critical | Automated RLS test suite in CI/CD pipeline; penetration testing per release |
| Conditional Access policy misconfiguration locking out users | Low | High | Break-glass admin account excluded from Conditional Access; change management review process |

---

## 6. Implementation Notes

### 6.1 Entra Application Registrations

| Application | Authentication Type | Token Audience | Notes |
|---|---|---|---|
| NeuByte Web App | OIDC / Authorization Code + PKCE | NeuByte API (`api://neubyte`) | User-facing; MFA enforced via Conditional Access |
| NeuByte API | Bearer token validation | `api://neubyte` | Validates Entra-issued JWTs on every request |
| Power BI Embed Service | Service Principal (certificate) | Power BI Service (`https://analysis.windows.net/powerbi/api`) | App Owns Data; RLS identity injected per request |
| Data Refresh Pipeline | Managed Identity | Azure SQL, Storage, Service Bus | No credentials; Managed Identity RBAC assignments |

### 6.2 Token Lifetime Configuration

| Token Type | Lifetime | Rationale |
|---|---|---|
| Access Token (user) | 60 minutes | Industry standard; balances security and UX |
| Refresh Token (user) | 14 days (rolling) | Persistent sessions for returning users |
| Power BI Embed Token | 60 minutes | Matches dashboard session; refreshed client-side before expiry |
| Service Principal Token | 60 minutes | Default Entra issuance; cached in-process |

### 6.3 Required Entra ID Roles and Permissions

- Power BI Embed Service Principal: Member of Entra security group assigned to Power BI workspace as **Member** role.
- Power BI Admin Settings: "Allow service principals to use Power BI APIs" enabled; scoped to the embed security group only.
- Key Vault: Embed service principal granted `Key Vault Certificate User` role on the certificates Key Vault.

### 6.4 Federation Onboarding Checklist (Per Client)

1. Receive client IdP metadata URL or XML file.
2. Register client domain in Entra External ID tenant.
3. Configure SAML 2.0 or OIDC federation with metadata.
4. Map client claims to NeuByte role schema (`neubyte_role`, `tenant_id`, `portfolio_scope`).
5. Validate `amr` claim assertion for MFA satisfaction; configure step-up Conditional Access rule if not asserted.
6. Test sign-in flow in pre-production environment with client test accounts.
7. Document federation configuration in NeuByte tenant registry.

---

## 7. Compliance & Security Considerations

- **GDPR**: User authentication events logged in Entra ID sign-in logs are processed within the configured Entra ID tenant region. Log retention is set to 90 days in Entra ID and exported to Azure Monitor Log Analytics for long-term retention per data residency requirements.
- **SOC 2 Type II**: Entra ID Conditional Access policy enforcement and MFA coverage are auditable controls satisfying SOC 2 CC6.1 (Logical and Physical Access Controls). Access reviews are scheduled quarterly via Entra ID Access Reviews.
- **Least Privilege**: Service principal permissions are scoped to the minimum Power BI API scopes required for embed token generation. No Global Administrator or Power BI Administrator roles are assigned to the service principal.
- **Break-Glass Access**: One break-glass administrator account is maintained, excluded from Conditional Access policies, stored in a hardware-secured vault, and subject to a mandatory access review any time it is used.
- **Penetration Testing**: Authentication flows, RLS enforcement, and token validation are included in the scope of the annual third-party penetration test.

---

## 8. Version History

| Version | Date       | Author                    | Status   | Changes                                      |
|---------|------------|---------------------------|----------|----------------------------------------------|
| 0.1     | 2026-04-10 | NeuByte Architecture Team | Draft    | Initial draft for team review                |
| 0.2     | 2026-04-24 | NeuByte Architecture Team | Draft    | Added Power BI Embedded section; expanded alternatives |
| 0.3     | 2026-05-08 | NeuByte Architecture Team | Proposed | Incorporated Security Lead review feedback; added RLS risk row |
| 1.0     | 2026-05-29 | NeuByte Architecture Team | Accepted | Approved by Engineering Lead and Security Lead; baselined |

---

*This document is maintained in the NeuByte architecture decision log. Questions and amendment proposals should be raised as pull requests against the ADR repository and reviewed by the Architecture and Security leads.*
