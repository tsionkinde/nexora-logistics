# Nexora Logistics – Database Design

## 1. Overview

Nexora Logistics uses PostgreSQL as its relational database and Prisma ORM for database access.

The database is designed to support customer orders, shipments, delivery attempts, driver and vehicle assignments, proof of delivery, and shipment tracking.

The design preserves operational history and supports multiple delivery attempts for a single shipment.

## 2. Main Entities

| Entity             | Responsibility                                 |
| ------------------ | ---------------------------------------------- |
| User               | Stores login credentials and user roles        |
| Customer           | Stores customer profile information            |
| Address            | Stores customer and delivery addresses         |
| Order              | Represents a customer's order                  |
| OrderItem          | Stores products or items belonging to an order |
| Shipment           | Represents a shipment created from an order    |
| ShipmentItem       | Connects order items to shipments              |
| Driver             | Stores driver information                      |
| Vehicle            | Stores vehicle information                     |
| Delivery           | Represents one delivery attempt for a shipment |
| DeliveryAssignment | Records driver and vehicle assignments         |
| ProofOfDelivery    | Stores evidence of successful delivery         |
| TrackingEvent      | Stores shipment tracking history               |
| Notification       | Stores notifications sent to users             |

## 3. Entity Relationships

- A User may have one Customer profile.
- A User may have one Driver profile.
- A Customer can create multiple Orders.
- An Order can contain multiple OrderItems.
- An Order can have multiple Shipments.
- A Shipment can contain multiple ShipmentItems.
- A Shipment can have multiple Delivery attempts.
- A Delivery can have multiple DeliveryAssignments over time.
- A Delivery can have one ProofOfDelivery.
- A Shipment can have multiple TrackingEvents.
- A User can receive multiple Notifications.

## 4. Important Relationship: Shipment and Delivery

A Shipment and a Delivery are not the same thing.

A Shipment represents the goods being transported.

A Delivery represents an attempt to deliver those goods to the customer.

For example:

1. Shipment SHP-001 is created.
2. A driver attempts delivery on Monday.
3. The customer is unavailable, so the delivery attempt is marked FAILED.
4. A new delivery attempt is scheduled for Tuesday.
5. Another driver is assigned.
6. The second attempt succeeds.
7. Proof of Delivery is recorded.

Both delivery attempts remain in the database.

Therefore, the relationship is:

**One Shipment → Many Delivery Attempts**

The `Delivery` table must have a `shipmentId` foreign key that is NOT unique.

## 5. Proposed Entity Fields

The following are the proposed core fields. Additional timestamps, indexes, constraints, and audit fields may be added during implementation.

### User

- id (Primary Key)
- fullName
- email (Unique)
- passwordHash
- role
- isActive
- createdAt
- updatedAt

### Customer

- id (Primary Key)
- userId (Foreign Key, Unique)
- phone
- createdAt
- updatedAt

### Address

- id (Primary Key)
- customerId (Foreign Key)
- label
- street
- city
- region
- postalCode
- country
- createdAt
- updatedAt

### Order

- id (Primary Key)
- orderNumber (Unique)
- customerId (Foreign Key)
- deliveryAddressId (Foreign Key)
- status
- totalAmount
- createdAt
- updatedAt

### OrderItem

- id (Primary Key)
- orderId (Foreign Key)
- itemName
- description
- quantity
- unitPrice
- createdAt

### Shipment

- id (Primary Key)
- shipmentNumber (Unique)
- trackingNumber (Unique)
- orderId (Foreign Key)
- status
- createdAt
- updatedAt

### ShipmentItem

- id (Primary Key)
- shipmentId (Foreign Key)
- orderItemId (Foreign Key)
- quantity

### Driver

- id (Primary Key)
- userId (Foreign Key, Unique)
- licenseNumber (Unique)
- availabilityStatus
- createdAt
- updatedAt

### Vehicle

- id (Primary Key)
- registrationNumber (Unique)
- vehicleType
- capacity
- status
- createdAt
- updatedAt

### Delivery

- id (Primary Key)
- shipmentId (Foreign Key, NOT Unique)
- deliveryNumber (Unique)
- status
- scheduledStart
- scheduledEnd
- actualStart
- completedAt
- failureReason
- createdAt
- updatedAt

Each Delivery record represents a separate attempt to deliver the same shipment.

### DeliveryAssignment

- id (Primary Key)
- deliveryId (Foreign Key)
- driverId (Foreign Key)
- vehicleId (Foreign Key)
- assignedAt
- scheduledStart
- scheduledEnd
- releasedAt
- status
- createdAt
- updatedAt

Assignments must be retained as history. Reassigning a delivery should not erase previous assignments.

### ProofOfDelivery

- id (Primary Key)
- deliveryId (Foreign Key, Unique)
- recipientName
- recipientConfirmed
- photoUrl (Optional)
- signatureUrl (Optional)
- deliveryNotes
- submittedAt
- createdAt

A ProofOfDelivery record belongs to one successful delivery attempt.

### TrackingEvent

- id (Primary Key)
- shipmentId (Foreign Key)
- status
- description
- location
- createdAt

Each tracking event records a shipment status update.

### Notification

- id (Primary Key)
- userId (Foreign Key)
- title
- message
- isRead
- createdAt

## 6. Status Management

The system uses separate status fields for Orders, Shipments, and Deliveries.

### Order Status

- PENDING
- CONFIRMED
- PROCESSING
- READY_FOR_DISPATCH
- COMPLETED
- CANCELLED

An Order is marked COMPLETED only after all its shipments have been successfully delivered.

### Shipment Status

- CREATED
- PICKED_UP
- IN_TRANSIT
- ARRIVED_AT_DESTINATION
- OUT_FOR_DELIVERY
- DELIVERED
- DELIVERY_FAILED
- RETURNED
- CANCELLED

A shipment can have several delivery attempts. A failed attempt does not automatically mean the shipment is permanently failed.

When another attempt is scheduled, the shipment can move back to OUT_FOR_DELIVERY when that attempt begins.

### Delivery Status

- SCHEDULED
- ASSIGNED
- IN_PROGRESS
- DELIVERED
- FAILED
- RESCHEDULED
- CANCELLED

Each delivery attempt has its own lifecycle.

A failed or rescheduled attempt remains in the database for historical reference.

## 7. Assignment Rules

- A driver must be active and eligible before assignment.
- A vehicle must be active and suitable for the delivery.
- A driver cannot have overlapping active assignments.
- A vehicle cannot have overlapping active assignments.
- Completed and cancelled assignments do not block future schedules.
- Assignment validation must be performed on the backend.
- Assignment creation must use a database transaction and appropriate concurrency protection to prevent conflicting assignments.

The overlap condition is:

`newStart < existingEnd AND newEnd > existingStart`

## 8. Important Database Constraints

The database should enforce:

- Unique user emails.
- Unique order numbers.
- Unique shipment numbers.
- Unique tracking numbers.
- Unique driver license numbers.
- Unique vehicle registration numbers.
- Unique delivery numbers.
- One Customer profile per User.
- One Driver profile per User.
- One ProofOfDelivery per Delivery.
- Valid foreign key relationships.
- Positive item quantities.
- Nonnegative prices and vehicle capacity.
- Valid scheduled delivery time ranges.

Business rules that depend on several records, such as assignment conflicts and status transitions, must also be enforced in the backend.

## 9. Order and Shipment Consistency

An order may contain multiple shipments.

The system must not mark an order as COMPLETED when only one of its shipments has been delivered.

The order can be marked COMPLETED when all associated shipments have status DELIVERED.

If an order has no shipments, it cannot be marked COMPLETED.

## 10. Data Integrity and History

- Use foreign keys to maintain relationships.
- Use database transactions for operations involving multiple related records.
- Preserve delivery attempts and assignment history.
- Record shipment status changes as TrackingEvents.
- Avoid permanently deleting important operational records.
- Use timestamps to maintain an accurate activity history.

## 11. Design Scope

This database design supports the initial MVP.

Advanced features such as GPS location history, automatic route optimization, payment processing, and third-party logistics integrations are outside the initial scope.
