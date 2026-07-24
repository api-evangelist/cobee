---
name: Reconcile consumptions and close a payroll cycle
description: Pull employee benefit consumptions for the open payroll cycle, then close it and open the next.
api: openapi/cobee-public-api-openapi-original.json
operations: [getPayrollCycles, getCompanyConsumptions, getEmployeeConsumptions, closePayrollCycle]
---

# Reconcile consumptions and close a payroll cycle

Authenticate first (see `cobee-authenticate.md`) and have a `companyId`.

## Steps

1. **Find the open cycle** — `GET /companies/{companyId}/payroll-cycles`
   (`getPayrollCycles`). The cycle with a `null` end date is the one currently
   open; keep its `payrollCycleId`.
2. **Pull company consumptions** — `GET /companies/{companyId}/consumptions`
   (`getCompanyConsumptions`), filtering by `payrollCycleId` and optionally
   `categories` / `groupBy`; use the `Accept` header or `format` query param to
   choose the report representation. For a single person use
   `GET /companies/{companyId}/employees/{employeeId}/consumptions`
   (`getEmployeeConsumptions`).
3. **Reconcile** against payroll before closing — this feeds the monthly report
   flow the docs describe.
4. **Close the cycle** — `POST /companies/{companyId}/payroll-cycles/{payrollCycleId}`
   (`closePayrollCycle`) to close the current cycle and open the next one.

## Rules

- Closing is a state transition: a `409 Conflict` means the cycle is already
  closed — re-fetch cycles rather than retrying.
- There is no idempotency-key; treat close as non-repeatable and confirm state
  with `getPayrollCycles` after.
- To simulate consumptions in **staging only**, use
  `POST /companies/{companyId}/payroll-cycles/{payrollCycleId}/employees/{employeeId}/consumptions`
  (`createEmployeeConsumption`) — it is sandbox-only (see `sandbox/cobee-sandbox.yml`).
