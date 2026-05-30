# Non‑Functional Requirements (NFRs)
*System‑Wide Quality Attributes for the NeuByte Fitness App*

This document defines the cross‑cutting, system‑level constraints that govern the design, implementation, and operation of the NeuByte Fitness App.
These requirements apply to all modules, including Authentication, Dashboard, Programs, Workout Logging, Notifications, Preferences, Weight Tracking, and Help.  

NFRs originate from:  

- Business Requirements Document (BRD)
- Architecture Decisions (ADRs)
- API Contracts
- UI/UX Standards
- Security and Compliance expectations

## 1. Security Requirements
### 1.1 Authentication & Session Security
- All authentication traffic must be transmitted over HTTPS.
- Passwords must be hashed using a secure, modern algorithm (bcrypt, Argon2).
- Session tokens must be:
 - HTTP‑only
 - Secure
 - Randomized
 - Expired after inactivity
- MFA must be supported using:
 - TOTP (Authenticator App)
 - Email OTP
- MFA codes must:
 - Expire after a defined time window
 - Be validated using standard TOTP algorithms
- Failed login attempts must be tracked and accounts temporarily locked after threshold.

### 1.2 Authorization
- All authenticated endpoints must validate session tokens.
- Users may only access resources they own (e.g., workout logs, weight entries, notifications).
- Unauthorized access attempts must be logged.

### 1.3 Data Protection
- Sensitive data (passwords, MFA secrets) must never be logged.
- User PII must be encrypted at rest where applicable.
- API must reject malformed or suspicious input (SQLi, XSS, injection attempts).

### 1.4 Audit Logging
- Authentication events (login, logout, MFA success/failure) must be logged.
- Program start/completion, workout completion, and profile updates must be logged.

## 2. Performance Requirements
### 2.1 API Response Times
- 95% of API requests should respond within < 300ms under normal load.
- Dashboard composite endpoint should respond within < 500ms.

### 2.2 Data Processing
- Weight trend calculations must complete within < 150ms.
- Notification unread count must compute in < 100ms.

### 2.3 UI Responsiveness
- All screens must load primary content within 1 second on broadband connections.
- Skeleton loaders must appear within 100ms for perceived responsiveness.

## 3. Reliability & Availability
### 3.1 Uptime
- The system should target 99.5% uptime for portfolio demonstration purposes.

### 3.2 Error Handling
- All API endpoints must return structured error responses.
- UI must display:
- Inline errors
- Screen‑level errors
- Retry actions

### 3.3 Idempotency
- Logout, marking notifications read, and deleting entries must be idempotent.

### 3.4 Session Reliability
- Sessions must persist across browser refreshes until expiration.
- Expired sessions must be rejected with clear messaging.

## 4. Scalability Requirements
### 4.1 Horizontal Scalability
- Backend must support horizontal scaling via stateless API design.
- Session tokens must not require server‑side storage.

### 4.2 Database Scalability
- Database must support:
 - Growth in workout logs
 - Growth in weight entries
 - Growth in notifications

- Queries must be indexed for:
 - user_id
 - created_at
 - program_id
 - workout_log_id

### 4.3 Caching
- Metadata (unit types, workout statuses, notification types) may be cached for 24 hours.

## 5. Maintainability Requirements
### 5.1 Code Structure
- API must follow modular service boundaries (ADR‑004).
- DTOs must be versioned with the API.
- API Contracts must remain synchronized with implementation.

### 5.2 Logging & Monitoring
- System must log:
 - Errors
 - Authentication events
 - Program/workout lifecycle events
 - Logs must be structured and queryable.

### 5.3 API Versioning
- Breaking changes must require a new API version.
- Deprecated endpoints must be documented.

## 6. Usability Requirements
### 6.1 Accessibility
- All UI screens must support:
 - Screen readers
 - ARIA roles
 - Keyboard navigation
 - Proper labeling
- Password fields must be masked.
- Error messages must be descriptive and actionable.

### 6.2 Consistency
- UI components must follow NeuByte design system.
- Terminology must match the global glossary.

### 6.3 Responsiveness
- Layouts must adapt to mobile, tablet, and desktop.

## 7. Observability Requirements
### 7.1 Metrics
System must expose metrics for:

- API latency
- Error rates
- Authentication failures
- Notification volume
- Workout logging volume

### 7.2 Health Checks
- API must expose a health endpoint for uptime monitoring.

## 8. Data Integrity Requirements
### 8.1 Transactional Consistency
- Workout logging operations must be atomic.
- Weight entries must not allow duplicates for the same date (optional BR).
- Program start/completion must update user_program consistently.

### 8.2 Referential Integrity
- All foreign keys must be validated:

- program_id
- workout_id
- exercise_id
- notification_type_id
- unit_type_id

## 9. Compliance Requirements
### 9.1 Privacy
- User data must not be shared with third parties.
- Only minimal PII is stored (email, optional phone).

### 9.2 Data Retention
- Logs and historical data must be retained for demonstration purposes.
- No auto‑purge unless explicitly configured.

## 10. Disaster Recovery Requirements
### 10.1 Backups
- Database backups must occur daily.
- Backups must be restorable within < 1 hour.

### 10.2 Recovery Objectives
- RPO (Recovery Point Objective): 24 hours
- RTO (Recovery Time Objective): 1 hour

## 11. Cross‑References
**BRD Sections Containing NFR‑Like Requirements**
- Login Flow (Security, Reliability)
- Dashboard (Performance, Reliability)
- Workout Logging (Performance, Data Integrity)
- Help (Accessibility, Reliability)
- Notifications (Performance, Reliability)

**ADRs Influencing NFRs**
- ADR‑001 Database Choice
- ADR‑002 Authentication Method
- ADR‑003 API Style
- ADR‑004 Service Boundaries
- ADR‑005 Deployment Strategy

## 12. Summary
These NFRs define the quality bar for the NeuByte Fitness App.
They ensure the system is:

- Secure
- Performant
- Reliable
- Scalable
- Maintainable
- Accessible
- Observable
- Compliant

…and aligned with NeuByte’s engineering standards.