# Appointment Lifecycle

## 1. Purpose

Define the initial lifecycle of the `Appointment` aggregate: its statuses, the
operations that move between them, and the rules that block invalid changes.
This is a Stage 1 design document; exceptional workflows will be refined later.

## 2. Statuses

| Status | Stored value | Meaning |
|---|---|---|
| `SCHEDULED` | `scheduled` | Appointment created; default status |
| `CONFIRMED` | `confirmed` | Appointment confirmed for the patient |
| `CANCELLED` | `cancelled` | Appointment cancelled; terminal |
| `CHECKED_IN` | `checked_in` | Patient arrival recorded |
| `COMPLETED` | `completed` | Appointment fulfilled; terminal |

`AppointmentStatus` is a value object implemented as a string enum. It describes
state; it is not an entity and has no lifecycle of its own.

## 3. State Transitions

```text
                    confirm()
   SCHEDULED ------------------> CONFIRMED
       |                          |     |
       | cancel()                 |     | check_in()
       v                          |     v
   CANCELLED                      |  CHECKED_IN
   (terminal)                     |     |
                                  |     | complete()
                                  |     v
                                  |  COMPLETED
                                  |  (terminal)
                                  +----> CANCELLED via cancel()
```

| From | Operation | To | Guard |
|---|---|---|---|
| `SCHEDULED` | `confirm()` | `CONFIRMED` | INV-01 to INV-03 hold from construction |
| `SCHEDULED` | `cancel()` | `CANCELLED` | Carried over from the earlier lifecycle model |
| `CONFIRMED` | `check_in()` | `CHECKED_IN` | INV-04, INV-06 |
| `CONFIRMED` | `cancel()` | `CANCELLED` | INV-05 |
| `CHECKED_IN` | `complete()` | `COMPLETED` | INV-07 |
| `CANCELLED` | Any operation | — | Rejected with `DomainException` |
| `COMPLETED` | Any operation | — | Rejected with `DomainException` |

`CANCELLED` and `COMPLETED` are terminal. `CHECKED_IN` is not terminal: it must
be followed by `complete()`.

## 4. Operation Contract

| Operation | Precondition | Postcondition | On violation |
|---|---|---|---|
| `confirm()` | Status is `SCHEDULED` | Status is `CONFIRMED` | `DomainException`, state unchanged |
| `cancel()` | Status is not `COMPLETED` | Status is `CANCELLED` | `DomainException`, state unchanged |
| `check_in()` | Status is `CONFIRMED` | Status is `CHECKED_IN` | `DomainException`, state unchanged |
| `complete()` | Status is `CHECKED_IN` | Status is `COMPLETED` | `DomainException`, state unchanged |

Only the aggregate root exposes these operations. Application services call
them after loading the root from a repository and before saving it. See
[aggregate-roots.md](aggregate-roots.md).

## 5. Deferred Workflows

The following concerns belong to later refinement and are not part of this
initial lifecycle:

| Concern | Reason it is deferred |
|---|---|
| No-show handling | Exceptional workflow; status not in this stage's enum |
| Rescheduling | Requires slot release and reservation in one transaction |
| Check-in after `SCHEDULED` | Rule INV-06 restricts check-in to confirmed appointments |
| Completion directly from `CONFIRMED` | Rule INV-07 requires a recorded arrival |
| Encounter creation | Check-in records arrival only; it does not create an encounter |

## 6. Conflict With Earlier Documentation

[domain-model.md](domain-model.md) and [domain-rules.md](domain-rules.md)
describe an earlier lifecycle that includes `NO_SHOW` and allows
`CONFIRMED` → `COMPLETED`. This stage introduces `CHECKED_IN` and INV-07, which
require `CHECKED_IN` before `COMPLETED`.

This conflict must be resolved before implementation. Confirm with domain
experts whether `NO_SHOW` and direct completion are retained, replaced, or
handled as exceptional workflows. Do not implement both sets of transitions.

## 7. Verification Checklist

- [ ] Each transition names its operation and guard.
- [ ] Terminal states reject every modifying operation.
- [ ] Unit tests cover valid transitions and rejected transitions.
- [ ] A rejected transition leaves the status unchanged.
- [ ] No deferred workflow is implemented implicitly.
