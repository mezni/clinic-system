# Value Objects

## 1. Purpose

Identify concepts that are defined entirely by their values. A value object
has no independent identity: when its values change, it is replaced.

This document is an initial design classification, not production code.

## 2. Classification Rule

| Criterion | Value object |
|---|---|
| Identity | None required |
| Equality | Compared by attribute values |
| Mutability | Immutable after creation |
| Validation | Performed in the constructor or factory |
| Lifecycle | None; replace instead of changing |
| Reuse | Safe to share because it cannot change |

A concept that must be tracked over time is an entity, even if it has value-like
attributes. See [entities.md](entities.md).

## 3. Value Object Inventory

| Value object | Attributes | Belongs to | Why it is a value object |
|---|---|---|---|
| `Patient ID` | Identifier value | `Patient` | Names a patient; has no behavior or lifecycle of its own |
| `Appointment ID` | Identifier value | `Appointment` | Identifies an appointment without holding appointment state |
| `Encounter ID` | Identifier value | `Encounter` | Identifies an encounter without holding clinical state |
| `Doctor ID` | Identifier value | Appointment, Encounter | Reference to a staff member acting as doctor |
| `Invoice ID` / `Payment ID` | Identifier value | Invoice, Payment | Identify billing records |
| `Contact Information` | Phone, email, address | `Patient` | Defined by its values |
| `Appointment Slot` | Doctor ID, start, end | `Appointment` | A bookable period; same values mean the same slot |
| `Appointment Status` | Status value | `Appointment` | Lifecycle state descriptor |
| `Encounter Status` | Status value | `Encounter` | Lifecycle state descriptor |
| `Availability` | Start, end, recurrence | `StaffMember` | A period when a staff member can work |
| `Working Schedule` | Collection of `Availability` | `StaffMember` | Describes working hours, not a separate thing |
| `Role` | Role name, permissions | `StaffMember` | A responsibility label |
| `Vital Signs` | Blood pressure, temperature, pulse, recorded at | `Encounter` | Measurements defined by their values |
| `Consultation` | Assessment, clinical impression | `Encounter` | Medical assessment content recorded during care |
| `Clinical Note Text` | Text, author, recorded at | `Clinical Note` | Note content; the note itself is a local entity |
| `Diagnosis Code` | Code system, code, description | `Diagnosis` | Coded clinical concept |
| `Service Fee` | Amount, currency | `Invoice Item` | A charge amount |
| `Money` | Amount, currency | Billing | Arithmetic and comparison of amounts |
| `Outstanding Balance` | Money value | Derived from `Invoice`, `Payment`, `Refund` | Calculated result, not stored independently |

## 4. Detailed Definitions

### Identifiers

IDs are value objects that wrap an identifier string. Two IDs are equal only
when their values match. They are immutable and cannot be constructed from an
empty value.

Examples: `PAT-1001`, `APT-1001`, `ENC-1001`, `DOC-1001`.

### Contact Information

Groups phone, email, and address. Validation belongs here: invalid phone or
email values must not enter the domain. Changing a phone number replaces the
whole `Contact Information` value on `Patient`.

### Appointment Slot

Defined by a doctor identifier plus a time interval. It is the value reserved
by an appointment.

- Two slots with the same doctor and interval are equal.
- A slot must have a start before its end.
- Overlapping slots are a conflict; the rule is enforced by the Appointment
  aggregate with support from the availability context.
- An empty slot is available; once reserved, the reservation is held by an
  `Appointment`, not by the slot.

### Availability and Working Schedule

`Availability` describes when a staff member can work. `Working Schedule` is
the collection of those periods. Neither creates an appointment; they describe
what may be booked.

### Lifecycle States

`Appointment Status` and `Encounter Status` are value objects representing
allowed states. They carry their own transition rules so invalid states cannot
be constructed.

Appointment values:

- `SCHEDULED`
- `CONFIRMED`
- `CANCELLED`
- `COMPLETED`
- `NO_SHOW`

Encounter values:

- `IN_PROGRESS`
- `COMPLETED`

### Clinical Values

`Vital Signs`, `Consultation`, `Diagnosis Code`, and `Clinical Note Text`
describe clinical facts. They are immutable: correcting a note creates a new
note content value on the same `Clinical Note` entity, not a new encounter.

`Medical History` is not a value object. It is a projection assembled from
encounters and clinical records.

### Money Values

`Service Fee`, `Payment` amount, `Refund` amount, and `Outstanding Balance`
use a `Money` value object containing amount and currency. Money arithmetic
must not combine different currencies.

`Outstanding Balance` is derived:

```text
outstanding = invoice total - payments + refunds
```

It is never edited directly.

## 5. Design Rules

- Make value objects immutable.
- Validate at construction; an invalid value object must not exist.
- Compare by value, not object identity.
- Do not give a value object a database row of its own unless persistence
  requires it; persistence representation is not domain identity.
- Do not add behavior that changes another aggregate.
- Replace a value object when any attribute changes.
- When a concept gains its own lifecycle or history, reclassify it as an
  entity.

## 6. Illustration Only

```python
@dataclass(frozen=True)
class AppointmentSlot:
    doctor_id: str
    start: datetime
    end: datetime

    def __post_init__(self):
        if self.start >= self.end:
            raise ValueError("slot start must be before end")


@dataclass(frozen=True)
class Money:
    amount: Decimal
    currency: str

    def __post_init__(self):
        if self.amount < 0:
            raise ValueError("money amount cannot be negative")
```

The final implementation will follow the project's architecture and testing
conventions.
