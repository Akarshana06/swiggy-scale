# Incident Response Runbook

## Purpose

This runbook provides a step-by-step guide for responding to production incidents affecting the SwiftEats platform. It is designed for junior engineers who may be handling incidents during on-call shifts.

---

# STEP 1 - DETECT

## Required CloudWatch Alarms

| Metric               | Threshold            | Severity |
| -------------------- | -------------------- | -------- |
| CPU Utilization      | >80% for 5 min       | Warning  |
| CPU Utilization      | >90% for 5 min       | Critical |
| DB Connections       | >90 connections      | Warning  |
| DB Connections       | >95 connections      | Critical |
| HTTP 5xx Errors      | >5% requests         | Critical |
| API Latency          | >1000 ms             | Warning  |
| API Latency          | >3000 ms             | Critical |
| SQS Queue Depth      | >10,000 messages     | Warning  |
| SQS Queue Depth      | >50,000 messages     | Critical |
| Redis Cache Hit Rate | <80%                 | Warning  |
| ALB Target Health    | Any unhealthy target | Critical |

---

# STEP 2 - TRIAGE

Follow this checklist in order.

### Check Database Connections

If database connections exceed 90:

Root Cause:
Database connection pool exhaustion.

Go to:
STEP 3A

---

### Check CPU Utilization

If CPU exceeds 80%:

Root Cause:
Application compute saturation.

Go to:
STEP 3B

---

### Check SQS Queue Depth

If queue depth is increasing continuously:

Root Cause:
Payment worker backlog.

Go to:
STEP 3C

---

### Check Redis Cache Hit Rate

If hit rate drops below 80%:

Root Cause:
Cache miss spike.

Go to:
STEP 3D

---

### Check HTTP 5xx Errors

If error rate exceeds 5%:

Review application logs and identify failing service.

Escalate if root cause is unclear.

---

# STEP 3 - RESPOND

## STEP 3A - Database Pool Exhaustion

Symptoms:

* Database connections >95
* Query latency increasing
* API timeouts

Actions:

1. Open AWS Console
2. Navigate to RDS Metrics
3. Confirm connection count
4. Verify PgBouncer health
5. Temporarily increase application cache usage
6. Route read traffic to replicas

Success Criteria:

* Connections below 80
* Query latency normalizes within 10 minutes

Responsible Team:

Platform Team

Slack Channel:

#platform-oncall

---

## STEP 3B - Compute Saturation

Symptoms:

* CPU >90%
* Response time increasing

Actions:

Increase Auto Scaling Group capacity.

AWS CLI:

aws autoscaling set-desired-capacity 
--auto-scaling-group-name swift-eats-asg 
--desired-capacity 20

Success Criteria:

* CPU below 70%
* Latency decreases within 10 minutes

Responsible Team:

Backend Team

Slack Channel:

#backend-oncall

---

## STEP 3C - Payment Queue Backup

Symptoms:

* SQS queue depth increasing
* Payment delays

Actions:

1. Open SQS Dashboard
2. Verify queue depth
3. Increase payment worker count

Success Criteria:

* Queue depth decreases steadily
* Payment processing time normalizes

Responsible Team:

Payments Team

Slack Channel:

#payments-oncall

---

## STEP 3D - Redis Cache Miss Spike

Symptoms:

* Cache hit rate below 80%
* Database traffic increasing

Actions:

1. Verify Redis health
2. Warm cache with popular restaurant data
3. Restart failed Redis nodes if required

Success Criteria:

* Cache hit rate above 90%
* Database load decreases

Responsible Team:

Platform Team

Slack Channel:

#platform-oncall

---

# STEP 4 - ROLLBACK

## Rollback Criteria

Perform rollback if:

* Error rate remains above 10%
* Service unavailable for more than 15 minutes
* New deployment identified as root cause

Do NOT rollback for temporary traffic spikes.

---

## Rollback Command

kubectl rollout undo deployment/swifteats-api

---

## Important Warning

Never rollback database schema changes.

Only rollback application code.

Database rollbacks require separate approval and procedure.

---

# STEP 5 - POSTMORTEM

## Incident Summary

Brief description of what happened.

---

## Timeline

List all significant events.

Example:

* 20:00 Alert Triggered
* 20:05 Investigation Started
* 20:20 Root Cause Identified
* 20:40 Mitigation Applied
* 21:00 Service Restored

---

## Root Cause

Describe the primary technical reason for the incident.

---

## Impact

Include:

* Duration
* Users affected
* Revenue impact
* Services affected

---

## What Worked Well

Document successful actions and processes.

---

## What Failed

Document gaps in systems, monitoring, or procedures.

---

## Action Items

| Action Item         | Owner     | Due Date   |
| ------------------- | --------- | ---------- |
| Example Improvement | Team Name | YYYY-MM-DD |

---

## Lessons Learned

Document key takeaways and future prevention measures.
