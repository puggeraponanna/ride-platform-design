# Resilience & Failure Mode Analysis

## 1. Resilience Patterns

### 1.1 Circuit Breakers
- Applied to all external PSP (Payment Service Provider) integrations.
- Fail-safe: Queue transaction updates if PSP is down.

### 1.2 Backpressure
- Drop stale location updates in the Go ingestion service if the Redis buffer is full.

### 1.3 Regional Failover
- **Global Load Balancer**: Health checks and automatic routing.
- **Cross-Region Replication**: CockroachDB ensures trip and user metadata is available globally.

## 2. Failure Modes

| Component | Failure | Mitigation |
| :--- | :--- | :--- |
| **Location Tracking** | Redis Cluster Down | Fallback to latest data in SQL DB. |
| **Dispatch Service** | Regional Outage | Traffic rerouted to secondary region. Matching state recovered from CockroachDB. |
| **Payments** | PSP Outage | Exponential backoff retries via Kafka outbox. |
