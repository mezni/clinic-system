
# Functional Requirements

## 1. Patient Management

| ID | Requirement | Actor | Priority |
|---|---|---|---|
| PAT-001 | Register patient | Receptionist | Must |
| PAT-002 | Generate patient ID | System | Must |
| PAT-003 | Search patients | Receptionist | Must |
| PAT-004 | View patient profile | Authorized staff | Must |
| PAT-005 | Update patient demographics | Receptionist | Must |
| PAT-006 | Record contact information | Receptionist | Must |
| PAT-007 | Deactivate patient | Administrator | Should |
| PAT-008 | Detect potential duplicates | System | Should |

## 2. Practitioner Management

| ID | Requirement | Actor | Priority |
|---|---|---|---|
| PRA-001 | Register practitioner | Administrator | Must |
| PRA-002 | Update practitioner | Administrator | Must |
| PRA-003 | Assign specialty | Administrator | Must |
| PRA-004 | Define availability | Administrator | Must |
| PRA-005 | View practitioner schedule | Staff | Must |
| PRA-006 | Deactivate practitioner | Administrator | Should |

## 3. Appointment Management

| ID | Requirement | Actor | Priority |
|---|---|---|---|
| APT-001 | Create appointment | Receptionist | Must |
| APT-002 | Assign practitioner | Receptionist | Must |
| APT-003 | Check availability | System | Must |
| APT-004 | Prevent conflicting appointments | System | Must |
| APT-005 | Reschedule appointment | Receptionist | Must |
| APT-006 | Cancel appointment | Receptionist | Must |
| APT-007 | Check patient in | Receptionist | Must |
| APT-008 | View daily schedule | Staff | Must |

## 4. Encounter Management

| ID | Requirement | Actor | Priority |
|---|---|---|---|
| ENC-001 | Start encounter | Physician | Must |
| ENC-002 | Link patient | System | Must |
| ENC-003 | Assign practitioner | System | Must |
| ENC-004 | Record clinical notes | Physician | Must |
| ENC-005 | Review patient history | Physician | Must |
| ENC-006 | Complete encounter | Physician | Must |
| ENC-007 | View encounter history | Staff | Must |

## 5. Clinical Records

| ID | Requirement | Actor | Priority |
|---|---|---|---|
| CLI-001 | Record diagnosis | Physician | Must |
| CLI-002 | Record allergies | Physician/Nurse | Must |
| CLI-003 | Record vital signs | Nurse/Physician | Must |
| CLI-004 | View medical history | Staff | Must |
| CLI-005 | Update observations | Authorized staff | Must |
| CLI-006 | View historical observations | Staff | Must |

## 6. Medication Management

| ID | Requirement | Actor | Priority |
|---|---|---|---|
| MED-001 | Create prescription | Physician | Must |
| MED-002 | Record medication details | Physician | Must |
| MED-003 | View medication history | Staff | Must |
| MED-004 | Discontinue prescription | Physician | Must |
| MED-005 | Record dispensing | Pharmacist | Should |
| MED-006 | Review allergies | Physician | Must |

## 7. Laboratory Management

| ID | Requirement | Actor | Priority |
|---|---|---|---|
| LAB-001 | Create laboratory request | Physician | Must |
| LAB-002 | View pending requests | Lab Technician | Must |
| LAB-003 | Record results | Lab Technician | Must |
| LAB-004 | Finalize report | Lab Technician | Must |
| LAB-005 | View historical results | Staff | Must |
| LAB-006 | Flag abnormal results | System | Should |

## 8. Organization Management

| ID | Requirement | Actor | Priority |
|---|---|---|---|
| ORG-001 | Register clinic | Administrator | Must |
| ORG-002 | Manage departments | Administrator | Should |
| ORG-003 | Manage rooms | Administrator | Must |
| ORG-004 | Assign practitioners to locations | Administrator | Should |

## 9. User and Access Management

| ID | Requirement | Actor | Priority |
|---|---|---|---|
| IAM-001 | Create user | Administrator | Must |
| IAM-002 | Authenticate users | System | Must |
| IAM-003 | Assign roles | Administrator | Must |
| IAM-004 | Enforce permissions | System | Must |
| IAM-005 | Deactivate user | Administrator | Must |
| IAM-006 | Restrict patient data access | System | Must |

## 10. Audit Management

| ID | Requirement | Actor | Priority |
|---|---|---|---|
| AUD-001 | Record login events | System | Must |
| AUD-002 | Record patient access | System | Must |
| AUD-003 | Record clinical modifications | System | Must |
| AUD-004 | Record appointment changes | System | Must |
| AUD-005 | Search audit history | Administrator | Should |

## 11. Requirement Management

All requirements must be traceable to:
- Business use cases
- Domain model
- Application use cases
- API endpoints
- Automated tests

Requirements must be validated before implementation.

