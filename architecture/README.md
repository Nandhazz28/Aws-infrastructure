# Architecture

Two separate designs are documented here. Only the first is deployed.

| | Implemented architecture | Proposed production architecture |
|---|---|---|
| Source file | [`implemented-architecture.mmd`](implemented-architecture.mmd) | [`proposed-production-architecture.mmd`](proposed-production-architecture.mmd) |
| Status | Deployed and (mostly) verified by screenshots | Conceptual design only |
| Colour in diagram | Green solid outline = deployed; grey dashed = deployed but empty | Orange dashed = proposed; green = already deployed |

Both files are valid Mermaid flowcharts and render directly on GitHub. The diagrams below are the same content as the `.mmd` files; edit the `.mmd` file and update the copy here when changing either.

## Implemented architecture

```mermaid
flowchart TB
    users(["Internet users<br/>HTTP 80 / HTTPS 443"])
    admin(["Administrator<br/>SSH 22 from 13.233.177.0/29"])

    subgraph region["AWS Region: ap-south-1 (Mumbai)"]
        subgraph vpc["VPC: production-vpc 10.20.0.0/16"]
            igw["Internet Gateway<br/>production-igw"]
            pubrt["Route table<br/>production-public-rt"]
            privrt["Route table<br/>production-private-rt"]

            subgraph az1["Availability Zone ap-south-1a"]
                subgraph pub1["Public-Subnet-1 10.20.1.0/24"]
                    ec2["EC2: production-web-server-public<br/>t3.micro, Ubuntu Server 24.04, Nginx<br/>Security group: production-ec2-sg<br/>EBS root volume: gp3, 8 GiB"]
                end
                subgraph priv1["Private-Subnet-1 10.20.11.0/24"]
                    priv1n["No instances deployed"]
                end
            end

            subgraph az2["Availability Zone ap-south-1b"]
                subgraph pub2["Public-Subnet-2 10.20.2.0/24"]
                    pub2n["No instances deployed"]
                end
                subgraph priv2["Private-Subnet-2 10.20.12.0/24"]
                    priv2n["No instances deployed"]
                end
            end
        end
        cw["Amazon CloudWatch<br/>basic EC2 monitoring"]
    end

    users -->|"HTTP / HTTPS"| igw
    admin -->|"SSH"| igw
    igw <-->|"permitted by security group"| ec2
    pubrt -.->|"associated"| pub1
    pubrt -.->|"associated"| pub2
    privrt -.->|"associated"| priv1
    privrt -.->|"associated"| priv2
    ec2 -.->|"metrics"| cw

    classDef deployed fill:#e6f4ea,stroke:#1e8e3e,color:#000
    classDef empty fill:#f1f3f4,stroke:#9aa0a6,stroke-dasharray:4 3,color:#000
    class ec2,igw,pubrt,privrt,cw deployed
    class priv1n,pub2n,priv2n empty
```

### What the diagram shows

- A single VPC (`production-vpc`, 10.20.0.0/16) across two Availability Zones, with two public and two private subnets.
- One Internet Gateway; one public and one private route table, each associated with two subnets.
- One EC2 instance in Public-Subnet-1 protected by a security group, running Nginx, with a gp3 root volume.
- Basic CloudWatch monitoring.
- Traffic: internet users reach the instance on 80/443 and the administrator on 22 (restricted source range), always through the Internet Gateway. Dotted lines are route-table associations and metrics, not traffic.
- Three subnets contain no workloads.

### Evidence caveats

The Availability Zones of Public-Subnet-1 and both private subnets, the subnet-to-route-table mapping, and the routes themselves are author-stated or inferred, not visible in screenshots. See the [missing-evidence checklist](../documentation/missing-evidence-checklist.md).

## Proposed production architecture

```mermaid
flowchart TB
    users(["Internet users"])

    subgraph region["AWS Region: ap-south-1 (Mumbai) - PROPOSED DESIGN, NOT DEPLOYED"]
        subgraph vpc["VPC: production-vpc 10.20.0.0/16 (deployed)"]
            igw["Internet Gateway: production-igw<br/>(deployed)"]
            alb["PROPOSED: Application Load Balancer<br/>spans both public subnets"]

            subgraph az1["Availability Zone ap-south-1a"]
                subgraph pub1["Public-Subnet-1 10.20.1.0/24 (deployed)"]
                    nat1["PROPOSED: NAT Gateway"]
                end
                subgraph app1["Private-Subnet-1 10.20.11.0/24 (deployed, application tier)"]
                    ec2a["PROPOSED: application instance<br/>Nginx, Auto Scaling Group member"]
                end
                subgraph db1["Database subnet 10.20.21.0/24 (PROPOSED, new)"]
                    dbp["PROPOSED: primary database"]
                end
            end

            subgraph az2["Availability Zone ap-south-1b"]
                subgraph pub2["Public-Subnet-2 10.20.2.0/24 (deployed)"]
                    nat2["PROPOSED: NAT Gateway"]
                end
                subgraph app2["Private-Subnet-2 10.20.12.0/24 (deployed, application tier)"]
                    ec2b["PROPOSED: application instance<br/>Nginx, Auto Scaling Group member"]
                end
                subgraph db2["Database subnet 10.20.22.0/24 (PROPOSED, new)"]
                    dbs["PROPOSED: standby database"]
                end
            end
        end

        asg["PROPOSED: Auto Scaling Group<br/>minimum 2 instances across both AZs"]
        cw["PROPOSED: CloudWatch metrics, logs and alarms"]
        bk["PROPOSED: AWS Backup and EBS snapshots<br/>with copy to a second region"]
    end

    users -->|"HTTP / HTTPS"| igw
    igw --> alb
    alb -->|"HTTP"| ec2a
    alb -->|"HTTP"| ec2b
    ec2a -->|"SQL"| dbp
    ec2b -->|"SQL"| dbp
    dbp <-->|"replication"| dbs
    ec2a -->|"outbound updates"| nat1
    ec2b -->|"outbound updates"| nat2
    nat1 --> igw
    nat2 --> igw
    asg -.->|"launches and replaces"| ec2a
    asg -.->|"launches and replaces"| ec2b
    ec2a -.->|"metrics and logs"| cw
    ec2b -.->|"metrics and logs"| cw
    dbp -.->|"backups"| bk

    classDef deployed fill:#e6f4ea,stroke:#1e8e3e,color:#000
    classDef proposed fill:#fff4e0,stroke:#ef6c00,stroke-dasharray:5 4,color:#000
    class igw deployed
    class alb,nat1,nat2,ec2a,ec2b,dbp,dbs,asg,cw,bk proposed
```

### What the diagram proposes

- An Application Load Balancer in the public subnets as the only internet-facing entry point.
- An Auto Scaling group of application instances across the two private subnets.
- A new pair of private database subnets (10.20.21.0/24 and 10.20.22.0/24, not yet created) with a primary and standby database in different Availability Zones.
- NAT gateways for outbound updates, CloudWatch for metrics and logs, and AWS Backup with a copy in a second Region.

**None of the orange components exist.** See [11](../documentation/11-load-balancing-and-auto-scaling.md) and [10](../documentation/10-backup-and-disaster-recovery.md) for the reasoning.

## Component status table

| Component | Status |
|---|---|
| VPC, four subnets, Internet Gateway, two route tables | Deployed |
| EC2 instance, Nginx, security group, EBS root volume | Deployed |
| CloudWatch basic monitoring | Deployed (default behaviour) |
| Linux users, group, setgid directory, ACL | Deployed |
| Application Load Balancer, target group | Proposed |
| Auto Scaling group, launch template | Proposed |
| Application instances in private subnets | Proposed |
| NAT gateways | Proposed |
| Database subnets and database tier | Proposed |
| Alarms, centralised logging | Proposed |
| AWS Backup, snapshots, cross-Region copy | Proposed |
