# SALESTORM — High-Scale Flash Sale System Design

> **Design first. Use AI to accelerate. Defend every decision.**

SALESTORM is a design-first solution for a high-volume e-commerce flash sale where **10,000 customers compete for only 100 available units**.

## Core Problem
Design a system that prevents overselling and duplicate reservations/orders, processes payments safely, maintains a correct order lifecycle, scales under peak traffic, and recovers from service failures.

## Proposed Workflow

```text
Customers
   ↓
Traffic Control / Virtual Waiting Room
   ↓
Controlled Admission
   ↓
Product / Sale
   ↓
Buy Now
   ↓
Purchase Queue
   ↓
Duplicate Request Check
   ↓
Inventory Allocation
   ↓
Temporary Reservation
   ↓
Checkout
   ↓
Payment
   ├── Failure / Timeout → Release Reservation → Next Eligible Customer
   └── Success
          ↓
     Confirm Inventory
          ↓
       Create Order
          ↓
       Fulfilment
          ↓
        Shipment
          ↓
      Notification
          ↓
        Delivery
```

## Why This Approach?

| Problem | Design Response |
| --- | --- |
| Huge traffic spike | Controlled admission / waiting room |
| Many buyers competing at once | Purchase queue |
| Only 100 units available | Concurrency-safe reservation |
| Customer needs time to pay | Temporary reservation |
| Payment fails or customer abandons | Reservation expiry and release |
| Repeated Buy/Pay requests | Idempotency / duplicate protection |
| Payment succeeds but Order Service fails | Recovery and reconciliation |
| Downstream failures | Retry, circuit breaker and failure handling |

## Three Core Guarantees

1. **Traffic safety** — sudden demand should not collapse the platform.
2. **Inventory correctness** — 100 units must never result in more than 100 successful sales.
3. **Transaction reliability** — customers should not be charged twice or lose a successful purchase because of a temporary service failure.

## Challenge Scenario

- Stock: **100 units**
- Concurrent users: **10,000**
- Payment success: **95%**
- Payment failure: **5%**
- Duplicate requests: **2%**
- Order Service unavailable: **30 seconds**

The design must also explain what changes when traffic increases by 50× and how database/payment-gateway failures are handled.

## Reservation Lifecycle

```text
AVAILABLE → RESERVED → PAYMENT_PENDING → CONFIRMED → SOLD

RESERVED → PAYMENT_FAILED → RELEASED
RESERVED → TIMEOUT → RELEASED
```

## Repository Structure

```text
01-requirements/
02-hld/
03-lld/
04-database/
05-api/
06-solid/
07-design-patterns/
08-scalability-reliability/
09-security-observability/
10-adr/
11-ai-assisted-validation/
12-presentation/
```

## Design Principle

Implementation is optional and supports the architecture rather than replacing it. Any prototype or simulation should validate a critical assumption such as concurrency safety, idempotency, reservation expiry, or recovery.

Every major decision should answer:

> **Why is this component needed, what problem does it solve, and what trade-off does it introduce?**
