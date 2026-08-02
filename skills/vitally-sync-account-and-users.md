---
name: Sync an account and its users
description: Create or upsert a Vitally Account, then attach its Users, keyed on your own externalId.
api: openapi/vitally-rest-openapi.yml
operations: [createAccount, updateAccount, createUser, listUsersForAccount]
---

# Sync an account and its users

Use this skill to push a customer account and its people into Vitally from your
source system, keeping records de-duplicated by `externalId`.

## Auth & host
- HTTP Basic auth: the Vitally REST API key is the **username**, password empty
  (`Authorization: Basic base64(<apiKey>:)`). See `authentication/vitally-authentication.yml`.
- Base URL: `https://{subdomain}.rest.vitally.io/resources` (US) or
  `https://rest.vitally-eu.io/resources` (EU).

## Steps
1. **Create/upsert the account** — `createAccount` (`POST /accounts`). Send
   `name` (required) and your `externalId`. Include `organizationId` if the
   account rolls up to an organization, and any `traits`. If the account already
   exists for that `externalId`, use `updateAccount` (`PUT /accounts/{accountId}`)
   with the Vitally `id`.
2. **Attach users** — for each person, `createUser` (`POST /users`) with `name`
   (required), `email`, your `externalId`, and the account association. Users may
   belong to multiple accounts.
3. **Verify** — `listUsersForAccount` (`GET /accounts/{accountId}/users`) to
   confirm the users are linked. This endpoint is paginated.

## Rules
- **externalId is the join key** — reusing it upserts rather than duplicating.
- **traits**: omitting a trait leaves it unchanged; set a trait to `null` to delete it.
- **Pagination**: list responses return `{ results, next }`; follow `next` until
  it is `null`. `limit` max/default is 100.
- **Errors**: `400` = fix the body; `401` = bad/missing auth header; `429` = over
  1000 req/min — back off using `RateLimit-Reset`. See `errors/vitally-problem-types.yml`.
- Unlinking a user from an account is **not** supported via REST — use the
  Analytics API unlink endpoint or the Vitally UI.
