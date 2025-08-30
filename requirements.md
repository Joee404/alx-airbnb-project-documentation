# Airbnb Clone Backend - Requirements Specifications

This document defines the technical and functional requirements for key backend features of the Airbnb Clone project. It includes API endpoints, input/output specifications, validation rules, and performance criteria.

---

## 1. User Authentication

**Description:**  
Handles registration, login, and secure session management for guests, hosts, and admins.

**API Endpoints:**

- **POST /api/auth/register**
  - Input: `{ "name": "string", "email": "string", "password": "string", "role": "guest|host" }`
  - Output: `{ "userId": "uuid", "token": "jwt" }`
  - Validation:
    - Email must be valid format.
    - Password must be at least 8 characters.
    - Role must be either `guest` or `host`.
- **POST /api/auth/login**
  - Input: `{ "email": "string", "password": "string" }`
  - Output: `{ "userId": "uuid", "token": "jwt" }`
  - Validation:
    - Email and password must match existing records.
- **GET /api/auth/me**
  - Input: JWT token in Authorization header.
  - Output: `{ "userId": "uuid", "name": "string", "email": "string", "role": "guest|host" }`
  - Validation: Token must be valid and not expired.

**Performance Criteria:**

- Registration and login requests must respond within 300ms.
- JWT tokens expire after 1 hour; refresh token mechanism optional.

---

## 2. Property Management

**Description:**  
Enables hosts to create, update, delete, and retrieve property listings.

**API Endpoints:**

- **POST /api/properties**
  - Input: `{ "title": "string", "description": "string", "location": "string", "price": "decimal", "amenities": ["string"], "availability": ["date"] }`
  - Output: `{ "propertyId": "uuid", "status": "created" }`
  - Validation:
    - Price must be positive number.
    - Title and location are required.
- **GET /api/properties**
  - Input: Optional filters: `location`, `priceRange`, `guests`, `amenities`
  - Output: `[ { "propertyId": "uuid", "title": "string", "price": "decimal", "location": "string", "availability": ["date"] } ]`
- **PUT /api/properties/:id**
  - Input: Same as POST; all fields optional
  - Output: `{ "propertyId": "uuid", "status": "updated" }`
  - Validation: Only the host who owns the property can update it.
- **DELETE /api/properties/:id**
  - Output: `{ "propertyId": "uuid", "status": "deleted" }`
  - Validation: Only property owner can delete.

**Performance Criteria:**

- Property search should respond within 500ms for up to 10,000 records.
- CRUD operations must maintain data consistency in the database.

---

## 3. Booking System

**Description:**  
Manages guest bookings, ensures availability, and tracks booking status.

**API Endpoints:**

- **POST /api/bookings**
  - Input: `{ "propertyId": "uuid", "guestId": "uuid", "startDate": "YYYY-MM-DD", "endDate": "YYYY-MM-DD" }`
  - Output: `{ "bookingId": "uuid", "status": "pending" }`
  - Validation:
    - `startDate` must be before `endDate`.
    - Property must be available for selected dates.
- **GET /api/bookings/:userId**
  - Input: `userId` (guest or host)
  - Output: `[ { "bookingId": "uuid", "propertyId": "uuid", "startDate": "YYYY-MM-DD", "endDate": "YYYY-MM-DD", "status": "pending|confirmed|canceled|completed" } ]`
- **PATCH /api/bookings/:id/cancel**
  - Output: `{ "bookingId": "uuid", "status": "canceled" }`
  - Validation: Only guest or host involved can cancel according to policy.
- **PATCH /api/bookings/:id/confirm**
  - Output: `{ "bookingId": "uuid", "status": "confirmed" }`
  - Validation: Only host can confirm pending booking.

**Performance Criteria:**

- Booking creation must prevent double-booking using date conflict checks.
- Booking status updates must reflect in real-time for both guest and host.
- System must handle at least 1000 concurrent bookings without failures.

---

## Notes

- **Security:** All endpoints require HTTPS. Sensitive data like passwords must be hashed using bcrypt or equivalent. JWT used for authentication.
- **Database:** PostgreSQL recommended. Tables: Users, Properties, Bookings, Payments, Reviews.
- **Error Handling:** Standardized error messages with HTTP codes (400, 401, 403, 404, 500).
