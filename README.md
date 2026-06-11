# Swiggy Scale Simulation

## World Cup Final Traffic Surge Analysis, Architecture Redesign, AWS Cost Estimation, and Incident Response Runbook

This project simulates how a Swiggy-like food delivery platform (SwiftEats) would behave during a massive traffic spike caused by an India vs Pakistan World Cup Final promotion campaign.

The objective is to analyze the failure points of a monolithic architecture, redesign the system for large-scale traffic, estimate infrastructure costs, and create an incident response runbook for production outages.

---

# Scenario

SwiftEats currently operates on a simple monolithic architecture:

* Single Node.js Express Server
* Single PostgreSQL Database
* No Redis Cache
* No CDN
* No Load Balancer
* No Auto Scaling
* Synchronous Payment Processing

A 50% discount promotion is sent to:

* 180 Million Users

Expected Click Through Rate:

* 8%

Expected Active Users:

* 14.4 Million Users

Traffic arrives within a 60-second window during the World Cup Final.

The goal is to determine whether the system can survive the traffic spike and design a scalable alternative architecture.

---

# Repository Structure

```text
swiggy-scale/
│
├── docs/
│   ├── FAILURE-CASCADE.md
│   ├── ARCHITECTURE.md
│   ├── COST-ESTIMATE.md
│   └── RUNBOOK.md
│
├── assets/
│
└── README.md
```

---

# Document Summary

| Document           | Description                                                                      |
| ------------------ | -------------------------------------------------------------------------------- |
| FAILURE-CASCADE.md | Traffic simulation, capacity calculations, failure analysis, and outage timeline |
| ARCHITECTURE.md    | Scalable architecture redesign capable of handling large traffic spikes          |
| COST-ESTIMATE.md   | AWS infrastructure cost calculations for baseline and peak traffic scenarios     |
| RUNBOOK.md         | Step-by-step incident response guide for production outages                      |

---

# Key Findings

### Peak Traffic

14.4 million users generate:

72 million API requests

Resulting in:

1.2 million requests per second (RPS)

---

### PostgreSQL Failure Point

Database connection pool exhaustion occurs at approximately:

689 RPS

Configured limit:

100 Connections

---

### Node.js Saturation

Single Node.js server capacity:

12,000–15,000 RPS

Expected traffic exceeds capacity by approximately:

80x

---

### Payment Processing Amplification

Synchronous payment calls hold database connections for up to:

2 seconds

This significantly increases connection pool exhaustion risk.

---

### Business Impact

Estimated outage cost:

₹4.2 crore per minute

A 45-minute outage could result in:

₹189 crore in losses

---

# Architecture Overview

The redesigned architecture introduces multiple layers to improve scalability and reliability:

* CloudFront CDN for static content delivery
* Application Load Balancer for traffic distribution
* Auto-scaled Node.js application servers
* Redis cache layer
* PgBouncer connection pooling
* PostgreSQL primary database with read replicas
* Amazon SQS payment queue
* Dedicated payment worker services

This architecture eliminates the primary bottlenecks identified during failure analysis and supports large-scale traffic events.

---

# Technologies Covered

The analysis focuses on the following technologies:

* Node.js
* PostgreSQL
* Redis
* AWS EC2
* AWS RDS
* AWS ElastiCache
* AWS CloudFront
* AWS Application Load Balancer
* Amazon SQS

---

# Conclusion

The original monolithic architecture cannot sustain World Cup Final traffic levels and fails due to cascading bottlenecks in database connections, application capacity, payment processing, and static asset delivery.

The redesigned architecture provides horizontal scalability, fault isolation, and operational resilience while maintaining a relatively low infrastructure cost compared to potential outage losses.
