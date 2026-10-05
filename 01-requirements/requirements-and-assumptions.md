# Requirements & Assumptions

## Problem Statement
SALESTORM is preparing for a limited-stock flash sale. The central scenario is **10,000 simultaneous purchase attempts competing for 100 available units**. The system must protect inventory, process payments safely, maintain a correct order lifecycle, and recover from failures.

## Functional Requirements
- Product discovery and flash-sale access.
- Buy Now / checkout.
- Temporary inventory reservation with expiry and release.
- No overselling.
- Duplicate reservation prevention.
- Safe, idempotent payment processing.
- Order creation and lifecycle tracking.
- Recovery when payment succeeds but Order Service fails.
- Fulfilment, shipment, notification and delivery tracking.
- Auditability of critical inventory, payment and order transitions.

## Non-Functional Requirements
- **Scalability:** horizontally scale during peak traffic.
- **Consistency:** inventory correctness is a strict guarantee.
- **Availability:** isolate failures where possible.
- **Reliability:** successful purchases must survive temporary downstream failures.
- **Performance:** keep browsing fast while protecting critical writes.
- **Security:** authentication, authorization, HTTPS, validation, rate limiting, secure payment handling, audit logs and secrets management.
- **Observability:** metrics, structured logs, tracing and alerts.

## Strict Guarantees
- Stock never becomes negative.
- 100 units never produce more than 100 successful sales.
- One logical purchase request does not create duplicate reservations.
- One logical payment does not create duplicate charges.
- Replayed processing does not create duplicate orders.

## Assumptions
- Flash-sale inventory is known before the sale begins.
- Reservations have an expiry time.
- Reserved quantity is unavailable to other customers until confirmed or released.
- Failed/expired reservations return stock to the available pool.
- Customers may retry because of slow networks or double-clicks.
- Payment may return success, failure or timeout/unknown.
- External calls and messages can be repeated, so repeated processing must be safe.
- Product browsing may use slightly stale cached data, but final allocation must use authoritative inventory state.
- Services and networks may fail temporarily, so workflows must be recoverable.

## Proposed Business Workflow

```text
Customer
  ↓
Waiting Room / Traffic Control
  ↓
Controlled Admission
  ↓
Buy Request
  ↓
Purchase Queue
  ↓
Duplicate Check
  ↓
Inventory Reservation
  ↓
Payment
  ├─ Failure/Timeout → Release or Reconcile
  └─ Success → Confirm Sale → Create Order
                              ↓
                 Fulfilment → Shipment → Notification → Delivery
```

## Critical Failure Scenarios
- Two customers compete for the last unit.
- Duplicate Buy requests.
- Reservation expiry or abandonment.
- Payment failure or timeout.
- Duplicate payment request.
- Payment succeeds but Order Service is unavailable.
- Database or payment-gateway failure.
- Inventory reaches zero.
- Traffic increases by 50×.

## Success Criteria
The design must demonstrate that inventory cannot be oversold, expired/failed reservations are released, duplicate business transactions are prevented, successful purchases eventually reach a valid order state, recovery paths are defined, and the architecture has a defensible scaling strategy.
