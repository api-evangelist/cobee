---
name: Onboard and invite an employee
description: Register a new employee under a benefit model and send their platform invitation.
api: openapi/cobee-public-api-openapi-original.json
operations: [getBenefitModels, registerEmployee, inviteEmployee, getEmployee]
---

# Onboard and invite an employee

Authenticate first (see `cobee-authenticate.md`). You need a `companyId` from
`GET /companies`.

## Steps

1. **Pick a benefit model** — `GET /companies/{companyId}/benefit-models`
   (`getBenefitModels`). Choose the model the employee should be enrolled in and
   keep its `id` (used as `modelId`).
2. **Register the employee** — `POST /companies/{companyId}/employees`
   (`registerEmployee`) with the employee payload: identity (`legalId`,
   `internalId`, name, email), the `modelId`, and any monetary limits /
   workday configuration. On success you get a `201` with the new employee `id`.
3. **Invite the employee** — `POST /companies/{companyId}/employees/{employeeId}/invite`
   (`inviteEmployee`) to send the platform invitation email.
4. **Verify** — `GET /companies/{companyId}/employees/{employeeId}`
   (`getEmployee`) to confirm the stored record and state.

## Rules

- Handle `409 Conflict` on register/invite — the employee may already exist or
  already be invited; reconcile rather than blindly retrying (no idempotency-key
  contract exists, see `conventions/cobee-conventions.yml`).
- `422` on invite means the employee is not in an invitable state.
- `403`/`404` mean your token cannot access that company or the company/model
  does not exist.
