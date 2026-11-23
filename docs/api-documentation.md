# Nexora Logistics - API Documentation

## 1. Overview

The Nexora Logistics backend will expose a RESTful API built with Node.js, Express, TypeScript, and PostgreSQL.

The API will allow the frontend application to communicate with the backend for authentication, customer management, order processing, shipment tracking, delivery scheduling, fleet management, and reporting.

## 2. API Base URL

During local development:

```text
http://localhost:5000/api/v1
```

All endpoints will use the `/api/v1` prefix.

## 3. Authentication

Authentication will use JSON Web Tokens (JWT).

Users must authenticate before accessing protected endpoints.

### Authentication Endpoints

| Method | Endpoint       | Description                                                | Access        |
| ------ | -------------- | ---------------------------------------------------------- | ------------- |
| POST   | /auth/register | Register a customer account                                | Public        |
| POST   | /auth/login    | Authenticate a user                                        | Public        |
| GET    | /auth/me       | Get current user information                               | Authenticated |
| POST   | /auth/refresh  | Refresh an access token, if refresh tokens are enabled     | Authenticated |
| POST   | /auth/logout   | Log out and invalidate the refresh session, if implemented | Authenticated |

Customer registration will be publicly available.

Staff accounts will be created by an administrator or through an authorized administrative process.

## 4. User Management

| Method | Endpoint          | Description                   | Access |
| ------ | ----------------- | ----------------------------- | ------ |
| GET    | /users            | List users                    | ADMIN  |
| GET    | /users/:id        | Get user details              | ADMIN  |
| POST   | /users            | Create a staff account        | ADMIN  |
| PATCH  | /users/:id        | Update user information       | ADMIN  |
| PATCH  | /users/:id/status | Activate or deactivate a user | ADMIN  |

## 5. Customer Management

| Method | Endpoint       | Description                     | Access                   |
| ------ | -------------- | ------------------------------- | ------------------------ |
| GET    | /customers/me  | Get current customer profile    | CUSTOMER                 |
| PATCH  | /customers/me  | Update current customer profile | CUSTOMER                 |
| GET    | /customers     | List customers                  | ADMIN, LOGISTICS_MANAGER |
| GET    | /customers/:id | Get customer details            | ADMIN, LOGISTICS_MANAGER |

Customers can only access and update their own profiles.

## 6. Address Management

| Method | Endpoint       | Description                 | Access                    |
| ------ | -------------- | --------------------------- | ------------------------- |
| POST   | /addresses     | Create an address           | Authenticated             |
| GET    | /addresses     | List accessible addresses   | Authenticated             |
| GET    | /addresses/:id | Get address details         | Authorized owner or staff |
| PATCH  | /addresses/:id | Update an address           | Authorized owner or staff |
| DELETE | /addresses/:id | Delete an address if unused | Authorized owner or staff |

Customers can manage their own addresses.

## 7. Order Management

| Method | Endpoint           | Description                     | Access                    |
| ------ | ------------------ | ------------------------------- | ------------------------- |
| POST   | /orders            | Create an order                 | CUSTOMER                  |
| GET    | /orders            | List accessible orders          | Role-based                |
| GET    | /orders/:id        | Get order details               | Owner or authorized staff |
| PATCH  | /orders/:id        | Update an eligible order        | Owner or authorized staff |
| PATCH  | /orders/:id/status | Update order status             | ADMIN, LOGISTICS_MANAGER  |
| POST   | /orders/:id/cancel | Request or perform cancellation | Owner or authorized staff |

Business rules:

- Customers can create orders through their accounts.
- Customers can only view their own orders.
- Customers cannot directly mark orders as completed.
- Order status transitions must follow the status management rules.
- An order must contain at least one item.

## 8. Shipment Management

| Method | Endpoint              | Description                 | Access                              |
| ------ | --------------------- | --------------------------- | ----------------------------------- |
| POST   | /shipments            | Create a shipment           | ADMIN, LOGISTICS_MANAGER            |
| GET    | /shipments            | List accessible shipments   | Role-based                          |
| GET    | /shipments/:id        | Get shipment details        | Owner or authorized staff           |
| PATCH  | /shipments/:id        | Update shipment information | ADMIN, LOGISTICS_MANAGER            |
| PATCH  | /shipments/:id/status | Update shipment status      | Authorized staff or assigned driver |

Business rules:

- Every shipment must belong to an existing order.
- Every shipment must have a unique tracking number.
- Shipment status changes must follow permitted transitions.
- Shipment status changes must create tracking history records.

## 9. Delivery Management

| Method | Endpoint                   | Description                | Access                               |
| ------ | -------------------------- | -------------------------- | ------------------------------------ |
| POST   | /deliveries                | Create a delivery          | ADMIN, LOGISTICS_MANAGER, DISPATCHER |
| GET    | /deliveries                | List accessible deliveries | Role-based                           |
| GET    | /deliveries/:id            | Get delivery details       | Authorized user                      |
| PATCH  | /deliveries/:id/status     | Update delivery status     | DISPATCHER or assigned DRIVER        |
| POST   | /deliveries/:id/reschedule | Reschedule a delivery      | DISPATCHER                           |
| POST   | /deliveries/:id/cancel     | Cancel a delivery          | Authorized staff                     |

Drivers can only update deliveries assigned to them.

## 10. Driver Management

| Method | Endpoint               | Description              | Access                               |
| ------ | ---------------------- | ------------------------ | ------------------------------------ |
| POST   | /drivers               | Create driver profile    | ADMIN, LOGISTICS_MANAGER             |
| GET    | /drivers               | List drivers             | ADMIN, LOGISTICS_MANAGER, DISPATCHER |
| GET    | /drivers/:id           | Get driver details       | Authorized staff or driver           |
| PATCH  | /drivers/:id           | Update driver details    | ADMIN, LOGISTICS_MANAGER             |
| GET    | /drivers/me/deliveries | View assigned deliveries | DRIVER                               |

## 11. Vehicle Management

| Method | Endpoint             | Description                | Access                               |
| ------ | -------------------- | -------------------------- | ------------------------------------ |
| POST   | /vehicles            | Register vehicle           | ADMIN, LOGISTICS_MANAGER             |
| GET    | /vehicles            | List vehicles              | ADMIN, LOGISTICS_MANAGER, DISPATCHER |
| GET    | /vehicles/:id        | Get vehicle details        | Authorized staff                     |
| PATCH  | /vehicles/:id        | Update vehicle information | ADMIN, LOGISTICS_MANAGER             |
| PATCH  | /vehicles/:id/status | Update vehicle status      | ADMIN, LOGISTICS_MANAGER             |

## 12. Driver and Vehicle Assignment

| Method | Endpoint                | Description               | Access                               |
| ------ | ----------------------- | ------------------------- | ------------------------------------ |
| POST   | /assignments            | Assign driver and vehicle | DISPATCHER                           |
| GET    | /assignments            | List assignments          | ADMIN, LOGISTICS_MANAGER, DISPATCHER |
| GET    | /assignments/:id        | Get assignment details    | Authorized staff                     |
| PATCH  | /assignments/:id        | Update an assignment      | DISPATCHER                           |
| POST   | /assignments/:id/cancel | Cancel an assignment      | DISPATCHER                           |

Business rules:

- Drivers must be active and eligible.
- Vehicles must be available and suitable.
- Driver scheduling conflicts must be rejected.
- Vehicle scheduling conflicts must be rejected.
- Concurrent conflicting assignments must not both succeed.
- Assignment history must be preserved.

## 13. Proof of Delivery

| Method | Endpoint              | Description                | Access          |
| ------ | --------------------- | -------------------------- | --------------- |
| POST   | /deliveries/:id/proof | Submit proof of delivery   | Assigned DRIVER |
| GET    | /deliveries/:id/proof | Retrieve proof of delivery | Authorized user |

The proof submission endpoint will support multipart form data when uploading photos or signatures.

Required information:

- Recipient name.
- Recipient confirmation.
- Delivery ID.

Optional evidence:

- Delivery photo.
- Recipient signature.
- Delivery notes.

A delivery cannot become DELIVERED without the required proof.

## 14. Customer Tracking

### Public Tracking

| Method | Endpoint                  | Description      | Access |
| ------ | ------------------------- | ---------------- | ------ |
| GET    | /tracking/:trackingNumber | Track a shipment | Public |

The public tracking endpoint will return only customer-safe information.

### Authenticated Customer Tracking

| Method | Endpoint                    | Description                   | Access   |
| ------ | --------------------------- | ----------------------------- | -------- |
| GET    | /customers/me/shipments     | List the customer's shipments | CUSTOMER |
| GET    | /customers/me/shipments/:id | View an owned shipment        | CUSTOMER |

The backend must verify shipment ownership before returning authenticated customer shipment information.

## 15. Notifications

| Method | Endpoint                | Description                    | Access        |
| ------ | ----------------------- | ------------------------------ | ------------- |
| GET    | /notifications          | List notifications             | Authenticated |
| PATCH  | /notifications/:id/read | Mark notification as read      | Owner         |
| PATCH  | /notifications/read-all | Mark all notifications as read | Authenticated |

## 16. Reports and Analytics

| Method | Endpoint            | Description                | Access                   |
| ------ | ------------------- | -------------------------- | ------------------------ |
| GET    | /reports/dashboard  | Get dashboard statistics   | ADMIN, LOGISTICS_MANAGER |
| GET    | /reports/orders     | Order reports              | ADMIN, LOGISTICS_MANAGER |
| GET    | /reports/shipments  | Shipment reports           | ADMIN, LOGISTICS_MANAGER |
| GET    | /reports/deliveries | Delivery reports           | ADMIN, LOGISTICS_MANAGER |
| GET    | /reports/drivers    | Driver performance reports | ADMIN, LOGISTICS_MANAGER |

## 17. Standard HTTP Status Codes

| Code | Meaning                                    |
| ---- | ------------------------------------------ |
| 200  | Request successful                         |
| 201  | Resource created successfully              |
| 204  | Request successful with no response body   |
| 400  | Invalid request or business rule violation |
| 401  | Authentication required                    |
| 403  | Permission denied                          |
| 404  | Resource not found                         |
| 409  | Resource conflict                          |
| 422  | Request validation failed                  |
| 429  | Too many requests                          |
| 500  | Internal server error                      |

## 18. Standard Error Response

Example:

```json
{
  "success": false,
  "message": "Driver is unavailable during the selected time.",
  "errors": []
}
```

## 19. Standard Success Response

Example:

```json
{
  "success": true,
  "message": "Order created successfully.",
  "data": {
    "id": "example-order-id",
    "status": "PENDING"
  }
}
```

## 20. Security Requirements

- Protected endpoints must require authentication.
- Role-based authorization must be enforced by the backend.
- Customers must only access their own resources.
- Passwords must never be returned in API responses.
- Request data must be validated.
- Public tracking endpoints must be rate-limited.
- Sensitive configuration must be stored in environment variables.
- API errors must not expose internal implementation details.

## 21. API Versioning

All API endpoints will use the `/api/v1` prefix.

Future breaking changes may be introduced through a new API version.
