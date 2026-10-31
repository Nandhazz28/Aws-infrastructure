# 12 - Troubleshooting and Lessons Learned

## Troubleshooting methodology

When a public web server is unreachable, work from the outside in and change one thing at a time.

| Step | Question | Where to look |
|---|---|---|
| 1 | Is the instance running, and do status checks pass? | EC2 instance list (3/3 checks, as in Figure 4.1) |
| 2 | Does it have a public IPv4 or Elastic IP? Did it change after a stop/start? | Instance details |
| 3 | Is the subnet associated with a route table that has `0.0.0.0/0` to the Internet Gateway? | Route table, Routes and Subnet associations tabs |
| 4 | Does the security group allow the port from the client's address? | Security group inbound rules |
| 5 | Does a network ACL block it? | Subnet, Network ACL tab |
| 6 | Is the service running and listening on the port? | `systemctl status nginx`, `ss -tlnp` on the host |
| 7 | Does it respond locally? | `curl -I http://localhost` on the host |
| 8 | What do the logs say? | `/var/log/nginx/error.log`, `journalctl -u nginx` |

Steps 6 to 8 commands are illustrative and were not captured in the evidence.

## Issues observed during this project

| Observation | Source | Handling |
|---|---|---|
| A subnet was deleted during the build | Subnet console banner (Figure 3.1) | Cleanup of an unwanted resource. The VPC page lists four subnets, matching the design. Which subnet was deleted, and why, was not recorded. |
| `setfacl`/`getfacl` required the `acl` package, installed with `apt` | Figure 5.2 | Install the package before configuring ACLs. Record prerequisites in a build script. |
| "Not authorized to call AWS Compute Optimizer" | Figure 8.1 | Non-blocking; the console identity lacks that permission. Grant only if the feature is wanted. |
| "Failed to load rule groups" for DNS Firewall | Figure 2.1 | Non-blocking; cause not investigated. |
| Public IPv4 changes on stop/start | Figure 4.1 (no Elastic IP) | Use an Elastic IP or a load balancer if a stable address is required. |

## Lessons learned

1. **Evidence matters.** Capturing the Routes tab, the Encryption section of the volume and `systemctl status` would have removed several "author-stated" entries from the documentation. Capture the page that proves the claim, not only the page that names the resource.
2. **Account-level defaults do not fix existing resources.** Enabling EBS default encryption protects new volumes only; the existing root volume needs a migration.
3. **Public does not mean exposed everywhere.** Opening 80/443 to the world is normal for a web server; opening SSH is not. Restricting 22 to a narrow range is a deliberate control.
4. **Open ports need listeners.** Allowing 443 in the security group is not the same as having HTTPS working.
5. **Burstable instances need credit monitoring.** CPU utilisation alone does not show throttling risk.
6. **Sanitise before publishing.** Console screenshots routinely include the account ID and public IPs, and terminal output can include real names.
7. **Design before scaling.** Private subnets without NAT or endpoints cannot patch themselves; plan egress before placing workloads there.
8. **Automate.** A hand-built environment is hard to reproduce. Infrastructure as code is the natural next step.

## Future improvements

Terraform or CloudFormation for the VPC and instance; TLS; CloudWatch alarms and agent; encrypted root volume; SSM Session Manager instead of SSH; the ALB, ASG and database tiers in [11](11-load-balancing-and-auto-scaling.md); snapshot automation per [10](10-backup-and-disaster-recovery.md).
