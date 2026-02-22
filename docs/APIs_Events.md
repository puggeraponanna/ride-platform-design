# APIs & Events Definitions

## 1. REST APIs

### 1.1 Ride Request & Matching
- **`POST /v1/rides`**: Rider initiates a ride request.
  ```json
  {
    "rider_id": "uuid",
    "pickup": {"lat": 12.97, "lng": 77.59},
    "destination": {"lat": 12.93, "lng": 77.62},
    "tier": "PREMIUM"
  }
  ```
- **`POST /v1/rides/{id}/cancel`**: Cancel an active ride request.
- **`WS /v1/rides/{id}/status`**: Persistent WebSocket connection for real-time match and trip status updates.

### 1.2 Driver APIs
- **`PUT /v1/drivers/status`**: Update location and availability.
  ```json
  {
    "driver_id": "uuid",
    "status": "ONLINE",
    "location": {"lat": 12.971, "lng": 77.591}
  }
  ```
- **`POST /v1/rides/{id}/accept`**: Driver accepts a matched ride.
- **`POST /v1/rides/{id}/decline`**: Driver declines a matched ride.

### 1.3 Trip Lifecycle
- **`POST /v1/trips/{id}/start`**: Driver starts the trip (OTP verification).
  ```json
  {
    "otp": "123456",
    "timestamp": 1677000500
  }
  ```
- **`POST /v1/trips/{id}/end`**: Complete trip and trigger fare calculation.

### 1.4 Pricing & Payments
- **`GET /v1/pricing/estimate`**: Get estimated fare and surge multiplier.
- **`POST /v1/payments/webhook`**: Callback from external PSPs (Stripe, Braintree).

---

## 2. Kafka Event Topics

### 2.1 `driver-locations` (High Volume)
- **Key**: `h3_cell_id`
- **Payload**: 
  ```json
  {
    "driver_id": "uuid",
    "location": {"lat": 12.97, "lng": 77.59},
    "timestamp": 1677000000
  }
  ```

### 2.2 `ride-events` (State Transitions)
- **Key**: `ride_id`
- **Payload**: 
  ```json
  {
    "event_type": "MATCHED",
    "ride_id": "uuid",
    "driver_id": "uuid",
    "rider_id": "uuid"
  }
  ```

### 2.3 `payment-events` (FinOps)
- **Key**: `payment_id`
- **Payload**: 
  ```json
  {
    "status": "SETTLED",
    "amount": 25.50,
    "ride_id": "uuid"
  }
  ```

### 2.4 `notification-events` (Async Delivery)
- **Key**: `user_id`
- **Payload**: 
  ```json
  {
    "channel": "PUSH",
    "template": "ride_arrival",
    "data": { "eta": "2 mins" }
  }
  ```
