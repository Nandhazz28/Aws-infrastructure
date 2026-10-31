# Screenshots

All images are real captures from the AWS console (Mumbai region) and a browser-based terminal session, taken on 31 October 2026. None are mock-ups or stock images.

## Layout

| Folder | Content |
|---|---|
| `networking/` | VPC, subnets, Internet Gateway, route tables |
| `ec2-linux/` | EC2 instance, Nginx page, Linux users, groups and ACLs |
| `security/` | Security group rules |
| `storage/` | EBS volume and EBS encryption settings |
| `monitoring/` | EC2 monitoring (CloudWatch) charts |

## Sanitisation

Before publication, the following were covered with solid black boxes:

- AWS account ID (console header, Owner columns, resource ARNs)
- Console user name (header)
- The instance's public IPv4 address
- A personal name stored in a Linux account's comment field
- All AWS resource identifiers: VPC, subnet, route table, internet gateway, EC2 instance, EBS volume, snapshot, security group and rule, network ACL, DHCP option set and the KMS key ARN. Resource **names** (for example `production-vpc`, `production-igw`, `production-ec2-sg`) are left visible because they are the evidence and are not identifiers.

The SSH source range 13.233.177.0/29 (an AWS service address range), private IP 10.20.1.254 and CIDR blocks were left visible. The unredacted originals are not part of this repository and must stay private.

Redaction was done by solid black boxes (pixels replaced, not blurred), located with OCR plus manual boxes, then re-checked by OCR and by eye. Because some console lists were clipped at the edge, partially visible rows were covered as well. **Any new screenshot added later must be redacted the same way before it is committed.**

Images are 1855 x 916 px PNG (the Nginx page is 1910 x 915), between roughly 100 and 210 KB each, so text remains readable.

## Index

| File | Documented in |
|---|---|
| `networking/vpc-configuration.png` | [02](../documentation/02-vpc-and-networking.md) |
| `networking/internet-gateway.png` | [02](../documentation/02-vpc-and-networking.md) |
| `networking/public-route-table.png` | [02](../documentation/02-vpc-and-networking.md) |
| `networking/subnet-configuration.png` | [03](../documentation/03-public-and-private-subnets.md) |
| `ec2-linux/ec2-instance-details.png` | [04](../documentation/04-ec2-and-nginx-deployment.md) |
| `ec2-linux/nginx-default-page.png` | [04](../documentation/04-ec2-and-nginx-deployment.md) |
| `ec2-linux/linux-users-groups-acl.png` | [05](../documentation/05-linux-administration.md) |
| `ec2-linux/linux-acl-install-and-user-creation.png` | [05](../documentation/05-linux-administration.md) |
| `security/security-group-rules.png` | [07](../documentation/07-security-architecture.md) |
| `storage/ebs-volume-details.png` | [08](../documentation/08-ebs-and-storage.md) |
| `storage/ebs-encryption-settings.png` | [08](../documentation/08-ebs-and-storage.md) |
| `monitoring/cloudwatch-monitoring.png` | [09](../documentation/09-cloudwatch-monitoring.md) |

Screenshots still needed are listed in the [missing-evidence checklist](../documentation/missing-evidence-checklist.md).
