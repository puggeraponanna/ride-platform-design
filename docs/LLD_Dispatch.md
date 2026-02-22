# Low-Level Design: Dispatch & Matching Service

## 1. Goal
Assign the most suitable driver to a rider request within **<1s (p95)** while preventing race conditions.

```mermaid
sequenceDiagram
    participant R as Rider
    participant AGW as API Gateway
    participant DS as Dispatch Service
    participant SS as Surge Service
    participant LS as Location Service
    participant D as Driver

    R->>AGW: POST /v1/rides (Request)
    AGW->>DS: Forward Request
    DS->>SS: GET /v1/surge (Multiplier)
    SS-->>DS: Return multiplier
    DS->>LS: GET /v1/locations (Nearby Drivers)
    LS-->>DS: Return Driver List
    DS->>DS: Rank Drivers (ETA, Rating, Surge)
    DS->>D: Push/WS Match Offer
    D->>DS: POST /v1/rides/{id}/accept
    DS->>DS: Commit Match (Redis Lock)
    DS-->>AGW: Match Confirmed
    AGW-->>R: WS /v1/rides/{id}/status (Matched)
```

## 2. Core Logic

### 2.1 Spatial Indexing (H3)
- Use **H3 level 7/8** for grouping drivers.
- Redis Sorted Sets for low-latency geo-queries.

### 2.2 Ranking Algorithm
The Dispatch Service first fetches the current **Surge Multiplier** from the Surge Service to calculate the final price.
The ranking score is then calculated as:
$$Score = w_1 \cdot ETA + w_2 \cdot Rating + w_3 \cdot Tier$$
- Configurable weights via feature flags.

### 2.3 Concurrency Control
- **Redis Distributed Locks**: `lock:driver:{id}` to ensure a driver is only matched to one ride at a time.
- **Idempotency**: `request_id` tracking at the API Gateway and Service level.

## 3. Handling Timeouts
- 10s driver acceptance window.
- Automatic re-ranking and failover to the next best driver.
