# Aggregate Roots

## 1. Purpose

Define which entity is allowed to change the state of the Appointment aggregate,
and how that change is exposed to application code. This is a Stage 1 design
document; implementation follows in a later step.

## 2. Definition

An aggregate root is the entity responsible for the consistency of its
aggregate. It is the **only entry point for modifying the aggregate**. External
code loads the root, calls a behavior method, and saves the root again.

```text
application service
      |
      v
appointment.confirm() ----> invariant check ----> status changes
      ^
      |
   repository.load / repository.save
```

In this design, a status change is a business operation, not an assignment:

```text
Preferred:  appointment.cancel()
Rejected:   appointment.status = "completed"
```

Direct assignment bypasses the checks in
[domain-invariants.md](domain-invariants.md). For example, it could mark a
cancelled appointment as completed after a receptionist cancelled it.

## 3. Root Classification

| Candidate | Aggregate root? | Reason |
|---|---|---|
| `Appointment` | Yes | Controls the appointment lifecycle |
| `TimeSlot` | No | Value object |
| `AppointmentStatus` | No | Value object |
| `Patient` | Potentially | May own patient-specific information |
| `Encounter` | Potentially | May own clinical encounter information |
| `Invoice` | Potentially | May own invoice items and totals |

Only `Appointment` is the root of the aggregate described in
[aggregates.md](aggregates.md).

## 4. Root Behavior

| Operation | Allowed from | Resulting status | Rules checked |
|---|---|---|---|
| `confirm()` | `SCHEDULED` | `CONFIRMED` | INV-01 to INV-03 at construction |
| `cancel()` | `SCHEDULED`, `CONFIRMED` | `CANCELLED` | INV-05 |
| `check_in()` | `CONFIRMED` | `CHECKED_IN` | INV-04, INV-06 |
| `complete()` | `CHECKED_IN` | `COMPLETED` | INV-07 |

Each method

1. checks the current status against the allowed starting status,
2. raises a `DomainException` if the transition is not allowed, and
3. mutates state only after all checks pass.

The transition table is the contract in
[appointment-lifecycle.md](appointment-lifecycle.md).

## 5. Encapsulation

The Stage 1 sketch exposes `status` as a public dataclass attribute. That is
not sufficient for production: application code must not be able to set status
to an arbitrary string or enum value.

Planned strengthening:

- keep the root's state private,
- expose behavior methods named with the ubiquitous language, and
- expose status read-only for queries.

Until that encapsulation is implemented, the root methods are the only
supported way to change state. Tests must exercise the methods, not attribute
assignment.

## 6. Repository Responsibility

A repository loads and saves the root; it does not modify aggregate internals:

```text
appointment = appointment_repository.get_by_id(appointment_id)
appointment.confirm()
appointment_repository.save(appointment)
```

Only one repository is needed for this aggregate. Local entities inside an
aggregate are never loaded or saved independently. Persistence and transaction
handling are infrastructure concerns described in later stages.

## 7. Unit Testing the Root

Tests should demonstrate that the root rejects invalid behavior:

| Test case | Expected result |
|---|---|
| Confirm a scheduled appointment | Status becomes `CONFIRMED` |
| Check in a cancelled appointment | `DomainException`, status unchanged |
| Complete a confirmed appointment without check-in | `DomainException`, status unchanged |
| Cancel a completed appointment | `DomainException`, status unchanged |
| Reject an invalid ID, patient ID, doctor ID, or slot | Construction fails |

A rejected operation must leave the aggregate in its previous valid state.

## 8. Design Trade-offs

| Decision | Benefit | Trade-off |
|---|---|---|
| Root controls modifications | Business rules are protected | More domain methods to maintain |
| Reference other roots by ID | Loose coupling | Extra retrieval in application services |
| Domain exceptions | Explicit rule violations | Callers must handle them |

## 9. Related Documents

- [aggregates.md](aggregates.md) — what the boundary contains.
- [domain-invariants.md](domain-invariants.md) — the rules the root enforces.
- [appointment-lifecycle.md](appointment-lifecycle.md) — the allowed transitions.
