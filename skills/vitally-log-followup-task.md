---
name: Log a follow-up task on an account
description: Find a Vitally account, resolve the owner, and create an assigned follow-up task.
api: openapi/vitally-rest-openapi.yml
operations: [listAccounts, getAccount, listAdmins, createTask]
---

# Log a follow-up task on an account

Create an actionable follow-up Task attached to a customer account and assigned
to a team member.

## Auth & host
- HTTP Basic auth with the REST API key as username (empty password).
- Base URL: `https://{subdomain}.rest.vitally.io/resources`.

## Steps
1. **Locate the account** — `listAccounts` (`GET /accounts`, paginated, filter by
   `status`) or `getAccount` (`GET /accounts/{accountId}`) if you already hold the
   Vitally `id`. Note the `id` and the `accountOwnerId` / `csmId`.
2. **Resolve the assignee** — `listAdmins` (`GET /admins`) to map the owner/CSM to
   a Vitally Admin `id` for `assignedToId`.
3. **Create the task** — `createTask` (`POST /tasks`). Required: `name` and
   `accountId`. Optional: `description` (restricted HTML), `dueDate`,
   `assignedToId`, `tags`, and custom `traits` (addressed by generated key,
   e.g. `vitally.custom.nextStep`).

## Rules
- `description` HTML is limited to an allow-list of tags
  (`a, img, p, div, br, ul, ol, li, b, u, i, strong, em, code`); other markup is
  stripped. See `conventions/vitally-conventions.yml`.
- No idempotency-key header exists; pass a stable `externalId` on the task to
  avoid duplicates across retries.
- Handle `429` by honoring `RateLimit-Reset`; writes may cost more than one unit
  of the 1000/min budget.
