# FAILURE CASCADE ANALYSIS

## Scenario

SwiftEats plans a 50% discount promotion during the India vs Pakistan World Cup Final.

Current Infrastructure:

- Single Node.js Express Server
- 1 CPU
- 4 GB RAM
- Single PostgreSQL Database
- max_connections = 100
- No Redis Cache
- No CDN
- No Load Balancer
- No Auto Scaling

Expected Traffic:

- Total Users = 180 Million
- Click Through Rate = 8%
- Active Users = 14.4 Million

## Traffic Simulation

Total Users = 180,000,000

CTR = 8%

Active Users

= 180,000,000 × 0.08

= 14,400,000 users

Assuming every user performs 5 API requests:

- Open Home Page
- Browse Restaurants
- Open Menu
- Apply Coupon
- Place Order

Total Requests

= 14,400,000 × 5

= 72,000,000 requests

Peak RPS

= 72,000,000 / 60

= 1,200,000 RPS 

## Initial Observation

The current architecture cannot survive the event.

## Expected Traffic:
1,200,000 RPS

Single Node.js Capacity:
15,000 RPS

Traffic exceeds server capacity by approximately 80x.

The system will experience cascading failures beginning with database connection exhaustion.

## Component Capacity Numbers

### PostgreSQL Database

Configuration:

- max_connections = 100

Assumptions:

- 90% requests are normal queries
- 10% requests involve payment processing

Average normal query time:

0.05 seconds

Average payment hold time:

1 second

Formula:

Connections Held =
(0.9 × RPS × 0.05)
+
(0.1 × RPS × 1)

Pool Exhaustion:

100 = (0.145 × RPS)

RPS = 689

Result:

The PostgreSQL connection pool is exhausted at approximately 689 RPS.

This is far below the expected traffic of 1,200,000 RPS.

### Node.js Event Loop

Server Configuration:

- Single Node.js process
- 1 CPU
- 4GB RAM

Approximate Capacity:

12,000–15,000 RPS

Expected Traffic:

1,200,000 RPS

Result:

Incoming traffic exceeds Node.js capacity by approximately 80 times.

The event loop becomes overloaded, request queues increase, and latency rises dramatically.

### Node.js Heap Memory

Available RAM:

4GB

Large request queues accumulate during overload.

At high traffic volumes the process eventually reaches memory limits and crashes with an Out Of Memory (OOM) error.

### Razorpay Payment Calls

Payment API Latency:

200ms – 2000ms

Average Assumption:

1 second

Problem:

The application waits synchronously for payment confirmation.

During this wait the database connection remains occupied.

This amplifies connection pool exhaustion and reduces throughput.

## Failure Cascade

### Failure 1: PostgreSQL Connection Pool Exhaustion

Severity: CRITICAL

Trigger:

~689 RPS

User Impact:

- Orders fail
- Database queries timeout
- Users receive 500 Internal Server Errors

Cascade Effect:

Node.js begins accumulating pending requests.

### Failure 2: Node.js Event Loop Saturation

Severity: CRITICAL

Trigger:

~15,000 RPS

User Impact:

- Slow responses
- Infinite loading screens
- Frequent request timeouts

Cascade Effect:

Request queues consume memory and increase CPU usage.

### Failure 3: Synchronous Payment Call Amplification

Severity: HIGH

Trigger:

High order volume

User Impact:

- Payments remain pending
- Order confirmations delayed
- Increased checkout failures

Cascade Effect:

Database connections remain occupied longer than expected, worsening pool exhaustion.

### Failure 4: Promo Code Race Condition

Severity: HIGH

Trigger:

Millions of users applying coupons simultaneously.

User Impact:

- Incorrect discount calculations
- Coupon abuse
- Failed transactions

Cascade Effect:

Increased database locking and transaction conflicts.

### Failure 5: Static Asset Network Saturation

Severity: MEDIUM

Trigger:

Large image traffic during traffic spike.

User Impact:

- Slow image loading
- Broken restaurant images
- Delayed page rendering

Cascade Effect:

Network bandwidth becomes saturated, leaving fewer resources for API traffic. 

## Incident Timeline

T+0s:
Push notification sent to 180 million users.

T+5s:
Millions of users open the application.

T+10s:
Database connections rapidly increase.

T+15s:
Connection pool reaches maximum capacity.

T+20s:
Database timeouts begin.

T+30s:
Node.js request queue grows significantly.

T+40s:
CPU utilization reaches 100%.

T+60s:
Large-scale API failures observed.

T+5m:
Order placement failures exceed acceptable limits.

T+15m:
Payment failures increase significantly.

T+20m:
Engineering team paged.

T+45m:
Traffic mitigation measures applied.

T+1h:
Partial service restoration.

T+2h:
System fully stabilized and operating normally.