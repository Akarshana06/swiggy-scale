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