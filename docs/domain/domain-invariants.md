# Domain Invariants

## 1. Purpose

Record the conditions that must always hold true inside the Appointment
aggregate, who enforces them, and how they are tested. This is a Stage 1 design
document; the rules are not yet implemented in production code.

An invariant is a business condition that must remain true within an aggregate.
Rules involving other aggregates are coordinated outside the boundary and are
not appointment invariants.

## 2. Invariant Inventory

| ID | Invariant | Enforced by | Phase |
|---|---|---|---|
| INV-01 | An appointment must have a valid time slot | `TimeSlot` value object | Construction |
| INV-02 | An appointment must have a patient ID | `Appointment` root | Construction |
| INV-03 | An appointment must have a doctor ID | `Appointment` root | Construction |
| INV-04 | A cancelled appointment cannot be checked in | `Appointment.check_in()` | State transition |
| INV-05 | A completed appointment cannot be cancelled | `Appointment.cancel()` | State transition |
| INV-06 | Only confirmed appointments can be checked in | `Appointment.check_in()` | State transition |
| INV-07 | Only checked-in appointments can be completed | `Appointment.complete()` | State transition |

Construction rules prevent an invalid aggregate from existing. Transition rules
prevent a valid aggregate from becoming invalid.

## 3. Construction Rules

`TimeSlot` validates its own values:

- start must be before end,
- start and end must use compatible timezones.

`Appointment` validates that it was given an appointment ID, patient ID, doctor
ID, and a `TimeSlot` of the expected type. A missing or malformed value raises a
validation error at construction; the aggregate is never created.

## 4. Transition Rules

| Operation | Starting status allowed | Rejected example |
|---|---|---|
| `confirm()` | `SCHEDULED` | Confirming a cancelled appointment |
| `cancel()` | `SCHEDULED`, `CONFIRMED` | Cancelling a completed appointment (INV-05) |
| `check_in()` | `CONFIRMED` | Checking in a cancelled appointment (INV-04) |
| `complete()` | `CHECKED_IN` | Completing a confirmed appointment without arrival (INV-07) |

INV-04 and INV-06 both guard `check_in()`: the current status must be
`CONFIRMED`, which also excludes `CANCELLED`. They are listed separately because
they come from different business statements.

A failed check raises `DomainException`, a base exception for domain rule
violations. The aggregate state must remain unchanged.

## 5. Why Direct Assignment Is Unsafe

```text
appointment.cancel()                  -> checks status, sets CANCELLED
appointment.status = "completed"      -> no checks, can overwrite CANCELLED
```

Application code must call behavior methods on the aggregate root. The later
implementation will strengthen encapsulation so status cannot be assigned from
outside the root; see section 5 of
[aggregate-roots.md](aggregate-roots.md).

## 6. Rules Outside This Aggregate

These rules are not appointment invariants because they involve other
aggregates or contexts:

| Rule | Owner |
|---|---|
| Appointment must reference an existing patient and doctor | Application service resolves identifiers before creation |
| Slot must be available for the doctor | Appointment Management with Staff Management availability data |
| Two appointments for the same doctor must not overlap | Reservation policy with data supplied by the application layer |
| Only an authorized user may cancel | Application / authorization layer |
| Persisting the change atomically | Database transaction |

See [domain-rules.md](domain-rules.md), sections 4 and 8, for the broader rule
placement. Infrastructure constraints never replace domain rules.

## 7. Reconciliation Needed

This stage's lifecycle adds `CHECKED_IN` and requires it before completion
(INV-07). Earlier documents include a `NO_SHOW` status and allow
`CONFIRMED` → `COMPLETED` directly (BR-APT-006 in
[domain-rules.md](domain-rules.md)). These models conflict.

Before implementation, confirm with domain experts:

- whether `CHECKED_IN` replaces or extends the earlier status list,
- whether `NO_SHOW` remains a terminal state, and
- whether `CONFIRMED` → `COMPLETED` is still permitted.

Until then, do not implement both transition sets. The working contract for
this stage is [appointment-lifecycle.md](appointment-lifecycle.md).

## 8. Verification Checklist

- [ ] Each invariant names its owner and enforcement phase.
- [ ] Each invariant has a unit test that attempts to violate it.
- [ ] Rejected operations leave the aggregate unchanged.
- [ ] Cross-aggregate rules are not embedded in `Appointment`.
- [ ] Status can only change through aggregate behavior methods.
