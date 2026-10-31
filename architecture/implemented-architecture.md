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
