# Domain Events

## 1. Purpose

Define how the Appointment domain records important facts that have already
happened, how those facts leave the domain, and what delivery guarantees are
still unresolved. This is a Stage 1 design document; the event described here is
illustrative and is not yet implemented in production code.

A Domain Event answers the question: **what happened in the business?**

## 2. Definition

A Domain Event is an immutable statement of a completed business occurrence.
Names use the past tense: `AppointmentBooked`, not `BookAppointment`. A command
expresses a request; an event records a fact.

| Event | Meaning |
|---|---|
| `AppointmentBooked` | An appointment was successfully booked |
| `AppointmentCancelled` | An appointment was cancelled |
| `AppointmentConfirmed` | An appointment was confirmed |
| `AppointmentCompleted` | An appointment was completed |
| `PatientRegistered` | A new patient was registered |

Only `AppointmentBooked` is in scope for this stage. The other names record the
future event vocabulary; they are not implemented yet.

## 3. AppointmentBooked

```python
from dataclasses import dataclass
from datetime import datetime, timezone
from uuid import UUID


@dataclass(frozen=True)
class AppointmentBooked:
    appointment_id: UUID
    patient_id: UUID
    doctor_id: UUID
    occurred_at: datetime

    @classmethod
    def create(
        cls,
        appointment_id: UUID,
        patient_id: UUID,
        doctor_id: UUID,
    ) -> "AppointmentBooked":
        return cls(
            appointment_id=appointment_id,
            patient_id=patient_id,
            doctor_id=doctor_id,
            occurred_at=datetime.now(timezone.utc),
        )
```

The event is frozen: consumers cannot rewrite the facts. It carries identities
and an occurrence timestamp, not the whole aggregate. It does not contain a
notification, an HTTP response, or a database row.

## 4. Recording Events in the Aggregate

The `Appointment` aggregate records the event when it is created. This keeps the
fact next to the behavior that makes it true.

```python
    @classmethod
    def book(
        cls,
        appointment_id: UUID,
        patient_id: UUID,
        doctor_id: UUID,
        time_slot: TimeSlot,
    ) -> "Appointment":
        appointment = cls(
            id=appointment_id,
            patient_id=patient_id,
            doctor_id=doctor_id,
            time_slot=time_slot,
        )
        appointment._events.append(
            AppointmentBooked.create(
                appointment_id=appointment.id,
                patient_id=appointment.patient_id,
                doctor_id=appointment.doctor_id,
            )
        )
        return appointment

    def collect_events(self) -> list[object]:
        events = list(self._events)
        self._events.clear()
        return events
```

`_events` is an internal collection with `init=False`, `repr=False`, and
`compare=False`, so it does not alter the aggregate's public constructor or
value semantics. Existing lifecycle methods (`confirm`, `cancel`, `check_in`,
`complete`) remain unchanged; adding events for those transitions is future
work and must respect [appointment-lifecycle.md](appointment-lifecycle.md).

`collect_events` drains the collection so a successfully dispatched event is
not sent twice from the same aggregate instance.

## 5. Dispatching

The Application Service reads the recorded events after saving and hands each
one to the dispatcher; see section 6 of
[application-services.md](application-services.md).

```text
Appointment.book()          -> records AppointmentBooked
repository.save()           -> persists the aggregate
collect_events()            -> reads and clears recorded events
EventDispatcher.dispatch()  -> delivers to interested handlers
```

Handlers may send a confirmation notice, update a read model, or start another
use case. The aggregate does not know who is interested and the dispatcher does
not decide business rules. This is loose coupling between business processes.

| Component | Responsibility |
|---|---|
| Aggregate | Records what happened |
| Application Service | Dispatches after a successful save |
| EventDispatcher port | Abstracts the delivery mechanism |
| Handler | Performs the reaction |

## 6. Delivery Concerns

The current order is save, then dispatch. That order has a known failure mode:

1. The appointment is saved successfully.
2. The application crashes before dispatch.
3. The event is never delivered and no notification is sent.

A second concern is duplicated delivery if dispatch is retried after a partial
failure. Consumers should therefore be idempotent.

For production, introduce the **Transactional Outbox Pattern**:

```text
BEGIN
  save appointment
  save AppointmentBooked to outbox
COMMIT
                         -> outbox publisher reads the outbox
                         -> dispatches the event
                         -> marks the outbox entry as published
```

Both the appointment and its event are persisted in one database transaction,
so the fact cannot be lost after a successful save. The outbox publisher and
handlers are infrastructure concerns for a later stage.

Concurrency is a separate concern: checking availability and saving are two
operations, so two concurrent bookings can both pass the check. Database-level
constraints or locking are needed later; see section 8 of
[appointment-booking-workflow.md](appointment-booking-workflow.md).

## 7. Event Names and Vocabulary

| Concept | Naming rule | Example |
|---|---|---|
| Command | Imperative request | `BookAppointmentCommand` |
| Event | Past-tense fact | `AppointmentBooked` |
| Handler | Reaction to a fact | Notification handler |

Using the ubiquitous language from
[ubiquitous-language.md](ubiquitous-language.md) keeps event names meaningful to
domain experts. An event named `AppointmentCreated` would describe a technical
write, not the business fact of booking.

## 8. Design Trade-offs

| Decision | Benefit | Trade-off |
|---|---|---|
| Domain Events | Loose coupling between processes | More event-handling complexity |
| Immutable event | Reliable historical fact | New facts need new event types |
| Aggregate records events | Fact is created with the state change | Aggregate holds extra internal state |
| Dispatch after save | Simple first implementation | Crash can lose delivery |
| Transactional outbox | Reliable delivery | Extra storage and publisher |

## 9. Verification Checklist

- [ ] Event names are past tense and use domain language.
- [ ] The event is immutable and contains identities and a timestamp only.
- [ ] `book()` records exactly one `AppointmentBooked` event.
- [ ] `collect_events()` clears the collection after reading.
- [ ] Dispatch happens only after a successful save.
- [ ] Outbox and concurrency controls are recorded as future work.

## 10. Related Documents

- [application-services.md](application-services.md) — dispatches recorded events.
- [domain-services.md](domain-services.md) — checks the rule before the event exists.
- [appointment-booking-workflow.md](appointment-booking-workflow.md) — full booking flow.
- [appointment-lifecycle.md](appointment-lifecycle.md) — lifecycle events to add later.
