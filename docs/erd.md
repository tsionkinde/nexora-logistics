# Nexora Logistics - Entity Relationship Diagram

## 1. Overview

This diagram represents the main database entities and their relationships.

The system uses PostgreSQL with Prisma ORM.

## 2. Entity Relationship Diagram

```mermaid
erDiagram
    USER ||--o| CUSTOMER : has
    USER ||--o| DRIVER : has
    USER ||--o{ NOTIFICATION : receives
    USER ||--o{ DELIVERY_ASSIGNMENT : creates
    USER ||--o{ TRACKING_EVENT : records
    USER ||--o{ PROOF_OF_DELIVERY : submits

    CUSTOMER ||--o{ ORDER : places

    ADDRESS ||--o{ ORDER : pickup_address
    ADDRESS ||--o{ ORDER : delivery_address
    ADDRESS ||--o{ SHIPMENT : origin
    ADDRESS ||--o{ SHIPMENT : destination

    ORDER ||--|{ ORDER_ITEM : contains
    ORDER ||--o{ SHIPMENT : produces

    SHIPMENT ||--|{ SHIPMENT_ITEM : contains
    ORDER_ITEM ||--o{ SHIPMENT_ITEM : allocated_to

    SHIPMENT ||--o| DELIVERY : has
    SHIPMENT ||--o{ TRACKING_EVENT : records

    DELIVERY ||--o{ DELIVERY_ASSIGNMENT : has
    DELIVERY ||--o| PROOF_OF_DELIVERY : confirmed_by

    DRIVER ||--o{ DELIVERY_ASSIGNMENT : assigned_to
    VEHICLE ||--o{ DELIVERY_ASSIGNMENT : uses

    USER {
        UUID id PK
        string fullName
        string email UK
        string passwordHash
        string role
        boolean isActive
    }

    CUSTOMER {
        UUID id PK
        UUID userId FK
        string companyName
        string phone
    }

    DRIVER {
        UUID id PK
        UUID userId FK
        string licenseNumber UK
        date licenseExpiryDate
        boolean isAvailable
    }

    ADDRESS {
        UUID id PK
        string contactName
        string phone
        string street
        string city
        string region
        string country
    }

    ORDER {
        UUID id PK
        string orderNumber UK
        UUID customerId FK
        UUID pickupAddressId FK
        UUID deliveryAddressId FK
        string status
        decimal totalAmount
    }

    ORDER_ITEM {
        UUID id PK
        UUID orderId FK
        string description
        int quantity
        decimal unitWeight
        decimal declaredValue
    }

    SHIPMENT {
        UUID id PK
        string trackingNumber UK
        UUID orderId FK
        UUID originAddressId FK
        UUID destinationAddressId FK
        string status
        datetime estimatedDeliveryDate
    }

    SHIPMENT_ITEM {
        UUID id PK
        UUID shipmentId FK
        UUID orderItemId FK
        int quantity
    }

    DELIVERY {
        UUID id PK
        UUID shipmentId FK
        string status
        datetime scheduledStart
        datetime scheduledEnd
        datetime completedAt
    }

    DELIVERY_ASSIGNMENT {
        UUID id PK
        UUID deliveryId FK
        UUID driverId FK
        UUID vehicleId FK
        UUID assignedById FK
        datetime scheduledStart
        datetime scheduledEnd
        string status
    }

    VEHICLE {
        UUID id PK
        string registrationNumber UK
        string vehicleType
        decimal capacityKg
        string status
    }

    PROOF_OF_DELIVERY {
        UUID id PK
        UUID deliveryId FK
        string recipientName
        boolean recipientConfirmed
        string photoUrl
        string signatureUrl
        UUID submittedById FK
        datetime submittedAt
    }

    TRACKING_EVENT {
        UUID id PK
        UUID shipmentId FK
        string previousStatus
        string newStatus
        string location
        UUID createdById FK
        datetime createdAt
    }

    NOTIFICATION {
        UUID id PK
        UUID userId FK
        string title
        string message
        boolean isRead
        datetime createdAt
    }
```

## 3. Relationship Explanation

- A user may have a customer profile or driver profile.
- A customer can place multiple orders.
- Each order contains one or more order items.
- One order may produce multiple shipments.
- Shipment items connect order items to shipments.
- Each shipment can have one delivery record.
- Each delivery can have multiple assignment records over time.
- A driver and vehicle can participate in multiple assignments over time.
- A completed delivery can have one proof of delivery record.
- A shipment can have many tracking events.
- A user can receive multiple notifications.

## 4. Important Design Decisions

1. UUIDs are used as primary keys.
2. Orders and shipments are separate entities because an order may be split into multiple shipments.
3. ShipmentItem supports partial shipment of order items.
4. DeliveryAssignment preserves driver and vehicle assignment history.
5. TrackingEvent preserves shipment status history.
6. ProofOfDelivery is stored separately to maintain delivery evidence.
7. Public tracking uses a unique tracking number rather than exposing internal database IDs.
