# Entities

## 1. Purpose

Identify which domain concepts have identity and continuity, and therefore
must be modeled as entities. This is an initial classification to guide future
implementation; it is not production code.

An entity is defined by its identity, not by the sum of its attributes. A
change to its attributes does not create a new entity.

## 2. Classification Rule

| Question | If yes | If no |
|---|---|---|
| Does the business track this concept over time? | Entity candidate | Value object or projection |
| Does it need a unique identifier? | Entity candidate | Value object |
| Can its values change while remaining the same concept? | Entity | Value object should be replaced |
| Is it only meaningful inside another concept? | Local entity or value object | Aggregate root |

## 3. Aggregate Roots

Aggregate roots are the only entities that may be loaded directly and modified
from outside their aggregate.

| Aggregate root | Identity | Why identity is required | Lifecycle |
|---|---|---|---|
| `Patient` | `Patient ID` | A person remains the same patient when contact details change | Registered → Active → Deactivated |
| `Appointment` | `Appointment ID` | A booked time is tracked through rescheduling and status changes | Scheduled → Confirmed → Completed / Cancelled / No-show |
| `Encounter` | `Encounter ID` | Clinical care is tracked independently of the appointment | In progress → Completed |
| `StaffMember` | Staff ID / `Doctor ID` when acting as a doctor | Employment and role data change while the person remains the same staff member | Registered → Active → Deactivated |
| `Invoice` | `Invoice ID` | A request for payment is tracked until settled | Issued → Partially paid → Paid / Refunded |
| `Payment` | `Payment ID` | Each receipt of money is distinct and may be refunded | Recorded → Refunded |

### Patient

`Patient` owns the patient's identity and contact information. It is the root
of the Patient aggregate.

- Identity: `Patient ID`, generated on registration.
- Behavior: register, update demographics, update contact information,
  deactivate.
- Invariants: unique identity, valid contact information, no silent deletion.
- Related requirements: PAT-001 to PAT-008.

`Patient Record` is the aggregate as a whole, not a second entity.
`Medical History` is a projection of clinical facts from encounters, not an
entity owned by `Patient`.

### Appointment

`Appointment` owns a reserved slot and its lifecycle. It references a patient
and a doctor by identifier.

- Identity: `Appointment ID`, generated when booked.
- Behavior: book, confirm, reschedule, cancel, check in, mark no-show,
  complete.
- Invariants: slot must be available, no overlapping appointment for the same
  doctor, terminal states cannot be changed.
- Related requirements: APT-001 to APT-008.

`Appointment Slot` is not an entity; it is a value object. Two slots with the
same doctor and time interval are the same business slot.

### Encounter

`Encounter` owns the clinical documentation created while care is provided.

- Identity: `Encounter ID`, generated when the encounter starts.
- Behavior: start, record clinical note, record diagnosis, record vital signs,
  define treatment plan, complete.
- Invariants: one patient and one doctor, valid status transitions, completion
  only when required documentation is present.
- Related requirements: ENC-001 to ENC-007, CLI-001 to CLI-006.

An encounter is never created by an appointment alone. It may reference an
appointment by `appointment_id`, but the two lifecycles remain separate.

### Staff Member

`StaffMember` owns employment identity, role, and working schedule.

- Identity: staff identifier, with `Doctor ID` used when the person is
  identified clinically.
- Behavior: register, assign role, define availability, update working
  schedule, deactivate.
- Invariants: assigned role must be valid, availability intervals must be
  coherent.
- Related requirements: PRA-001 to PRA-006.

`Doctor`, `Nurse`, and `Receptionist` are roles, not additional people. The
domain uses `Doctor` when the staff member is providing medical care.

### Invoice

`Invoice` owns the charges requested for payment.

- Identity: `Invoice ID`.
- Behavior: issue, add invoice item, recalculate total, mark settled.
- Invariants: each item has a service fee, invoice total equals the sum of
  items, outstanding balance derives from payments and refunds.
- Related requirements: future Billing context.

### Payment

`Payment` owns the fact that money was received.

- Identity: `Payment ID`.
- Behavior: record payment, issue refund.
- Invariants: positive amount, references a valid invoice, refund cannot
  exceed the payment.
- Related requirements: future Billing context.

`Payment` is not an `Invoice Item`: one is money received, the other is money
requested.

## 4. Local Entities

Local entities have identity only inside their aggregate root. They are not
loaded independently and are not shared across aggregates.

| Local entity | Parent aggregate | Local identity | Why it is an entity |
|---|---|---|---|
| `Invoice Item` | `Invoice` | Invoice item number | Each charge on an invoice must be individually identified and adjusted |
| `Clinical Note` | `Encounter` | Note number within encounter | Notes may be recorded and corrected while belonging to the same encounter |
| `Diagnosis` | `Encounter` | Diagnosis number within encounter | Multiple diagnoses can be recorded and reviewed separately |
| `Treatment Plan` | `Encounter` | Plan number within encounter | Multiple planned interventions can be tracked within one encounter |

Local identity is not a global identifier. A `Diagnosis` is never referenced
from another aggregate by its local number.

## 5. Explicitly Not Entities

| Term | Classification | Reason |
|---|---|---|
| `Patient Record` | Aggregate view | It is the information held by the `Patient` aggregate |
| `Medical History` | Projection | Derived from clinical records, not independently edited |
| `Appointment Slot` | Value object | Defined by doctor and time interval |
| `Appointment Schedule` | Read model | A view over appointments |
| `Consultation` | Value object / documentation | A medical assessment recorded during an encounter |
| `Vital Signs` | Value object | Measurements defined by their values |
| `Prescription` | Future aggregate | Has independent identity and lifecycle in the Medication context |
| `Availability` | Value object | A period defined by values, owned by a staff member |
| `Working Schedule` | Collection of value objects | No identity of its own |
| `Role` | Value object | A responsibility label, not a person |
| `Contact Information` | Value object | Defined by phone, email, and address |
| `Outstanding Balance` | Derived value | Calculated from invoice, payments, and refunds |
| `Service Fee` | Value object | An amount charged for a service |
| `Cancellation` | Outcome / status change | A lifecycle transition of an appointment |
| `Rescheduling` | Behavior | A change to the appointment slot |
| `No-show` | Outcome / status | Not a person, object, or separate record |
| `Check-in` | Event / behavior | Records arrival on an appointment |
| `Customer` | Not used | The care recipient is always `Patient` |
| `Client` | Not used | Registration does not create a different identity |
| `Visit` | Not used | Ambiguous between `Appointment` and `Encounter` |

## 6. Implementation Guidance

When entities are implemented:

- Generate identities inside the domain or via an explicit identity service;
  do not expose database sequence values as business identifiers.
- Keep entity state private and expose behavior methods named with the
  ubiquitous language.
- Enforce invariants inside the entity or aggregate root before state changes.
- Reference other aggregates only by identifier.
- Do not add an entity solely because a noun appears in a user story.
- Revisit this classification when a concept gains an independent lifecycle.
