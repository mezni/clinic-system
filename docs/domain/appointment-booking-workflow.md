# Appointment Booking Workflow

## 1. Purpose

Document the `BookAppointment` use case end to end and show how Domain
Services, Application Services, and Domain Events divide the work. This is a
Stage 1 design document; the workflow described here is illustrative and is not
yet implemented in production code.

The goal is separation: the endpoint handles HTTP, the Application Service
coordinates the use case, the Domain Service enforces availability, the
aggregate owns its state, and an event announces the result.

## 2. Business Requirements

| ID | Requirement |
|---|---|
| BR-01 | A patient must have a valid identifier |
| BR-02 | A doctor must have a valid identifier |
| BR-03 | The appointment must have a valid time slot |
| BR-04 | The doctor must not have another active appointment during that period |
| BR-05 | A successfully booked appointment starts with `SCHEDULED` |
| BR-06 | A successfully booked appointment records an `AppointmentBooked` event |

These refine the appointment requirements in
[functional-requirements.md](../requirements/functional-requirements.md)
(`APT-001` to `APT-004`) and the invariant inventory in
[domain-invariants.md](domain-invariants.md).

## 3. Participants

```text
Receptionist
    |
    v
FastAPI endpoint ----> BookAppointmentService (application)
                             |
       +---------------------+---------------------+
       |                     |                     |
       v                     v                     v
AppointmentAvailability   Appointment           AppointmentRepository
Service (domain)          aggregate (domain)    (domain protocol)
       |                     |                     |
       +---- check ----------+---- save -----------+
                             |
                             v
                    EventDispatcher (application port)
                             |
                             v
                    handlers / notifications
```

| Participant | Layer | Role |
|---|---|---|
| FastAPI endpoint | Interface | Parses input, authorizes, delegates, maps result |
| `BookAppointmentCommand` | Application | Explicit use-case input |
| `BookAppointmentService` | Application | Coordinates the steps |
| `AppointmentAvailabilityService` | Domain | Decides whether the doctor is free |
| `Appointment` | Domain | Creates a valid aggregate and records the event |
| `AppointmentRepository` | Domain protocol | Defines conflict check and save operations |
| `EventDispatcher` | Application port | Abstracts event delivery |
| Notification handler | Outside the use case | Reacts to `AppointmentBooked` |

## 4. Sequence

```text
Receptionist -> Endpoint: POST appointment request
Endpoint     -> Application Service: execute(command)
Application  -> TimeSlot: construct(start, end)
Application  -> Domain Service: ensure_available(doctor_id, time_slot)
Domain Service -> Repository: has_conflicting_appointment(doctor_id, time_slot)
Domain Service -> Application: available, or DomainException
Application  -> Aggregate: book(appointment_id, patient_id, doctor_id, time_slot)
Aggregate    -> Application: Appointment + recorded AppointmentBooked
Application  -> Repository: save(appointment)
Application  -> Dispatcher: dispatch(event)
Dispatcher   -> Handler: notification
Application  -> Endpoint: booked appointment
```

The order matters. Availability is checked before creation, creation records the
event, and the event is dispatched only after a successful save. A failure at
any step stops the remaining steps.

## 5. Step-by-Step Responsibilities

| Step | Owner | Rule or concern |
|---|---|---|
| Validate request shape and permissions | Endpoint | HTTP and authorization, not business rules |
| Build `TimeSlot` | Application Service / Value Object | start before end, compatible timezones |
| Check doctor availability | Domain Service | Business decision, raises `DomainException` on conflict |
| Create `Appointment` with `SCHEDULED` | Aggregate root | INV-01 to INV-03, factory method |
| Record `AppointmentBooked` | Aggregate root | Fact of the booking |
| Persist the aggregate | Repository implementation | Infrastructure detail behind a protocol |
| Dispatch recorded events | Application Service | Via the dispatcher port |
| Send notification | Event handler | Loose coupling from the booking use case |

The Application Service coordinates but does not decide availability, validate
time itself, or implement storage. See
[application-services.md](application-services.md) and
[domain-services.md](domain-services.md).

## 6. Illustrative Contracts

The use case depends on two narrow abstractions:

```python
from typing import Protocol
from uuid import UUID

from clinic_system.domain.entities.appointment import Appointment
from clinic_system.domain.value_objects.time_slot import TimeSlot


class AppointmentRepository(Protocol):
    def has_conflicting_appointment(
        self,
        doctor_id: UUID,
        time_slot: TimeSlot,
    ) -> bool: ...

    def save(self, appointment: Appointment) -> None: ...


class EventDispatcher(Protocol):
    def dispatch(self, event: object) -> None: ...
```

These are abstractions, not database or messaging implementations. The
application can later use PostgreSQL, SQLite, or an in-memory repository
without changing domain rules. The full illustrative `BookAppointmentService`
is in section 6 of [application-services.md](application-services.md).

## 7. Overlap Detection

The repository's conflict check is expressed by a query equivalent to:

```sql
SELECT EXISTS (
    SELECT 1
    FROM appointments
    WHERE doctor_id = :doctor_id
      AND status NOT IN ('cancelled')
      AND start_time < :requested_end
      AND end_time > :requested_start
);
```

Two intervals overlap when both conditions hold:

```text
existing.start < requested.end
AND existing.end > requested.start
```

Adjacent slots are allowed because the comparisons are strict:

```text
Appointment A: 10:00-10:30
Appointment B: 10:30-11:00
Result: no overlap
```

Cancelled appointments are excluded, so their time becomes available again.
The check is a domain policy implemented behind the repository protocol; an
infrastructure constraint may enforce it but does not replace the domain rule.

## 8. Failure Handling

| Failure | Behavior |
|---|---|
| Invalid patient or doctor ID | Rejected before the availability check |
| Invalid time slot | `TimeSlot` validation error; no repository or dispatcher call |
| Doctor unavailable | `DomainException("Doctor is not available during this time slot")`; no save, no event |
| Repository save fails | Operation fails; no event is dispatched |
| Dispatcher fails after save | Appointment remains saved; event delivery is unresolved |

Domain exceptions are translated to HTTP responses at the endpoint. The domain
service itself never builds an API response.

Production concerns not solved in this stage:

- **Concurrent bookings:** two requests can both pass the availability check
  before either saves. Use database constraints, locking, or serialization to
  prevent double booking.
- **Save/dispatch crash:** an appointment can be saved without its event being
  delivered. Use the Transactional Outbox Pattern so the event is stored in the
  same transaction; see section 6 of [domain-events.md](domain-events.md).

## 9. Planned Structure

```text
src/
└── clinic_system/
    ├── domain/
    │   ├── entities/appointment.py
    │   ├── events/appointment_booked.py
    │   ├── services/appointment_availability.py
    │   ├── repositories/appointment_repository.py
    │   ├── value_objects/time_slot.py
    │   ├── value_objects/appointment_status.py
    │   └── exceptions/domain_exception.py
    └── application/
        ├── commands/book_appointment.py
        ├── services/book_appointment_service.py
        └── ports/event_dispatcher.py
```

The domain layer defines business concepts and abstractions. The application
layer coordinates their execution. No files exist in these directories yet.

Tests belong in `tests/unit/application/test_book_appointment.py` and use mocks
for the repository, availability service, and dispatcher; the command to run
once the tooling is configured is `uv run pytest`.

## 10. Unit Test Matrix

| Test | Expected result |
|---|---|
| Doctor available | Appointment created with `SCHEDULED` status and saved |
| Doctor unavailable | `DomainException`; no save and no dispatch |
| Invalid time slot | Validation exception; no save |
| Repository save fails | Application operation fails; no dispatch |
| Appointment successfully booked | One `AppointmentBooked` event dispatched |
| Availability check fails | Repository `save` never called |
| Adjacent slots | No conflict reported |

## 11. Design Trade-offs

| Decision | Benefit | Trade-off |
|---|---|---|
| Domain Service for availability | Cross-appointment rule is centralized | Can become a dumping ground |
| Application Service | Visible, testable use case | More classes and files |
| Domain Event | Loose coupling to notifications | Event delivery complexity |
| Repository Protocol | Infrastructure independence | Extra abstraction |
| Command DTO | Explicit use-case input | More DTOs |
| Dispatch after save | Simple first implementation | Crash can lose the event |

These trade-offs are intentional in Stage 1; production hardening is deferred.

## 12. Verification Checklist

- [ ] The endpoint contains no booking business logic.
- [ ] The Application Service does not decide availability or validate slots itself.
- [ ] The Domain Service has no HTTP or database implementation dependency.
- [ ] The aggregate starts as `SCHEDULED` and records exactly one event.
- [ ] Saving happens before dispatch, and failures stop later steps.
- [ ] Overlap detection allows adjacent slots and ignores cancelled appointments.
- [ ] Concurrency and outbox requirements are documented as future work.
- [ ] Unit tests cover success, rule violation, validation, and persistence failure.

## 13. Related Documents

- [domain-services.md](domain-services.md) — the availability rule.
- [application-services.md](application-services.md) — the coordinating service.
- [domain-events.md](domain-events.md) — the booked event and delivery concerns.
- [appointment-lifecycle.md](appointment-lifecycle.md) — the initial `SCHEDULED` status.
