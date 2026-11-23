## 3. `assignment-rules.md`

This document addresses the requirement to prevent driver and vehicle scheduling conflicts.

````markdown
# Nexora Logistics - Driver and Vehicle Assignment Rules

## 1. Purpose

This document defines the rules for assigning drivers and vehicles to deliveries.

The system must prevent scheduling conflicts and ensure that drivers and vehicles are available during the requested delivery period.

## 2. Assignment Information

A delivery assignment should contain:

- Assignment ID.
- Delivery ID.
- Driver ID.
- Vehicle ID.
- Scheduled start time.
- Scheduled end time.
- Assignment status.
- Assigned by user ID.
- Creation date.
- Last updated date.

## 3. Driver Assignment Rules

Before assigning a driver, the system must verify:

1. The driver exists.
2. The driver account is active.
3. The driver is eligible for delivery assignments.
4. The driver has no overlapping active assignment.
5. The driver has the required license or qualifications for the assigned vehicle, where applicable.

A driver cannot be assigned to two deliveries that overlap in time.

## 4. Vehicle Assignment Rules

Before assigning a vehicle, the system must verify:

1. The vehicle exists.
2. The vehicle is active.
3. The vehicle is available for operation.
4. The vehicle has no overlapping active assignment.
5. The vehicle has sufficient capacity for the shipment.
6. The vehicle is suitable for the delivery requirements.

A vehicle cannot be assigned to two deliveries that overlap in time.

## 5. Scheduling Conflict Detection

Two time intervals overlap when:

```text
newStart < existingEnd
AND
newEnd > existingStart
```
````
