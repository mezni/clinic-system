
# Stakeholders and User Roles

## 1. Stakeholders

| Stakeholder | Responsibility |
|---|---|
| Clinic Owner | Business oversight |
| Clinic Administrator | System administration |
| Receptionist | Patient registration and scheduling |
| Physician | Clinical consultations |
| Nurse | Patient preparation and observations |
| Laboratory Technician | Laboratory workflows |
| Pharmacist | Medication workflows |
| Patient | Personal healthcare access |
| IT Administrator | Technical infrastructure |

## 2. Application Roles

### ADMIN
- Manage application users
- Assign roles
- Configure clinic settings
- Review audit logs

### RECEPTIONIST
- Register patients
- Search patients
- Update demographic information
- Schedule appointments
- Manage appointment check-in

### PHYSICIAN
- View authorized patient records
- Manage clinical encounters
- Record diagnoses
- Create prescriptions
- Create laboratory requests
- Complete consultations

### NURSE
- View authorized patient records
- Record vital signs
- Assist with encounters

### LAB_TECHNICIAN
- View laboratory orders
- Process laboratory tests
- Record results
- Finalize laboratory reports

### PHARMACIST
- Review medication requests
- Manage medication dispensing workflows

### PATIENT
- View personal profile
- View appointments
- Request appointment changes

## 3. Authorization Model

The initial system uses Role-Based Access Control (RBAC).

A user can have multiple roles.
A role contains multiple permissions.
Permissions control actions on resources.

## 4. Security Principles

- Least privilege
- Deny by default
- Separation of duties
- Patient-level access restrictions
- Audit sensitive operations
- Protect personal health information

## 5. Assumptions

- One clinic initially.
- Multiple users and practitioners.
- Users may have multiple roles.
- Patient access is restricted to their own records.
- Administrative access does not automatically imply
  unrestricted clinical access.

