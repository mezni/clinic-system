# Domain Services

## 1. Purpose

Define when business logic belongs in a Domain Service instead of an Entity, a
Value Object, or an Application Service. This is a Stage 1 design document; the
service described here is illustrative and is not yet implemented in production
code.

A Domain Service contains domain logic that does not naturally belong to one
Entity or Value Object. It answers the question: **what does the business
allow?**

## 2. Definition

| Element | Appointment example |
|---|---|
| Name | `AppointmentAvailabilityService` |
| Type | Domain Service |
| Operation | `ensure_available(doctor_id, time_slot)` |
| Input | Doctor identity and a requested `TimeSlot` |
| Output | `None` when available; `DomainException` when unavailable |
| Dependency | `AppointmentRepository` protocol for existing appointments |

The operation involves a doctor, a time slot, and existing appointments. No
single `Appointment` owns that knowledge, so the rule is placed in a service in
the domain layer.

## 3. Why Availability Is Not an Appointment Method

`Appointment` protects its own invariants (INV-01 to INV-07 in
[domain-invariants.md](domain-invariants.md)). Those rules concern one aggregate
instance. Availability compares a requested slot against **other appointments
of the same doctor**, which live outside the aggregate being created.

| Placement | Assessment |
|---|---|
| `TimeSlot` | Rejected: a slot cannot know about a doctor's calendar |
| `Appointment` | Rejected: construction should not query a repository or scan siblings |
| API endpoint | Rejected: mixes HTTP concerns with a business decision |
| Domain Service | Accepted: expresses and enforces a cross-appointment business rule |
| Application Service | Rejected as owner: it may coordinate the check but must not decide the rule |

This follows the aggregate boundary in [aggregates.md](aggregates.md): the
schedule is a read model over many appointments, not part of one aggregate.

## 4. Boundaries

A Domain Service must

- contain business decisions,
- use domain terminology (`doctor_id`, `time_slot`, `ensure_available`),
- depend only on domain types and abstractions such as a repository protocol,
- be independent of HTTP frameworks, response objects, and concrete databases.

It must not

- build API responses or raise HTTP errors,
- implement SQL or call ORM models directly,
- manage transactions or publish events,
- coordinate a whole use case.

Persistence details remain behind the repository contract in section 5 of
[appointment-booking-workflow.md](appointment-booking-workflow.md).

## 5. Illustrative Contract

```python
from uuid import UUID

from clinic_system.domain.exceptions.domain_exception import DomainException
from clinic_system.domain.repositories.appointment_repository import (
    AppointmentRepository,
)
from clinic_system.domain.value_objects.time_slot import TimeSlot


class AppointmentAvailabilityService:
    def __init__(self, repository: AppointmentRepository) -> None:
        self.repository = repository

    def ensure_available(
        self,
        doctor_id: UUID,
        time_slot: TimeSlot,
    ) -> None:
        conflict = self.repository.has_conflicting_appointment(
            doctor_id=doctor_id,
            time_slot=time_slot,
        )
        if conflict:
            raise DomainException(
                "Doctor is not available during this time slot"
            )
```

`ensure_available` signals success by returning normally. There is no boolean
result for the application service to reinterpret; the rule has one business
outcome. The domain exception is translated to an API response at the edge of
the system, not here.

## 6. Rule Placement Summary

| Question | Owner |
|---|---|
| Is the time slot well formed? | `TimeSlot` value object |
| Are IDs present and valid? | `Appointment` aggregate root |
| Is the doctor free at that time? | `AppointmentAvailabilityService` |
| Which steps run to book an appointment? | Application Service |
| What happened after booking? | Domain Event |
| How is the appointment stored? | Repository implementation |

A useful rule: if the answer is **what is allowed by the business**, look first
at Aggregates and Domain Services.

## 7. Unit Testing

Tests use a mock or in-memory repository and exercise the service directly,
without FastAPI, PostgreSQL, or an event dispatcher.

| Test case | Expected result |
|---|---|
| Repository reports no conflict | `ensure_available` returns without error |
| Repository reports a conflict | `DomainException` is raised |
| Conflict message | Uses domain wording, not HTTP status text |
| Adjacent slots | No conflict is reported by the repository query |
| Cancelled appointment exists at the same time | No conflict, because cancelled appointments are excluded |

The service makes one availability query and performs no writes. Tests must not
depend on a real database.

## 8. Design Trade-offs

| Decision | Benefit | Trade-off |
|---|---|---|
| Domain Service | Centralizes a cross-entity business rule | Can become a dumping ground for unrelated rules |
| Repository protocol | Domain stays infrastructure-independent | Adds an abstraction to maintain |
| Exception on violation | Business rule is explicit | Caller must translate to an API response |
| Single-purpose method | Easy to test in isolation | May need more services as rules grow |

Keep the service narrow. If it starts validating passwords, formatting
responses, or opening transactions, that logic belongs in another layer.

## 9. Verification Checklist

- [ ] The service contains a business decision, not orchestration.
- [ ] It has no HTTP, ORM, or concrete database dependency.
- [ ] Its language comes from the ubiquitous language in
      [ubiquitous-language.md](ubiquitous-language.md).
- [ ] A conflict raises `DomainException` and performs no writes.
- [ ] It has unit tests for both the available and unavailable cases.

## 10. Related Documents

- [application-services.md](application-services.md) — coordinates the availability check.
- [domain-events.md](domain-events.md) — records the outcome of a successful booking.
- [appointment-booking-workflow.md](appointment-booking-workflow.md) — the full use case.
- [domain-rules.md](domain-rules.md) — broader rule placement.
