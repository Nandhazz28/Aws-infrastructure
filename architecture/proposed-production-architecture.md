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
