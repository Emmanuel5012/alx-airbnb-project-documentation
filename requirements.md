# Backend System Requirements

## 1. User Authentication

### Functional Requirements
1. **User Registration**
   - Users can register with email and password.
   - System assigns a default `guest` role.
   - Email verification required before account activation.

2. **User Login**
   - Successful login returns a JWT token with `userId` and `role`.
   - Failed attempts trigger a 5-minute lockout after 3 tries.

3. **Password Reset**
   - Users can request a password reset link via email.
   - Reset links expire in 24 hours.

### Technical Specifications
#### API Endpoints
| Method | Endpoint               | Request Body                                                                 | Success Response (200)                          |
|--------|------------------------|-----------------------------------------------------------------------------|------------------------------------------------|
| POST   | `/auth/register`       | `{"email": "user@example.com", "password": "SecurePass123!"}`               | `{"token": "jwt.xyz", "userId": "123"}`        |
| POST   | `/auth/login`          | `{"email": "user@example.com", "password": "SecurePass123!"}`               | `{"token": "jwt.xyz", "role": "guest"}`        |
| POST   | `/auth/reset-password` | `{"email": "user@example.com"}`                                             | `{"message": "Reset link sent"}`               |

#### Validation Rules
- **Email**: Valid RFC 5322 format, unique in database.
- **Password**: Minimum 8 characters, at least 1 uppercase letter and 1 special character.

#### Performance
- Max latency: 500ms for all auth endpoints.
- Throughput: Support 1,000 requests/minute.

---

## 2. Property Management

### Functional Requirements
1. **Create/Update Property**
   - Hosts can add properties with title, price, location, and amenities.
   - Hosts can edit only their own properties.

2. **Property Search**
   - Guests can filter by location, price range, and availability dates.
   - Results paginated (20 items/page).

### Technical Specifications
#### API Endpoints
| Method | Endpoint                | Query Params                     | Success Response (200)                          |
|--------|-------------------------|----------------------------------|------------------------------------------------|
| POST   | `/properties`           | `{"title": "Beach Villa", "price": 200, ...}` | `{"propertyId": "789"}`        |
| GET    | `/properties`           | `?location=NY&maxPrice=100`      | `{"results": [...], "total": 25}`              |

#### Data Validation
- **Price**: Positive number, ≤ ₦10,000/night.
- **Location**: Non-empty string, validated against Google Maps API.

#### Performance
- Search latency: ≤ 1s for ₦10,000 properties in database.
- DB: Indexes on `location` and `price` fields.

---

## 3. Booking System

### Functional Requirements
1. **Create Booking**
   - Guests can book available properties with valid payment.
   - Overlapping dates are rejected.

2. **Payment Processing**
   - Integrate with Stripe for credit card payments.
   - Retry failed payments once before rejecting.

### Technical Specifications
#### API Endpoints
| Method | Endpoint         | Request Body                                                                 | Error Responses (4xx)                          |
|--------|------------------|-----------------------------------------------------------------------------|------------------------------------------------|
| POST   | `/bookings`      | `{"propertyId": "789", "dates": {"checkIn": "2025-07-01", ...}}`            | `400` - Invalid dates, `402` - Payment failed |

#### Business Rules
- **Availability**: Reject if property is booked for ≥ 50% of requested dates.
- **Payment**: Charge full amount at booking time.

#### Performance
- Booking creation: ≤ 800ms (including Stripe API call).
- Concurrent bookings: Support 500 bookings/minute.

---

## Cross-Cutting Requirements
### Security
- All endpoints use HTTPS.
- JWT tokens expire in 24 hours.

### Reliability
- 99.9% uptime SLA for auth and booking services.
- Database backups every 6 hours.

### Monitoring
- Log all auth failures and booking transactions.
- Alert on >5% payment failure rate.
