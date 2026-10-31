# 01 - Project Overview

## Purpose

This project documents a small AWS environment built in the Asia Pacific (Mumbai) region (`ap-south-1`): a custom VPC with public and private subnets, a single public Ubuntu web server running Nginx, restricted network access through a security group, Linux user and permission management, and basic monitoring and storage inspection.

It also describes, separately and clearly labelled, how the environment would be extended into a production-grade design (load balancing, Auto Scaling, a private database tier, backup and disaster recovery). That second part is a design exercise only.

## Evidence classification

Every claim in this repository is tagged with one of the following statuses.

| Status | Meaning |
|---|---|
| **Verified** | Visible in a screenshot in `screenshots/`. |
| **Author-stated** | Reported by the project author, but not visible in any screenshot. Listed in [missing-evidence-checklist.md](missing-evidence-checklist.md). |
| **Inspected** | A console page was viewed, but nothing was configured by this project. |
| **Conceptual** | Designed or studied only. Nothing is deployed. |

## Scope summary

| Area | Status | Notes |
|---|---|---|
| VPC `production-vpc` (10.20.0.0/16) | Verified | [02](02-vpc-and-networking.md) |
| Four subnets (2 public, 2 private) | Verified (CIDRs); AZ of three subnets author-stated | [03](03-public-and-private-subnets.md) |
| Internet Gateway `production-igw` attached | Verified | [02](02-vpc-and-networking.md) |
| Public and private route tables with 2 subnet associations each | Verified (associations); routes author-stated | [02](02-vpc-and-networking.md) |
| EC2 `t3.micro`, Ubuntu, 3/3 status checks | Verified | [04](04-ec2-and-nginx-deployment.md) |
| Nginx default page | Verified (page); service status and URL not captured | [04](04-ec2-and-nginx-deployment.md) |
| Security group `production-ec2-sg` | Verified (inbound rules); outbound rule not captured | [07](07-security-architecture.md) |
| Linux users, group, setgid directory, ACL | Verified | [05](05-linux-administration.md) |
| EBS root volume gp3 8 GiB | Verified | [08](08-ebs-and-storage.md) |
| Account-level EBS default encryption | Verified (inspected) | [08](08-ebs-and-storage.md) |
| Root volume encryption state | Author-stated (unencrypted); screenshot cuts off before the field | [08](08-ebs-and-storage.md) |
| CloudWatch basic monitoring | Verified (inspected) | [09](09-cloudwatch-monitoring.md) |
| IAM | Conceptual; no IAM evidence captured | [06](06-iam-and-access-management.md) |
| Backup and disaster recovery | Conceptual | [10](10-backup-and-disaster-recovery.md) |
| ALB, Auto Scaling, private database tier | Conceptual | [11](11-load-balancing-and-auto-scaling.md) |

## Repository map

| Path | Content |
|---|---|
| `README.md` | Project summary and gallery |
| `architecture/` | Implemented and proposed architecture diagrams (Mermaid) |
| `documentation/` | One document per topic, 01 to 12, plus the missing-evidence checklist |
| `linux/` | Linux administration notes tied to the screenshots |
| `screenshots/` | Sanitised console and terminal evidence, grouped by topic |

## Sanitisation

The screenshots were captured from a live AWS account. Account IDs, the public IP address and a personal name were blacked out before publication. See [screenshots/README.md](../screenshots/README.md).
