---
name: Sync a venue's activity schedule
description: Retrieve a studio's classes, courses and workshops from the Eversports Provider API and keep a local schedule in sync.
api: graphql/eversports-provider-api.graphql
operations: [venues, activityGroups, activities]
auth: Bearer token
endpoint: https://provider-api.eversportsmanager.io/api/graphql
---

# Sync a venue's activity schedule (Eversports Provider API)

The Provider API is **read-only** GraphQL. Use it to pull activity schedules
(classes, trainings, workshops, courses, events, retreats, camps, educations)
for the venues your token is provisioned for.

## Auth
Send `Authorization: Bearer <TOKEN>` on every HTTP POST to
`https://provider-api.eversportsmanager.io/api/graphql`. Tokens are issued by
Eversports (support@eversports.com).

## Steps

1. **List the venues** you have access to with the `venues` query. Page with
   `first` + `after` (Relay cursor connections): read `edges[].node.id` and
   `pageInfo.hasNextPage` / `pageInfo.endCursor`.

2. **List activity groups** for a venue with `activityGroups(venueIds: [ID!], timeRange: TimeRangeInput)`
   to get the recurring definitions (title, sport, description, images).

3. **List activity instances** with `activities(venueIds: [ID!], timeRange: TimeRangeInput, first: Int, after: Cursor)`.
   Each `Activity` carries start/end, location, room, teacher, and availability
   (`bookable`, `limited`). Filter with `sportIds`, `teacherIds`, `isCancelled`,
   `hasOnlineStream` as needed.

4. **Page through** every connection until `pageInfo.hasNextPage` is false.

## Rules
- Recommended refresh cadence: ~1 hour for venues, ~30 minutes for the activity
  list (see `conventions/eversports-conventions.yml`).
- This API cannot create or modify activities — it is read-only.
- Errors arrive in the GraphQL top-level `errors[]` array (see `errors/eversports-error-codes.yml`).
