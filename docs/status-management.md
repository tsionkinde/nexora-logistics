# Nexora Logistics - Status Management

## 1. Purpose

This document defines the statuses, meanings, and allowed transitions for orders, shipments, and deliveries.

The purpose is to ensure that logistics operations follow a consistent workflow and prevent invalid status updates.

The backend must validate every status transition before saving it to the database.

## 2. Order Statuses

An order represents a customer's request for goods or services.

### 2.1 Available Statuses

| Status             | Description                                       |
| ------------------ | ------------------------------------------------- |
| PENDING            | The order has been created but not yet confirmed. |
| CONFIRMED          | The order has been reviewed and approved.         |
| PROCESSING         | The order is being prepared.                      |
| READY_FOR_DISPATCH | The order is ready for shipment.                  |
| COMPLETED          | The order has been successfully delivered.        |
| CANCELLED          | The order has been cancelled.                     |

### 2.2 Allowed Order Transitions

```text
PENDING
   |
   v
CONFIRMED
   |
   v
PROCESSING
   |
   v
READY_FOR_DISPATCH
   |
   v
COMPLETED
```
