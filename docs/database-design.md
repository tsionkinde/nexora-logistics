# Nexora Logistics - Database Design

## 1. Overview

The Nexora Logistics database will use PostgreSQL as its relational database management system.

Prisma ORM will be used to define database models, manage migrations, and interact with the database from the backend.

The database is designed to support order management, shipment processing, delivery scheduling, driver and vehicle assignment, customer tracking, and proof of delivery.

## 2. Database Design Principles

The database will follow these principles:

- Use primary keys to uniquely identify records.
- Use foreign keys to maintain relationships.
- Apply appropriate unique constraints.
- Use timestamps to track record creation and updates.
- Validate important business rules in the backend.
- Use database transactions for critical operations.
- Avoid unnecessary duplication of data.
- Preserve shipment and delivery history.

## 3. Main Database Tables

### 3.1 User

Stores system user account information.

| Field        | Type      | Description                                            |
| ------------ | --------- | ------------------------------------------------------ |
| id           | UUID      | Primary key                                            |
| fullName     | VARCHAR   | User's full name                                       |
| email        | VARCHAR   | Unique email address                                   |
| passwordHash | VARCHAR   | Hashed password                                        |
| role         | ENUM      | ADMIN, LOGISTICS_MANAGER, DISPATCHER, DRIVER, CUSTOMER |
| isActive     | BOOLEAN   | Account status                                         |
| createdAt    | TIMESTAMP | Account creation time                                  |
| updatedAt    | TIMESTAMP | Last update time                                       |

### 3.2 Customer

Stores additional information about customers.

| Field       | Type      | Description                            |
| ----------- | --------- | -------------------------------------- |
| id          | UUID      | Primary key                            |
| userId      | UUID      | Unique foreign key referencing User    |
| companyName | VARCHAR   | Customer's company name, if applicable |
| phone       | VARCHAR   | Customer contact number                |
| createdAt   | TIMESTAMP | Creation time                          |
| updatedAt   | TIMESTAMP | Last update time                       |

A customer profile is associated with one User account.

### 3.3 Address

Stores pickup and delivery addresses.

| Field       | Type      | Description               |
| ----------- | --------- | ------------------------- |
| id          | UUID      | Primary key               |
| contactName | VARCHAR   | Contact person's name     |
| phone       | VARCHAR   | Contact number            |
| street      | VARCHAR   | Street or locality        |
| city        | VARCHAR   | City                      |
| region      | VARCHAR   | Region or state           |
| country     | VARCHAR   | Country                   |
| postalCode  | VARCHAR   | Postal code, if available |
| createdAt   | TIMESTAMP | Creation time             |

### 3.4 Order

Stores customer orders.

| Field             | Type      | Description                      |
| ----------------- | --------- | -------------------------------- |
| id                | UUID      | Primary key                      |
| orderNumber       | VARCHAR   | Unique order reference           |
| customerId        | UUID      | Foreign key referencing Customer |
| pickupAddressId   | UUID      | Foreign key referencing Address  |
| deliveryAddressId | UUID      | Foreign key referencing Address  |
| status            | ENUM      | Order status                     |
| totalAmount       | DECIMAL   | Total declared order value       |
| notes             | TEXT      | Additional order notes           |
| createdAt         | TIMESTAMP | Creation time                    |
| updatedAt         | TIMESTAMP | Last update time                 |

### 3.5 OrderItem

Stores individual items belonging to an order.

| Field         | Type      | Description                   |
| ------------- | --------- | ----------------------------- |
| id            | UUID      | Primary key                   |
| orderId       | UUID      | Foreign key referencing Order |
| description   | VARCHAR   | Item description              |
| quantity      | INTEGER   | Number of units               |
| unitWeight    | DECIMAL   | Weight per unit               |
| declaredValue | DECIMAL   | Declared value per unit       |
| createdAt     | TIMESTAMP | Creation time                 |

### 3.6 Shipment

Stores shipment information.

| Field                 | Type      | Description                     |
| --------------------- | --------- | ------------------------------- |
| id                    | UUID      | Primary key                     |
| trackingNumber        | VARCHAR   | Unique tracking number          |
| orderId               | UUID      | Foreign key referencing Order   |
| originAddressId       | UUID      | Foreign key referencing Address |
| destinationAddressId  | UUID      | Foreign key referencing Address |
| status                | ENUM      | Shipment status                 |
| estimatedDeliveryDate | TIMESTAMP | Estimated delivery time         |
| createdAt             | TIMESTAMP | Creation time                   |
| updatedAt             | TIMESTAMP | Last update time                |

One order may have multiple shipments.

### 3.7 ShipmentItem

Connects shipment records to the items included in each shipment.

| Field       | Type    | Description                        |
| ----------- | ------- | ---------------------------------- |
| id          | UUID    | Primary key                        |
| shipmentId  | UUID    | Foreign key referencing Shipment   |
| orderItemId | UUID    | Foreign key referencing OrderItem  |
| quantity    | INTEGER | Quantity included in this shipment |

This allows an order to be divided into multiple shipments.

### 3.8 Driver

Stores driver-specific information.

| Field             | Type      | Description                         |
| ----------------- | --------- | ----------------------------------- |
| id                | UUID      | Primary key                         |
| userId            | UUID      | Unique foreign key referencing User |
| licenseNumber     | VARCHAR   | Unique driving license number       |
| licenseExpiryDate | DATE      | License expiration date             |
| phone             | VARCHAR   | Driver contact number               |
| isAvailable       | BOOLEAN   | Availability indicator              |
| createdAt         | TIMESTAMP | Creation time                       |
| updatedAt         | TIMESTAMP | Last update time                    |

### 3.9 Vehicle

Stores fleet and vehicle information.

| Field              | Type      | Description                              |
| ------------------ | --------- | ---------------------------------------- |
| id                 | UUID      | Primary key                              |
| registrationNumber | VARCHAR   | Unique vehicle registration              |
| vehicleType        | VARCHAR   | Vehicle category                         |
| capacityKg         | DECIMAL   | Maximum carrying capacity                |
| status             | ENUM      | AVAILABLE, IN_USE, MAINTENANCE, INACTIVE |
| createdAt          | TIMESTAMP | Creation time                            |
| updatedAt          | TIMESTAMP | Last update time                         |

### 3.10 Delivery

Stores delivery information for a shipment.

| Field          | Type      | Description                             |
| -------------- | --------- | --------------------------------------- |
| id             | UUID      | Primary key                             |
| shipmentId     | UUID      | Unique foreign key referencing Shipment |
| status         | ENUM      | Delivery status                         |
| scheduledStart | TIMESTAMP | Planned delivery start                  |
| scheduledEnd   | TIMESTAMP | Planned delivery end                    |
| actualStart    | TIMESTAMP | Actual delivery start                   |
| completedAt    | TIMESTAMP | Delivery completion time                |
| failureReason  | TEXT      | Reason for unsuccessful delivery        |
| createdAt      | TIMESTAMP | Creation time                           |
| updatedAt      | TIMESTAMP | Last update time                        |

### 3.11 DeliveryAssignment

Stores driver and vehicle assignment history.

| Field          | Type      | Description                      |
| -------------- | --------- | -------------------------------- |
| id             | UUID      | Primary key                      |
| deliveryId     | UUID      | Foreign key referencing Delivery |
| driverId       | UUID      | Foreign key referencing Driver   |
| vehicleId      | UUID      | Foreign key referencing Vehicle  |
| scheduledStart | TIMESTAMP | Assignment start                 |
| scheduledEnd   | TIMESTAMP | Assignment end                   |
| status         | ENUM      | ACTIVE, COMPLETED, CANCELLED     |
| assignedById   | UUID      | Foreign key referencing User     |
| createdAt      | TIMESTAMP | Assignment creation time         |

Assignment records are preserved to maintain assignment history.

### 3.12 ProofOfDelivery

Stores delivery confirmation information.

| Field              | Type      | Description                             |
| ------------------ | --------- | --------------------------------------- |
| id                 | UUID      | Primary key                             |
| deliveryId         | UUID      | Unique foreign key referencing Delivery |
| recipientName      | VARCHAR   | Person receiving the shipment           |
| recipientConfirmed | BOOLEAN   | Confirmation of receipt                 |
| photoUrl           | VARCHAR   | Optional proof image location           |
| signatureUrl       | VARCHAR   | Optional signature image location       |
| notes              | TEXT      | Additional delivery notes               |
| submittedById      | UUID      | Foreign key referencing User            |
| submittedAt        | TIMESTAMP | Proof submission time                   |

### 3.13 TrackingEvent

Stores shipment status history.

| Field          | Type      | Description                      |
| -------------- | --------- | -------------------------------- |
| id             | UUID      | Primary key                      |
| shipmentId     | UUID      | Foreign key referencing Shipment |
| previousStatus | VARCHAR   | Previous shipment status         |
| newStatus      | VARCHAR   | New shipment status              |
| location       | VARCHAR   | Event location, if available     |
| description    | TEXT      | Event description                |
| createdById    | UUID      | Foreign key referencing User     |
| createdAt      | TIMESTAMP | Event timestamp                  |

### 3.14 Notification

Stores notifications sent to users.

| Field     | Type      | Description                  |
| --------- | --------- | ---------------------------- |
| id        | UUID      | Primary key                  |
| userId    | UUID      | Foreign key referencing User |
| title     | VARCHAR   | Notification title           |
| message   | TEXT      | Notification content         |
| isRead    | BOOLEAN   | Read status                  |
| createdAt | TIMESTAMP | Creation time                |

## 4. Entity Relationships

| Relationship                     | Cardinality        |
| -------------------------------- | ------------------ |
| User - Customer                  | One-to-zero-or-one |
| User - Driver                    | One-to-zero-or-one |
| Customer - Order                 | One-to-many        |
| Address - Order (pickup)         | One-to-many        |
| Address - Order (delivery)       | One-to-many        |
| Order - OrderItem                | One-to-many        |
| Order - Shipment                 | One-to-many        |
| Shipment - ShipmentItem          | One-to-many        |
| OrderItem - ShipmentItem         | One-to-many        |
| Address - Shipment (origin)      | One-to-many        |
| Address - Shipment (destination) | One-to-many        |
| Shipment - Delivery              | One-to-one         |
| Delivery - DeliveryAssignment    | One-to-many        |
| Driver - DeliveryAssignment      | One-to-many        |
| Vehicle - DeliveryAssignment     | One-to-many        |
| Delivery - ProofOfDelivery       | One-to-zero-or-one |
| Shipment - TrackingEvent         | One-to-many        |
| User - Notification              | One-to-many        |

## 5. Important Database Constraints

- User email addresses must be unique.
- Customer user IDs must be unique.
- Driver user IDs must be unique.
- Driver license numbers must be unique.
- Vehicle registration numbers must be unique.
- Order numbers must be unique.
- Shipment tracking numbers must be unique.
- A delivery must belong to one shipment.
- A shipment must belong to one order.
- A shipment item must reference a valid order item.
- Proof of delivery must be associated with one delivery.
- Assignment scheduling must prevent overlapping active assignments.
- Quantities and weights must be positive.
- Delivery end time must be later than delivery start time.

## 6. Data Integrity

Critical operations must use database transactions.

Examples include:

- Creating an order and its order items.
- Creating a shipment and its shipment items.
- Assigning a driver and vehicle.
- Completing a delivery and recording proof.
- Updating shipment status and creating tracking history.

The backend will enforce business rules that cannot be guaranteed by ordinary database constraints alone.

## 7. Conclusion

This database design provides a structured foundation for the Nexora Logistics MVP.

It supports the core logistics workflows while allowing future improvements such as GPS tracking, route optimization, and external logistics integrations.
