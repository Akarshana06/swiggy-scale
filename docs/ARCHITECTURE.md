# SwiftEats Architecture Redesign

## Goal

Design an architecture capable of handling a World Cup Final traffic surge while preventing the failures identified in FAILURE-CASCADE.md.

## Current Architecture

```text
                    Users
                       |
                       |
                +-------------+
                |  Node.js    |
                |  Monolith   |
                +-------------+
                       |
                       |
                +-------------+
                | PostgreSQL  |
                | Single DB   |
                +-------------+ 

                
---

## Section 2: New Architecture Diagram




## Redesigned Architecture


                           Users
                              |
                              |
                     +----------------+
                     | CloudFront CDN |
                     +----------------+
                              |
                              |
                     +----------------+
                     | Application    |
                     | Load Balancer  |
                     +----------------+
                              |
          -----------------------------------------
          |                 |                     |
          |                 |                     |
   +-------------+   +-------------+     +-------------+
   | Node.js #1  |   | Node.js #2  | ... | Node.js #20 |
   +-------------+   +-------------+     +-------------+
          |
          |
      +---------+
      | Redis   |
      | Cache   |
      +---------+
          |
          |
     +-----------+
     | PgBouncer |
     +-----------+
          |
          |
   +-------------------+
   | PostgreSQL        |
   | Primary Database  |
   +-------------------+
          |
    -----------------
    |               |
    |               |
+----------+   +----------+
| Read Rep |   | Read Rep |
+----------+   +----------+

          |
          |
      +---------+
      |  SQS    |
      +---------+
          |
          |
   +---------------+
   | Payment Worker|
   +---------------+
          |
          |
      Razorpay


---

# Explain Components

```md
### CloudFront CDN

Purpose:

- Serves images
- Serves static files
- Reduces requests reaching Node.js

TTL:

- Restaurant Images: 24 hours
- Static Assets: 7 days
```

---

```md
### Application Load Balancer

Purpose:

- SSL Termination
- Health Checks
- Traffic Distribution
- Rate Limiting

Benefit:

Traffic is spread across multiple application servers instead of a single server.
```

---

```md
### Node.js Auto Scaling

Baseline:

4 instances

World Cup Peak:

20 instances

Scaling Trigger:

- CPU > 70%
- Request Count > Threshold

Benefit:

Capacity grows automatically during traffic spikes.
```

---

```md
### Redis Cache

Stores:

- Restaurant Lists
- Menus
- Promo Data
- Frequently Accessed Reads

TTL:

Restaurant Data = 5 minutes

Promo Data = 1 minute

Benefit:

Reduces database load significantly.
```

---

```md
### PgBouncer

Purpose:

Connection Pooling

Benefit:

Thousands of client requests can share a smaller set of database connections.
```

---

```md
### PostgreSQL Read Replicas

Read Replicas:

2

Purpose:

Read traffic routed to replicas.

Write traffic routed to primary.

Benefit:

Reduces load on primary database.
```

---

```md
### SQS Payment Queue

Purpose:

Decouple payment processing from order placement.

Benefit:

Orders are queued immediately instead of waiting for Razorpay responses.
```

---

```md
### Payment Worker Service

Purpose:

Consumes payment requests from SQS.

Processes payment asynchronously.

Benefit:

Slow payment APIs do not block user requests.
```

---

# Section 3: Component Justification Table

```md
## Component Justification

| Component | Failure Prevented | How It Prevents It |
|------------|------------------|--------------------|
| CloudFront CDN | Static Asset NIC Saturation | Serves images and static assets from edge locations |
| Application Load Balancer | Single Server Bottleneck | Distributes traffic across many servers |
| Multiple Node.js Instances | Event Loop Saturation | Traffic shared among multiple instances |
| Redis Cache | Database Overload | Frequently requested data served from cache |
| PgBouncer | Connection Pool Exhaustion | Reuses database connections efficiently |
| Read Replicas | Read Query Overload | Read traffic distributed away from primary |
| SQS Queue | Payment Amplification | Payment requests buffered asynchronously |
| Payment Workers | Slow Payment Calls | Processes payment requests independently |
| Auto Scaling | Traffic Spikes | Automatically adds capacity during peak demand |
```

---

# Conclusion

```md
## Result

The redesigned architecture removes all major bottlenecks identified in the failure cascade.

The system can scale horizontally, reduce database pressure, isolate payment processing, and serve static content efficiently during large traffic spikes.
```

---

