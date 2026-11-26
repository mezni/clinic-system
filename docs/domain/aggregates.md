# Aggregates

## 1. Purpose

Define the aggregate boundary for `Appointment` and explain why its related
concepts are modelled inside or outside that boundary. This is Stage 1 domain
discovery: the document records design intent, not implemented production code.

## 2. What an Aggregate Is

An aggregate is a cluster of related domain objects treated as **one consistency
boundary**. It has:

| Element | Appointment aggregate |
|---|---|
| One aggregate root | `Appointment`, identified by `Appointment ID` |
| Entities | None in the initial model |
| Value objects | `TimeSlot`, `AppointmentStatus`, `Appointment ID` wrapper |
| Business rules | INV-01 to INV-07 in [domain-invariants.md](domain-invariants.md) |

The aggregate is the unit that must be consistent when a business operation
finishes. Anything whose consistency does not have to hold at the same moment
belongs outside the boundary.

## 3. Appointment Aggregate Boundary

```text
Appointment aggregate                 Other aggregates (referenced by ID)
+-------------------------------+     +-----------------------------+
| Appointment (root)            |     | Patient                     |
|   Appointment ID              |     |   Patient ID                |
|   patient_id      ------------|---->|                             |
|   doctor_id       ------------|---->| StaffMember (Doctor role)   |
|   TimeSlot                    |     |   Doctor ID                 |
|     start, end                |     +-----------------------------+
|   AppointmentStatus           |
+-------------------------------+     +-----------------------------+
                                      | Encounter (separate root)   |
                                      +-----------------------------+
```

`Appointment` holds `patient_id` and `doctor_id`, not the `Patient` and
`StaffMember` objects. This follows the DDD principle: **reference other
aggregates by identity**. It keeps the appointment small and avoids loading an
object graph across several consistency boundaries.

## 4. Boundary Decisions

| Concept | Inside appointment aggregate? | Reason |
|---|---|---|
| `Appointment` | Yes | Root; owns the lifecycle and its invariants |
| `TimeSlot` | Yes | Value object describing the reserved interval; the appointment slot from [value-objects.md](value-objects.md) |
| `AppointmentStatus` | Yes | Value object describing lifecycle state |
| `Patient ID`, `Doctor ID` | Yes, as references | Needed for identity, but the referenced aggregates are not owned |
| `Patient` | No | Has its own lifecycle and hundreds of possible appointments |
| `Doctor` / `StaffMember` | No | Availability and employment belong to Staff Management |
| `Encounter` | No | Created when care starts, with its own identity and lifecycle |
| Appointment schedule | No | Read model over many appointments |

A `Patient` and an `Appointment` should not automatically belong to the same
aggregate. Each appointment can be confirmed, cancelled, or completed
independently of the patient's other appointments.

## 5. Aggregate Root Candidates

| Candidate | Aggregate root? | Reason |
|---|---|---|
| `Appointment` | Yes | Controls its own lifecycle transitions |
| `TimeSlot` | No | Value object, no identity |
| `AppointmentStatus` | No | Value object, no identity |
| `Patient` | Potentially | May own patient-specific information |
| `Encounter` | Potentially | May own clinical encounter information |
| `Invoice` | Potentially | May own invoice items and totals |

Candidate roots in other contexts are recorded here for completeness. Their
boundaries are defined by their own contexts, not by the Appointment aggregate.

## 6. Aggregate and Transaction Boundaries

An aggregate defines the consistency boundary; a database transaction provides
atomic persistence of the changes to one aggregate:

```text
BEGIN
  appointment = appointment_repository.get_by_id(appointment_id)
  appointment.confirm()
  appointment_repository.save(appointment)
COMMIT
```

If a rule fails, the transaction must not persist an invalid state. Cross-aggregate
checks, such as doctor availability or slot conflicts, are coordinated outside
the aggregate by an application service; see section 8 of
[domain-rules.md](domain-rules.md).

| Concept | Responsibility |
|---|---|
| Aggregate | Defines the consistency boundary |
| Aggregate root | Protects the invariants |
| Repository | Loads and saves aggregates |
| Database transaction | Provides atomic persistence |
| Application service | Coordinates use cases across aggregates |

## 7. Design Trade-offs

| Decision | Benefit | Trade-off |
|---|---|---|
| Small aggregate | Easier consistency management | More coordination between aggregates |
| Reference by ID | Loose coupling | Extra retrieval when referenced data is needed |
| Root controls modification | Business rules are protected | More explicit domain methods |
| Immutable value objects | Predictable behavior | Changes create new instances |

## 8. Verification Checklist

- [ ] Every object inside the boundary is needed for an appointment invariant.
- [ ] Other aggregates are referenced only by identifier.
- [ ] State changes go through root behavior methods, not public attribute assignment.
- [ ] One transaction changes at most one aggregate's state.
- [ ] Invariants have a named owner and a failing unit test.

## 9. Related Documents

- [aggregate-roots.md](aggregate-roots.md) — the only entry point for changing the aggregate.
- [domain-invariants.md](domain-invariants.md) — rules the root must enforce.
- [appointment-lifecycle.md](appointment-lifecycle.md) — allowed state transitions.
