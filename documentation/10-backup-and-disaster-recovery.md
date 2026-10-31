# 10 - Backup and Disaster Recovery

> **Status: conceptual.** No backup job, snapshot schedule or recovery test was implemented or evidenced. The only snapshot visible in the evidence is the AMI snapshot the root volume was created from, which is not a backup of the running server.

## What it is

Backup is a restorable copy of data. Disaster recovery (DR) is the plan and infrastructure to restore service after a major failure such as an AZ outage, an accidental deletion or a compromised account.

## Why it is needed

A single instance in a single Availability Zone with no backups loses everything if the volume or the zone fails. Resilience must be designed, not assumed.

## How it works

- **RPO** (recovery point objective): how much data loss is acceptable. **RTO** (recovery time objective): how long recovery may take.
- EBS snapshots are incremental, stored durably by AWS and can be copied across Regions and restored to new volumes.
- AWS Backup and Data Lifecycle Manager automate schedules, retention and copies.
- DR patterns range from backup and restore (cheapest, slowest) through pilot light and warm standby to multi-site active/active (fastest, costliest).

## Current state versus gaps

| Capability | Current state |
|---|---|
| Snapshot schedule for the root volume | None evidenced |
| Cross-Region copy | None |
| Tested restore | None |
| Infrastructure as code | None; the environment was built by hand |
| Data tier | None; the web server is stateless (default page) |

## Proposed strategy (not implemented)

| Layer | Approach | Illustrative target |
|---|---|---|
| Server configuration | Build an encrypted golden AMI; keep configuration in code (user data or Ansible) | Rebuild in minutes |
| Volumes | Daily snapshots with 7 to 30 day retention via AWS Backup | RPO 24 h |
| Database (future) | Multi-AZ deployment with automated backups and point-in-time recovery | RPO of minutes |
| Regional failure | Copy snapshots and AMIs to a second Region such as `ap-south-2`; keep the network as code | RTO of hours (backup and restore) |
| Account-level protection | Separate backup account or vault lock for ransomware resilience | n/a |
| Validation | Scheduled restore test and documented runbook | Quarterly |

The RPO and RTO values above are illustrative and must be set from real business requirements.

## Security implications

Backups inherit the sensitivity of the source. Encrypt them, restrict access to the backup vault, and protect against deletion by a compromised administrator.

## Cost implications

Snapshots are billed per GB-month of stored changed data. Cross-Region copies add storage and data-transfer costs. A warm standby or active/active design roughly doubles infrastructure cost, so match the pattern to the required RTO.

## Real-world use cases

Recovering from accidental deletion, ransomware, failed deployments, AZ or Region outages, and satisfying audit requirements for retention.

## Implemented versus studied

Implemented: none. Studied: all of the above.
