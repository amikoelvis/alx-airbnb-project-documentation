# 🏡 Airbnb Clone Backend - Detailed Requirements Specification

## 📘 Table of Contents

1. [User Authentication](#1-user-authentication)
2. [Property Management](#2-property-management)
3. [Booking System](#3-booking-system)
4. [Payment Processing](#4-payment-processing)
5. [Search & Filtering](#5-search--filtering)
6. [Review & Ratings](#6-review--ratings)
7. [Notifications](#7-notifications)
8. [Admin Dashboard](#8-admin-dashboard)

---

## 1. User Authentication

### 🔐 Sign Up

**Endpoint:** `POST /api/auth/register`

**Request Body:**

```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "Password123!",
  "role": "guest" // or "host"
}
```

**Validations:**

- Email must be unique and valid format.
- Password must be min 8 chars, include uppercase, number, special char.
- Role must be either guest or host.

Response:

```json
{
  "user": { "id": 1, "email": "john@example.com", "role": "guest" },
  "token": "jwt-token"
}
```

### 🔑 Login

**Endpoint:** `POST /api/auth/login`
**Request Body:**

```json
{
  "email": "john@example.com",
  "password": "Password123!"
}
```

**Response:**

```json
{
  "token": "jwt-token",
  "user": {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
  }
}
```

### 👤 Profile Management

**Endpoint:** `GET /api/users/me`
Auth Required: ✅
**Response:**

```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com",
  "profilePhoto": "/uploads/photo.jpg",
  "role": "host"
}
```

**Endpoint:** `PUT /api/users/me`
**Request Body:**

```json
{
  "name": "John D.",
  "profilePhoto": "/uploads/new.jpg"
}
```

## 2. Property Management

### 🏠 Create Property

**Endpoint:** `POST /api/properties`
**Auth:** Host only
**Request Body:**

```json
{
  "title": "Cozy Beach House",
  "description": "A great stay by the sea.",
  "location": "Miami, FL",
  "pricePerNight": 120,
  "amenities": ["wifi", "pool", "air_conditioning"],
  "images": ["img1.jpg", "img2.jpg"],
  "maxGuests": 4,
  "availableFrom": "2025-07-01",
  "availableTo": "2025-12-31"
}
```

**Validations:**

- Price must be a positive number.
- Dates must be in valid ISO format.

### ✏️ Update Property

**Endpoint:** `PUT /api/properties/:id`
**Auth:** Host only
**Validations:** Must own the property.

### ❌ Delete Property

**Endpoint:** `DELETE /api/properties/:id`
**Auth:** Host only

### 📄 Get Host Listings

**Endpoint:** GET /api/properties/my
**Auth:** Host only
**Response:** List of property objects

## 3. Booking System

### 📅 Create Booking

**Endpoint:** `POST /api/bookings`
**Request Body:**

```json
{
  "propertyId": 1,
  "checkIn": "2025-08-10",
  "checkOut": "2025-08-15"
}
```

**Validations:**

- Check-in < Check-out.
- No overlapping booking exists.
- Property is available within that date range.

**Response:**

```json
{
  "id": 123,
  "status": "pending",
  "totalPrice": 600
}
```

### ❌ Cancel Booking

**Endpoint:** `DELETE /api/bookings/:id`
**Auth:** Guest or Host
**Rules:** Cancellation allowed up to 24 hours before check-in.

### 📖 View Bookings

**Endpoint:** `GET /api/bookings/my`
**Response:** List of user's bookings

## 4. Payment Processing

### 💳 Create Payment Session

Endpoint: `POST /api/payments/create-session`
**Request Body:**

```json
{
  "bookingId": 123
}
```

**Response:** Stripe session URL

### 🧾 Webhook for Payment Confirmation

**Endpoint:** `POST /api/payments/webhook`
**Handled by:** Stripe

**Updates:**

- Booking marked as confirmed
- Payment marked as successful

## 5. Search & Filtering

### 🔍 Search Listings

**Endpoint:** `GET /api/properties/search`

**Query Parameters:**

- location=miami
- minPrice=50
- maxPrice=200
- guests=2
- amenities=wifi,pool
- page=1

**Response:**

```json
{
  "total": 100,
  "page": 1,
  "results": [{...}, {...}]
}
```

**Performance:**

- Cached with Redis (location + filters).
- Response in <300ms for 90% of requests.

## 6. Review & Ratings

### ✍️ Post Review

**Endpoint:** POST /api/reviews
**Request Body:**

```json
{
  "bookingId": 123,
  "rating": 4,
  "comment": "Nice place!"
}
```

**Rules:**

- Only guests can review.
- Only after booking is completed.

### 📄 Get Property Reviews

**Endpoint:** GET /api/properties/:id/reviews

**Response:**

```json
[
  {
    "user": { "name": "Alice" },
    "rating": 5,
    "comment": "Amazing!",
    "createdAt": "2025-06-20"
  }
]
```

## 7. Notifications

### 📧 Email Events

**Trigger on:**

- Booking created
- Payment completed
- Booking canceled

**Tools:** SendGrid / Mailgun

### 🔔 In-App Notifications

**Endpoint:** `GET /api/notifications`

**Response:** List of notifications (unread/read)

## 8. Admin Dashboard

### 👥 Get All Users

**Endpoint:** `GET /api/admin/users`
**Auth:** Admin only

### 🏘 Manage Listings

**Endpoint:** `GET /api/admin/properties`
**Actions:** Block/unblock, delete

### 📊 Booking Insights

**Endpoint:** `GET /api/admin/bookings`
**Features:** Sort/filter by date, user, property
