# Clinic Domain Documentation

## 1. Purpose

This directory documents the business domain of `clinic-system`: a system for
managing patients, appointments, clinical care, staff, and billing for a
primary-care clinic.

The documentation establishes a shared business vocabulary before domain
models are implemented. It is the reference for developers, domain experts,
stakeholders, and users when they discuss clinic operations.

## 2. Why This Documentation Exists

In Domain-Driven Design (DDD), developers and domain experts should use the
same terminology. Similar words can describe different business concepts:

| Term | Possible interpretation |
|---|---|
| Patient | Person receiving medical care |
| Customer | Person paying for services |
| Client | Person registered at the clinic |
| Appointment | Scheduled visit |
| Visit | Actual patient encounter |
| Consultation | Medical interaction |

Without explicit definitions, these terms lead to incorrect entities,
relationships, and business rules. This documentation fixes their meaning
within the clinic domain.

## 3. Documents

| Document | Purpose |
|---|---|
| [glossary.md](glossary.md) | Business terminology, core concepts, and distinctions between similar terms |
| [ubiquitous-language.md](ubiquitous-language.md) | Business statements, domain interpretations, and naming conventions for future code |

## 4. Scope

The initial vocabulary covers five areas:

1. Patient management
2. Appointment management
3. Clinical management
4. Staff management
5. Billing and payments

The glossary is the authoritative vocabulary for these areas. Requirements in
[`../requirements/`](../requirements/) describe what the system must do; this
directory defines the words used to describe it. Where earlier requirements
use `Practitioner`, the specific clinical role discussed here is `Doctor`.

## 5. DDD Principle

> Similar words do not automatically represent the same domain object.

For example, an `Appointment` (a planned visit) and an `Encounter` (an actual
clinical interaction) are separate concepts and must not be represented by the
same entity.

## 6. Next Step

Step 07 — Domain Model: Entities, Value Objects & Domain Rules. That step will
identify which concepts need identity, which are defined by their values, and
which business rules belong inside the domain.
