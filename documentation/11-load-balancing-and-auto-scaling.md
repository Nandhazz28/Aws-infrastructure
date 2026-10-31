# 11 - Load Balancing and Auto Scaling

> **Status: conceptual.** No Application Load Balancer, target group, launch template or Auto Scaling group was created. See the [proposed architecture diagram](../architecture/README.md#proposed-production-architecture).

## What it is

An **Application Load Balancer (ALB)** distributes HTTP/HTTPS requests across healthy targets in multiple Availability Zones. An **Auto Scaling group (ASG)** keeps a desired number of identical instances running and adjusts that number based on demand or health.

## Why it is needed

The current design has a single instance in a single AZ: one failure causes an outage and one instance bounds capacity. An ALB plus an ASG removes the single point of failure and lets capacity follow load.

## How it works

1. The ALB sits in the public subnets and accepts traffic from the Internet Gateway.
2. A listener (80/443) forwards to a target group. Health checks mark instances healthy or unhealthy.
3. The ASG launches instances from a **launch template** (AMI, instance type, security group, user data, IAM instance profile) into the private subnets of both AZs.
4. Scaling policies, for example target tracking on average CPU or request count per target, add or remove instances. Unhealthy instances are replaced.
5. Application instances in private subnets only accept traffic from the ALB's security group, and reach the internet for updates through NAT gateways.

## Proposed design for this project

| Component | Proposal |
|---|---|
| ALB | Internet-facing, in Public-Subnet-1 and Public-Subnet-2; HTTPS listener with an ACM certificate; HTTP redirected to HTTPS |
| Target group | HTTP on port 80, health check on `/` |
| Launch template | Encrypted AMI with Nginx, `t3.micro` or similar, instance role with SSM and CloudWatch permissions |
| ASG | Minimum 2, desired 2, maximum 4 (illustrative), spanning Private-Subnet-1 and Private-Subnet-2 |
| Scaling policy | Target tracking, around 50% average CPU, or ALB request count per target |
| Security groups | ALB SG: 80/443 from the internet. App SG: 80 only from the ALB SG. No inbound SSH; use SSM Session Manager |
| Database tier | New private subnets (for example 10.20.21.0/24 and 10.20.22.0/24) with a Multi-AZ managed database, reachable only from the app SG |

The 10.20.21.0/24 and 10.20.22.0/24 subnets do not exist today; they are a proposal.

## Security implications

Terminating TLS at the ALB moves certificate management to ACM and removes public exposure of the instances. Security-group references between tiers are stronger than CIDR rules. Instances must be stateless, with sessions and data stored outside them.

## Cost implications

An ALB is billed per hour plus load-balancer capacity units. NAT gateways add hourly and per-GB charges (one per AZ for resilience). Two or more instances double compute cost relative to today. Scheduled scaling or a lower minimum can reduce non-production cost.

## Real-world use cases

Public web applications and APIs, blue/green deployments, absorbing traffic spikes, and rolling instance replacement for patching.

## Implemented versus studied

Implemented: none. Studied: ALB, target groups, launch templates, ASG scaling policies, multi-tier security group design.
