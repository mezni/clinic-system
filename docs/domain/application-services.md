# Application Services

## 1. Purpose

Define where the appointment booking use case is coordinated and what that
coordinator is, and is not, responsible for. This is a Stage 1 design document;
the service described here is illustrative and is not yet implemented in
production code.

An Application Service answers the question: **how is this use case executed?**

## 2. Definition

An Application Service coordinates one use case. It retrieves what is needed,
calls domain objects, persists the resulting aggregate, and dispatches the
events the aggregate recorded. It owns no business rules.

| Element | Appointment example |
|---|---|
| Service | `BookAppointmentService` |
| Use case | `BookAppointment` |
| Input | `BookAppointmentCommand` |
| Collaborators | Repository, `AppointmentAvailabilityService`, `EventDispatcher` |
| Result | The saved `Appointment` aggregate |

The FastAPI endpoint should be thin: parse and authorize the request, construct
the command, call the service, and translate the result. The booking workflow
must not live in the endpoint.

## 3. Responsibilities

The service coordinates these steps in order:

1. Build a `TimeSlot` from the command's start and end values.
2. Ask the Domain Service to check doctor availability.
3. Create the aggregate through `Appointment.book(...)`, which records
   `AppointmentBooked`.
4. Save the aggregate through the repository.
5. Dispatch the events recorded by the aggregate.
6. Return the booked appointment to the caller.

```text
FastAPI endpoint
      |
      v
BookAppointmentService.execute(command)
      |-- TimeSlot(start, end)
      |-- AppointmentAvailabilityService.ensure_available(...)
      |-- Appointment.book(...)          -> Appointment + AppointmentBooked
      |-- AppointmentRepository.save(...)
      '-- EventDispatcher.dispatch(...)
```

If a step fails, later steps do not run: a failed availability check performs
no save and dispatches no event.

## 4. Application Input: Command

A command is an explicit request to perform an operation. It is a data transfer
object, not a domain entity, and carries no behavior or business rules.

```python
from dataclasses import dataclass
from datetime import datetime
from uuid import UUID


@dataclass(frozen=True)
class BookAppointmentCommand:
    patient_id: UUID
    doctor_id: UUID
    start: datetime
    end: datetime
```

The command belongs to the application layer. Domain objects receive domain
types (`UUID`, `TimeSlot`), not the command itself.

## 5. Application Port: Event Dispatcher

```python
from typing import Protocol


class EventDispatcher(Protocol):
    def dispatch(self, event: object) -> None: ...
```

A protocol lets the application dispatch events without depending on a
particular messaging system. The concrete implementation belongs to
infrastructure and can be replaced in tests with a mock.

## 6. Illustrative Service

```python
from uuid import uuid4

from clinic_system.application.commands.book_appointment import (
    BookAppointmentCommand,
)
from clinic_system.application.ports.event_dispatcher import EventDispatcher
from clinic_system.domain.entities.appointment import Appointment
from clinic_system.domain.repositories.appointment_repository import (
    AppointmentRepository,
)
from clinic_system.domain.services.appointment_availability import (
    AppointmentAvailabilityService,
)
from clinic_system.domain.value_objects.time_slot import TimeSlot


class BookAppointmentService:
    def __init__(
        self,
        repository: AppointmentRepository,
        availability_service: AppointmentAvailabilityService,
        event_dispatcher: EventDispatcher,
    ) -> None:
        self.repository = repository
        self.availability_service = availability_service
        self.event_dispatcher = event_dispatcher

    def execute(self, command: BookAppointmentCommand) -> Appointment:
        time_slot = TimeSlot(start=command.start, end=command.end)
        self.availability_service.ensure_available(
            doctor_id=command.doctor_id,
            time_slot=time_slot,
        )
        appointment = Appointment.book(
            appointment_id=uuid4(),
            patient_id=command.patient_id,
            doctor_id=command.doctor_id,
            time_slot=time_slot,
        )
        self.repository.save(appointment)
        for event in appointment.collect_events():
            self.event_dispatcher.dispatch(event)
        return appointment
```

`collect_events()` is this stage's design for reading and clearing the
aggregate's recorded events. It is not part of the existing aggregate sketch
and must be added before this service can be implemented.

## 7. Responsibility Split

| Responsibility | Owner |
|---|---|
| Decide whether the doctor is available | Domain Service |
| Protect appointment state and transitions | Aggregate root |
| Validate a time interval | `TimeSlot` value object |
| Coordinate the booking steps | Application Service |
| Persist the aggregate | Repository implementation |
| Deliver notifications | Event handlers behind the dispatcher |
| Translate errors into HTTP responses | FastAPI endpoint |

The Application Service must not reimplement domain checks. If a rule answers
**what is allowed**, it belongs in an Aggregate or Domain Service; see
[domain-services.md](domain-services.md).

## 8. Unit Testing

The service is tested with mocks for all three collaborators, so no database,
broker, or web server is required.

| Test case | Expected result |
|---|---|
| Doctor available | `Appointment` is created and saved |
| Doctor unavailable | `DomainException`; no save, no dispatch |
| Invalid time slot | Validation error; no availability check, no save |
| Repository save fails | Operation fails; no event is dispatched |
| Successful booking | Exactly one `AppointmentBooked` event is dispatched |
| Availability check fails | Repository `save` is never called |

Ordering assertions are useful: availability is checked before creation, and
saving happens before dispatch.

## 9. Design Trade-offs

| Decision | Benefit | Trade-off |
|---|---|---|
| Application Service | Clear, testable use-case orchestration | Additional classes and files |
| Thin endpoint | HTTP concerns stay at the edge | Endpoint and service must be wired explicitly |
| Command DTO | Explicit use-case input | More DTOs to maintain |
| Port protocol | Infrastructure is replaceable | Adds another abstraction |

The small orchestration class is deliberate: it keeps the workflow independent
of FastAPI and gives each collaborator one reason to change.

## 10. Verification Checklist

- [ ] The service contains no business rule or state transition logic.
- [ ] The endpoint only parses, authorizes, delegates, and maps the result.
- [ ] Every collaborator is injected through an abstraction.
- [ ] No save occurs when availability or validation fails.
- [ ] Events are dispatched only after a successful save.
- [ ] Unit tests use mocks and run without infrastructure.

## 11. Related Documents

- [domain-services.md](domain-services.md) — the availability rule the service calls.
- [domain-events.md](domain-events.md) — the event the service dispatches.
- [appointment-booking-workflow.md](appointment-booking-workflow.md) — the complete flow.
- [aggregate-roots.md](aggregate-roots.md) — the aggregate behavior used by the service.
