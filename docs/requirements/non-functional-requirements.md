
# Non-Functional Requirements

## 1. Purpose

Define the quality attributes and operational constraints
for the clinic-system application.

## 2. Performance

- NFR-PER-001: Standard API requests should achieve p95
  response time below 500 ms under normal load.
- NFR-PER-002: Patient searches should achieve p95
  response time below 2 seconds.
- NFR-PER-003: Appointment calendar loading should
  achieve p95 response time below 2 seconds.

## 3. Availability and Reliability

- NFR-REL-001: Target 99.9% monthly availability.
- NFR-REL-002: Confirmed appointments must not be
  silently lost.
- NFR-REL-003: Application errors must use consistent
  error responses.
- NFR-REL-004: Database recovery procedures must be
  documented and tested.

## 4. Security and Privacy

- NFR-SEC-001: Protected operations require authentication.
- NFR-SEC-002: Authorization must be enforced by the backend.
- NFR-SEC-003: Apply least-privilege access.
- NFR-SEC-004: Encrypt data in transit and protect
  stored sensitive data.
- NFR-SEC-005: Secrets must not be committed to Git.
- NFR-SEC-006: Sensitive operations must be auditable.
- NFR-SEC-007: Sessions must have secure expiration behavior.

## 5. Maintainability

- NFR-MAI-001: Domain logic must not depend on frameworks.
- NFR-MAI-002: Organize business capabilities into modules.
- NFR-MAI-003: Use Ruff and mypy for code quality.
- NFR-MAI-004: Provide unit, integration, and E2E tests.
- NFR-MAI-005: Document significant architecture decisions.

## 6. Usability and Accessibility

- NFR-UX-001: Support desktop and tablet layouts.
- NFR-UX-002: Provide understandable validation messages.
- NFR-UX-003: Target WCAG 2.2 AA.
- NFR-UX-004: Minimize unnecessary steps in common workflows.

## 7. Operations and Observability

- NFR-OPS-001: Use structured application logging.
- NFR-OPS-002: Monitor API errors and response times.
- NFR-OPS-003: Provide application and database health checks.
- NFR-OPS-004: Define database backup and retention policies.
- NFR-OPS-005: Support reproducible deployments.

## 8. Initial Workload Assumptions

- One primary-care clinic.
- Multiple practitioners.
- Approximately 2,000 registered patients.
- Approximately 50 appointments per day.

These are planning assumptions and must be validated.

## 9. Validation Strategy

- Performance: Load testing.
- Security: Authorization and security tests.
- Reliability: Recovery and failure testing.
- Maintainability: Architecture tests and code reviews.
- Usability: Workflow testing.
- Observability: Log and monitoring verification.

## 10. Open Questions

- What is the required recovery time objective (RTO)?
- What is the required recovery point objective (RPO)?
- What are the required backup retention periods?
- What are the applicable privacy and regulatory requirements?
- Will the application be deployed on-premises or in the cloud?

