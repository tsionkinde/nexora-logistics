## 4. `proof-of-delivery.md`

```markdown
# Nexora Logistics - Proof of Delivery

## 1. Purpose

Proof of Delivery (POD) is the information recorded to confirm that a shipment has been successfully delivered to its intended recipient.

The system must require valid proof before a driver can mark a delivery as DELIVERED.

## 2. Required Information

The proof of delivery record should contain the following information:

| Field                  | Description                                             | Required |
| ---------------------- | ------------------------------------------------------- | -------- |
| Delivery ID            | Identifies the completed delivery.                      | Yes      |
| Recipient Name         | Name of the person receiving the shipment.              | Yes      |
| Delivery Date and Time | Date and time the delivery was completed.               | Yes      |
| Recipient Confirmation | Confirmation that the recipient received the shipment.  | Yes      |
| Delivery Notes         | Additional information about the delivery.              | No       |
| Proof Image            | Photo showing the delivered goods or delivery location. | Optional |
| Recipient Signature    | Digital signature of the recipient.                     | Optional |
| Submitted By           | Driver who submitted the proof.                         | Yes      |

At least one additional evidence item, such as a photo or signature, may be required according to the final delivery policy.

For the initial MVP, recipient name and confirmation will be mandatory. A photo or signature will be supported as additional evidence.

## 3. Proof of Delivery Workflow

1. The driver opens an assigned delivery.
2. The driver starts the delivery.
3. The driver reaches the delivery destination.
4. The driver enters the recipient's name.
5. The driver confirms that the recipient received the shipment.
6. The driver adds optional delivery notes.
7. The driver uploads a proof image or records a recipient signature when available.
8. The backend validates the submitted information.
9. The proof of delivery record is saved.
10. The delivery status changes to DELIVERED.
11. The associated shipment status changes to DELIVERED.
12. The associated order status changes to COMPLETED when all required shipments have been successfully delivered.

## 4. Validation Rules

- Only the driver assigned to the delivery or an authorized user can submit proof.
- The delivery must be in the IN_PROGRESS state.
- Recipient name must not be empty.
- Recipient confirmation must be provided.
- Uploaded files must use supported formats.
- Uploaded file size must be restricted.
- The delivery must not already be completed.
- Proof of delivery must be saved before the delivery status changes to DELIVERED.

## 5. Security and Storage

- Proof images must be stored securely.
- Uploaded files must be validated on the backend.
- Customers must only access proof associated with their own shipments.
- Drivers must not modify proof submitted by another driver unless explicitly authorized.
- Proof records must include the submitting user's ID and submission timestamp.

## 6. Failed Delivery

If a delivery cannot be completed:

1. The driver selects FAILED.
2. The driver provides a failure reason.
3. The system records the failed delivery attempt.
4. The dispatcher reviews the failure.
5. The dispatcher may reschedule or cancel the delivery according to business rules.

Possible failure reasons include:

- Customer unavailable.
- Incorrect delivery address.
- Customer refused the shipment.
- Access to delivery location unavailable.
- Other reason.

## 7. Acceptance Criteria

- A delivery cannot be completed without required proof information.
- Only authorized users can submit proof.
- Proof must be associated with the correct delivery.
- Successful proof submission must update the delivery status.
- Failed delivery attempts must include a reason.
- Proof submission must be recorded with a timestamp and user ID.
```
