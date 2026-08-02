---
name: Find at-risk accounts and summarize activity
description: Scan Vitally accounts for churn-risk signals (health score, NPS, activity) and summarize.
api: openapi/vitally-rest-openapi.yml
operations: [listAccounts, listUsersForAccount, listNpsResponses, listNotes]
---

# Find at-risk accounts and summarize activity

Read-only workflow to surface accounts trending toward churn and gather the
context an agent needs to summarize them.

## Auth & host
- HTTP Basic auth with the REST API key as username (empty password).
- Base URL: `https://{subdomain}.rest.vitally.io/resources`.

## Steps
1. **Page through accounts** — `listAccounts` (`GET /accounts?status=active`).
   Follow `next` until `null`. For each account inspect `healthScore` (0-10),
   `npsScore` (-100..100), `nextRenewalDate`, and the
   `lastSeenTimestamp` / `lastInboundMessageTimestamp` recency fields.
2. **Flag risk** — treat low `healthScore`, negative `npsScore`, an imminent
   `nextRenewalDate`, or stale activity timestamps as risk indicators.
3. **Gather context** — for each flagged account: `listUsersForAccount`
   (`GET /accounts/{accountId}/users`) for the people, `listNpsResponses`
   (`GET /npsResponses`) for verbatim feedback, and `listNotes` (`GET /notes`)
   for recent CS notes.
4. **Summarize** — produce a short risk summary per account (why at risk, who to
   contact, suggested next step).

## Rules
- This is a **read-only** skill — do not create or mutate records.
- Respect pagination (`{ results, next }`, `limit` max 100) and the 1000/min
  rate budget; back off on `429` using `RateLimit-Reset`.
- Sort with `sortBy=createdAt` when you need a stable full sweep of
  frequently-updated data.
