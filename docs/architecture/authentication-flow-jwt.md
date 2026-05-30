Authentication Flow (JWT)
NeuByte Fitness App Platform
Version: 1.0
Last Updated: 2026‑05‑29

## 1. Purpose
This document defines the **JWT‑based authentication flow** used by the NeuByte Fitness App Platform.
It describes how users authenticate, how tokens are issued and validated, and how the system enforces secure access across all backend services.

This is a **documentation artifact**, not a diagram.
Diagrams (sequence, flow, DFD) may accompany this document but are not required.

## 2. Overview
NeuByte uses a stateless authentication model based on JSON Web Tokens (JWT).
After a user successfully completes:

    1. CAPTCHA
    2. Username + Password
    3. MFA challenge

…the backend issues a **signed JWT access token** that the client uses for all subsequent API calls.  

The JWT replaces the older concept of a “session token” and provides:  

- Stateless authentication
- Strong cryptographic verification
- Clear expiration rules
- Role/permission claims
- Compatibility with API Gateway enforcement

## 3. Actors
**User**
Initiates login and interacts with the application.   

**Client Application (Web or Mobile)**  

- Sends login requests  
- Stores the JWT
- Sends JWT on each API call

**Authentication Service**  

- Validates credentials  
- Orchestrates MFA
- Issues JWT access tokens
- Issues optional refresh tokens

**Identity Provider (IdP)**
- Performs MFA verification
- Validates user identity
- Returns identity confirmation

**Token Validator (API Gateway + Backend Services)**
- Validates JWT signature
- Validates claims
- Enforces expiration
- Enforces authorization

## 4. Authentication Flow (Step‑by‑Step)
### 4.1 CAPTCHA Verification

- User submits CAPTCHA challenge
- Backend verifies human interaction
- Prevents automated credential stuffing

### 4.2 Credential Submission
- User submits username + password
- Authentication Service validates credentials
- If valid → proceed to MFA
- If invalid → return error

### 4.3 MFA Challenge
- Authentication Service requests MFA verification from IdP
- User submits OTP or authenticator code
- IdP confirms identity
- Authentication Service receives MFA success

### 4.4 JWT Issuance
Once MFA succeeds, the Authentication Service generates:

- **Access Token (JWT)**
- **Refresh Token** (optional, depending on policy)

The JWT includes:

- `sub` (user ID)
- `iat` (issued at)
- `exp` (expiration)
- `roles`
- `permissions`
- `tenant` (optional)

The token is signed using a secure algorithm (e.g., RS256).

### 4.5 Token Storage
Client stores the JWT in:

- In‑memory storage (preferred)
- Secure storage (mobile)

**Never** in localStorage if avoidable.

### 4.6 Authenticated Requests
For every API call:

---

Authorization: Bearer <jwt>

---

API Gateway and backend services:  

- Validate signature
- Validate expiration
- Validate claims
- Authorize access

### 4.7 Token Expiration
When `exp` is reached:

- Token becomes invalid
- API returns 401 Unauthorized

### 4.8 Token Refresh (Optional)
If refresh tokens are enabled:

Client sends refresh token to /auth/refresh

- Authentication Service issues a new access token
- Refresh token rotation is enforced

### 4.9 Logout
Logout is client‑side:

Client deletes JWT

Refresh token is invalidated server‑side (if used)

## 5. JWT Structure
### 5.1 Header
Specifies algorithm and token type:  

---

json
{
  "alg": "RS256",
  "typ": "JWT"
}

---

### 5.2 Payload (Claims)
Contains identity and authorization data:

`sub` — User ID

`iat `— Issued at

`exp` — Expiration

`roles` — User roles

`permissions` — Fine‑grained access

`tenant` — Optional multi‑tenant support

### 5.3 Signature
Ensures token integrity:

---

HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)

---

## 6. Security Considerations
- All communication uses TLS 1.2+
- JWTs contain no sensitive data
- Short expiration windows reduce risk
- Refresh tokens stored securely
- RS256 signing prevents tampering
- API Gateway enforces authentication
- Services validate claims independently

## 7. Error Conditions
**Expired Token**
- API returns 401
- Client must refresh or reauthenticate

**Invalid Signature**
- Token rejected
- Possible tampering or misconfiguration

**Missing Token**
- API returns 401
- Client must include Authorization header

**Malformed Token**
- Token rejected
- Client must request a new token

## 8. Related Artifacts  

**- BackendArchitecture.md**
**-ComponentRelationships.md**
**-SystemArchitecture.md**
**-DataArchitecture.md**
**-ADR‑002: Authentication Method**
**-API‑Auth.md (if applicable)**
**-Sequence Diagram: JWT Authentication (optional)**

## 9. Version History
| Version | Date | Author | Description |
| --- | --- | --- | --- |
| 1.0 | 2026‑05‑29 | Gordon | Initial creation of Authentication Flow (JWT) documentation. |