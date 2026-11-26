# Domain Model

## 1. Purpose

Turn the vocabulary in [glossary.md](glossary.md) and
[ubiquitous-language.md](ubiquitous-language.md) into an initial domain model.
This document defines which concepts are modeled inside each bounded context,
how they relate, and where the boundaries lie.

This is a design document. No production domain code has been implemented yet.

## 2. Modeling Principles

- Use the ubiquitous language exactly; do not introduce storage or framework
  names into the model.
- Model behavior and invariants, not just data fields.
- An `Appointment` and an `Encounter` are separate aggregates with separate
  identities and lifecycles.
- Cross-context references use identifiers, not object graphs.
- A concept defined only by its values is a value object; a concept with
  continuity and identity is an entity.
- An aggregate root is the only entry point for changing its internal state.

See [entities.md](entities.md), [value-objects.md](value-objects.md), and
[domain-rules.md](domain-rules.md) for the detailed classification.

## 3. Bounded Contexts

| Context | Responsibility | Primary aggregate roots |
|---|---|---|
| Patient Management | Register and maintain patient information | `Patient` |
| Appointment Management | Reserve appointment slots and manage the appointment lifecycle | `Appointment` |
| Clinical Management | Record care delivered during an encounter | `Encounter` |
| Staff Management | Manage staff members, clinical roles, and availability | `StaffMember` |
| Billing and Payments | Invoice services and record money received | `Invoice`, `Payment` |

Future contexts not modeled in detail here: Medication, Laboratory,
Organization, User and Access Management, and Audit.

Boundaries protect meaning: `Doctor` in Staff Management describes the person
and availability; `doctor_id` in Appointment Management and Clinical
Management is only a reference to that person.

## 4. Aggregate Overview

```text
Patient Management        Appointment Management        Clinical Management
+----------------+        +--------------------+        +------------------+
| Patient (root) |<------ | Appointment (root) |<------ | Encounter (root) |
| Patient ID     |  id    | Appointment ID     |  id    | Encounter ID     |
| Contact Info   |        | Patient ID (ref)   |        | Patient ID (ref) |
|                |        | Doctor ID (ref)    |        | Doctor ID (ref)  |
|                |        | Appointment Slot   |        | Clinical Notes   |
|                |        | Status             |        | Diagnosis        |
+----------------+        +--------------------+        | Treatment Plan   |
                                   |                    | Vital Signs      |
                                   |                    +------------------+
Staff Management                   |
+--------------------+              |
| StaffMember (root) |<-------------+
| Staff ID           |
| Role               |
| Working Schedule   |
+--------------------+

Billing and Payments
+--------------------+        +------------------+
| Invoice (root)     |<------ | Payment (root)   |
| Invoice ID         |  id    | Payment ID       |
| Patient ID (ref)   |        | Invoice ID (ref) |
| Invoice Items      |        | Amount           |
| Outstanding Balance|        +------------------+
+--------------------+
```

Arrows indicate references by identity. They do not imply object ownership
across aggregate boundaries.

## 5. Core Concepts

### Patient

`Patient` is the aggregate root for patient management. The `Patient ID`
identifies the person across the system. `Patient Record` refers to the
information held in this aggregate, not to a second entity.

`Contact Information` is a value object. `Medical History` is not owned by the
`Patient` aggregate as editable data; it is a projection of clinical facts
recorded in `Encounter` aggregates.

Appointment and encounter history are referenced by `patient_id`, not embedded
in the patient aggregate.

### Appointment

`Appointment` is the aggregate root for a scheduled time. It reserves an
`Appointment Slot`, references a `Patient` and a `Doctor` by identifier, and
has a lifecycle status.

`Appointment Slot` is a value object: it is defined by a practitioner, start
time, and end time and has no independent lifecycle. `Appointment Schedule` is
a view over appointments, not an entity.

`Cancellation`, `Rescheduling`, `Check-in`, and `No-show` are behaviors or
outcomes of an appointment, not separate domain objects.

### Encounter

`Encounter` is the aggregate root for actual clinical interaction. It is
created only when care begins, never when an appointment is booked. It
references the `Patient` and `Doctor` by identifier and may optionally be
linked to the appointment that led to it.

`Clinical Note`, `Diagnosis`, `Treatment Plan`, and `Vital Signs` belong to the
encounter aggregate. `Consultation` is the assessment documented within the
encounter. `Prescription` is modeled in the future Medication context because
it has its own identity and lifecycle.

### Staff Member and Doctor

`StaffMember` is the aggregate root in Staff Management. `Role` describes the
responsibility assigned to that person; `Doctor`, `Nurse`, and `Receptionist`
are roles, not separate people.

`Working Schedule` is a collection of `Availability` value objects. The
availability data used when booking appointments comes from this context.

### Invoice and Payment

`Invoice` is the aggregate root for a request for payment. It contains
`Invoice Items`, each with a `Service Fee`. `Outstanding Balance` is derived
from the invoice total, payments, and refunds; it is not an independent entity.

`Payment` is a separate aggregate root because money received has its own
identity and lifecycle. It references the invoice by `invoice_id`. A `Refund`
reverses money previously received.

## 6. Lifecycles

### Appointment Status

| Status | Meaning |
|---|---|
| `SCHEDULED` | Appointment created and slot reserved |
| `CONFIRMED` | Appointment confirmed for the patient |
| `CANCELLED` | Appointment terminated before the encounter |
| `COMPLETED` | Appointment fulfilled |
| `NO_SHOW` | Patient did not attend |

Terminal states are `CANCELLED`, `COMPLETED`, and `NO_SHOW`.

### Encounter Status

| Status | Meaning |
|---|---|
| `IN_PROGRESS` | Clinical interaction has started |
| `COMPLETED` | Clinical interaction has ended and documentation is final |

Billing and prescription activities may occur during or after an encounter,
but they do not change the appointment status.

## 7. Relationships and Invariants

The model enforces these high-level relationships:

- One `Appointment` reserves one `Appointment Slot` for one patient and one
  doctor.
- One `Encounter` belongs to one patient and one doctor; it may be linked to at
  most one appointment.
- One `Invoice` belongs to one patient and may have many `Invoice Items`.
- One `Invoice` may receive many `Payments` and `Refunds`.
- Availability is owned by a staff member; an appointment cannot create
  availability.

Detailed rules are recorded in [domain-rules.md](domain-rules.md).

## 8. Traceability

| Requirement area | Domain model concept |
|---|---|
| PAT-001 to PAT-008 | `Patient` aggregate |
| APT-001 to APT-008 | `Appointment` aggregate and `Appointment Slot` |
| ENC-001 to ENC-007 | `Encounter` aggregate |
| CLI-001 to CLI-006 | Clinical concepts inside `Encounter` |
| PRA-001 to PRA-006 | `StaffMember`, `Role`, `Availability` |
| MED-001 to MED-006 | Future Medication context (`Prescription`) |
| LAB-001 to LAB-006 | Future Laboratory context |
| ORG-001 to ORG-004 | Future Organization context |
| IAM-001 to IAM-006 | Future User and Access Management context |
| AUD-001 to AUD-005 | Future Audit context |

## 9. Open Questions

- Is `Consultation` a separate clinical record when an encounter contains
  multiple assessments?
- Should `Diagnosis` have its own lifecycle, for example confirmed versus
  suspected?
- Should `Prescription` be promoted from a future context in the first
  implementation?
- How are duplicate patients merged without losing historical references?
