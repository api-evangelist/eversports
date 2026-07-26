---
name: List sessions and make a reservation
description: Use the Eversports Aggregator API to discover venue sessions and place, cancel, or check in a reservation for a user.
api: graphql/eversports-aggregator.graphql
operations: [venues, sessions, makeReservation, cancelReservation, checkIn]
auth: Bearer token
endpoint: https://aggregator.eversports.io/v1/graphql
---

# List sessions and make a reservation (Eversports Aggregator API)

The Aggregator API is GraphQL with both read queries and reservation mutations.
Build and test against `https://aggregator-test.eversports.io/v1/graphql` before
switching to `https://aggregator.eversports.io/v1/graphql`.

## Auth
Send `Authorization: Bearer <TOKEN>` on every HTTP POST. Tokens are issued by
Eversports.

## Steps

1. **Discover venues** with the `venues(first: Int!, after: Cursor)` query
   (Relay cursor pagination). Optionally map your external id with the
   `setAggregatorVenueId(aggregatorVenueId, eversportsVenueId)` mutation.

2. **List sessions** with
   `sessions(first: Int!, venueId: ID, locationId: ID, timeRangeInDays: Float, isCancelled: Boolean, updatedAfter: DateTime)`.
   `first` is capped at 100 (default 50). Each `Session` exposes start/end,
   `availableSpots`, class, sport, room, teachers.

3. **Reserve** a spot with
   `makeReservation(sessionId: ID!, user: UserInput!, checkedIn: Boolean)`.
   Returns the created `Reservation`.

4. **Check in** with `checkIn(reservationId: ID!)`, or **cancel** with
   `cancelReservation(reservationId: ID!, isLateCancellation: Boolean)`.

5. **Reconcile** after a session ends by listing `reservations(sessionId: ID!, first: Int!)`
   (~1 day after the session).

## Rules
- No idempotency key is documented — de-duplicate reservation attempts client-side
  (see `conventions/eversports-conventions.yml`).
- Pagination is Relay cursor connections; page until `pageInfo.hasNextPage` is false.
- The spec is under active development and may change (`lifecycle/eversports-lifecycle.yml`).
- Errors arrive in the GraphQL `errors[]` array (`errors/eversports-error-codes.yml`).
