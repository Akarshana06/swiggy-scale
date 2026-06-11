# AWS Cost Estimate

## Goal

Estimate the monthly AWS infrastructure cost required to support SwiftEats under normal traffic conditions and during a World Cup Final traffic surge.

## Baseline Monthly Cost

### EC2 - Node.js Application Servers

Instance Type:

t3.medium

Price:

$0.0416/hour

Quantity:

4

Calculation:

0.0416 × 720 × 4

= $119.81/month 

### RDS PostgreSQL Primary

Instance Type:

db.r6g.large

Price:

$0.252/hour

Calculation:

0.252 × 720

= $181.44/month 

### PostgreSQL Read Replicas

Instance Type:

db.r6g.large

Quantity:

2

Calculation:

0.252 × 720 × 2

= $362.88/month 

### ElastiCache Redis

Instance Type:

cache.t3.medium

Price:

$0.068/hour

Calculation:

0.068 × 720

= $48.96/month 

### Application Load Balancer

Estimated Cost:

$25/month

Includes:

- Load Balancer Hours
- LCU Usage 

### CloudFront CDN

Estimated Monthly Data Transfer:

5 TB

Estimated Cost:

$100/month 

### Amazon SQS

Estimated Messages:

100 million/month

Estimated Cost:

$10/month 

## Baseline Monthly Total

EC2:
$119.81

RDS Primary:
$181.44

Read Replicas:
$362.88

Redis:
$48.96

ALB:
$25

CloudFront:
$100

SQS:
$10

Total:

$848.09/month 

## World Cup Final Peak Cost

Normal Servers:

4

Peak Servers:

20

Additional Servers:

16 

### Extra EC2 During Event

Instance Type:

t3.medium

Price:

$0.0416/hour

Additional Instances:

16

Duration:

4 hours

Calculation:

16 × 0.0416 × 4

= $2.66 

### Extra CloudFront Traffic

Additional Transfer:

10 TB

Estimated Additional Cost:

$50 

### Peak Event Additional Cost

Extra EC2:

$2.66

Extra CloudFront:

$50

Total Extra Cost:

$52.66 

## Business Justification

Estimated Revenue Loss During Outage:

₹4.2 crore per minute

Assuming a 45-minute outage:

4.2 × 45

= ₹189 crore

Monthly Infrastructure Cost:

≈ $848/month

Peak Event Extra Cost:

≈ $53

Conclusion:

Investing less than $1,000 per month in scalable infrastructure is dramatically cheaper than risking an outage capable of causing losses of approximately ₹189 crore during a major event.

The redesigned architecture provides a strong return on investment while significantly improving reliability and scalability. 
