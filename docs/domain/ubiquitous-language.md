# Ubiquitous Language

## 1. Purpose

Document how business stakeholders describe clinic work and how those
statements translate into domain terminology. The same terms must be used in
conversations, documentation, requirements, and code.

See [glossary.md](glossary.md) for the definitions of every term used here.

## 2. Business Statement

> "When a patient books an appointment with a doctor, the receptionist checks
> the doctor's availability."

## 3. Domain Interpretation

1. A `Patient` requests an `Appointment`.
2. The `Doctor` has an availability schedule.
3. The `Appointment` reserves an available `Appointment Slot`.
4. The `Appointment` receives a unique identifier.
5. The `Appointment` has a lifecycle status.

Example:

| Field | Value |
|---|---|
| Patient | John Smith |
| Doctor | Dr. Sarah |
| Date | 2026-12-01 |
| Time | 10:00 AM |

Resulting appointment:

```text
Appointment:
    id: APT-1001
    patient_id: PAT-1001
    doctor_id: DOC-1001
    status: CONFIRMED
```

`CONFIRMED` is a state of the `Appointment`. It does not mean that an
`Encounter` exists; an encounter is created only when clinical care actually
takes place.

## 4. Additional Translations

| Business statement | Domain interpretation |
|---|---|
| "The receptionist registers a new patient." | `Receptionist` performs `Registration`; the system creates a `Patient` and a `Patient ID`. |
| "The patient arrived for their visit." | `Receptionist` records `Check-in` on the `Appointment`. |
| "The doctor saw the patient." | `Doctor` starts an `Encounter` for the `Patient`; an `Encounter ID` is assigned. |
| "The doctor wrote up the assessment and medication." | `Doctor` records a `Consultation`, `Diagnosis`, `Clinical Note`, and `Prescription` in the `Encounter`. |
| "The patient did not show up." | The `Appointment` is marked `No-show`; it is not a `Cancellation`. |
| "The appointment was moved to next week." | `Receptionist` performs `Rescheduling` of the `Appointment` to another `Appointment Slot`. |
| "The clinic sent a bill and the patient paid part of it." | An `Invoice` with `Invoice Items` exists; a `Payment` reduces the `Outstanding Balance`. |

## 5. Naming Conventions for Future Code

These conventions preserve the ubiquitous language when the domain model is
implemented. They are illustrative; no domain code is implemented at this
stage.

- Use the glossary term as the class or concept name: `Patient`,
  `Appointment`, `Encounter`, `Doctor`, `Invoice`.
- Use explicit role names: `Receptionist` and `Nurse` rather than generic
  `User` in domain conversations about those roles.
- Use identifiers derived from the concept name: `patient_id`,
  `appointment_id`, `encounter_id`, `doctor_id`.
- Use lifecycle states from the domain vocabulary: `SCHEDULED`, `CONFIRMED`,
  `CANCELLED`, `COMPLETED`, `NO_SHOW`.
- Use business verbs for operations: `register_patient`, `book_appointment`,
  `reschedule_appointment`, `check_in`, `start_encounter`, `record_payment`.
- Do not use `Customer`, `Client`, or unqualified `Visit` as domain names.
- Do not collapse `Appointment` and `Encounter` into one type.
- Do not use technical storage names in business conversations or domain
  terminology, for example `appointment_row` or `patient_entity`.

Illustration only (to be refined when Entities, Value Objects, Aggregates, and
domain invariants are introduced):

```python
class AppointmentStatus:
    SCHEDULED = "scheduled"
    CONFIRMED = "confirmed"
    CANCELLED = "cancelled"
    COMPLETED = "completed"
    NO_SHOW = "no_show"


class Appointment:
    def __init__(
        self,
        appointment_id,
        patient_id,
        doctor_id,
        scheduled_at,
        status,
    ):
        self.appointment_id = appointment_id
        self.patient_id = patient_id
        self.doctor_id = doctor_id
        self.scheduled_at = scheduled_at
        self.status = status
```

## 6. Validation Checklist

A statement or design uses the ubiquitous language when:

- [ ] Each business concept has a term from the glossary.
- [ ] The same term always carries the same meaning.
- [ ] Similar words have not been merged into one domain concept.
- [ ] Lifecycle states use the agreed status values.
- [ ] Identifiers and operation names follow the concept they describe.
