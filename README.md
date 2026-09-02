# AWS-VPC
## What is VPC?
- **VPC (Virtual Private Cloud)** is an isolated network in AWS.
- It helps secure infrastructure by controlling public and private access.
- Each region has a default VPC, but you can create custom ones.
---
## Key Components
- **Internet Gateway (IGW):** Provides internet access to public subnet.
- **Public Subnet:** Exposed to the internet.
- **Private Subnet:** Internal-only, no direct internet access.
- **NAT Gateway:** Allows private subnet instances to access the internet securely.
- **Routing Tables:**
  - Public RT → Routes traffic to IGW.
  - Private RT → Routes traffic to NAT Gateway.
---
## Setup Steps
1. Create a VPC (CIDR: 10.0.0.0/16).
2. Create and attach an Internet Gateway.
3. Create Public Subnet (10.0.0.0/24).
4. Create Private Subnet (10.0.1.0/24).
5. Create NAT Gateway in Public Subnet with Elastic IP.
6. Configure Public Route Table → `0.0.0.0/0 → IGW`.
7. Configure Private Route Table → `0.0.0.0/0 → NAT Gateway`.
8. Create Security Groups (allow SSH, HTTP, DB traffic).
9. Launch EC2 Jump Server in Public Subnet.
10. Launch EC2 Database Server in Private Subnet.
11. Connect to DB Server via Jump Server.
---
## Notes
- NAT Gateway is AWS-managed (preferred over NAT Instance).
- VPC Endpoints allow access to AWS services (S3, DynamoDB) without internet/NAT.
- VPN can provide secure full internet access for private networks.
---
## Why I Learned This
- ✅ Real-time company infrastructure  
- ✅ Work-from-home secure access  
- ✅ Learning purpose & AWS practice  
---
## Next Learning Goals
- Explore Load Balancers in Public Subnet.
- Practice Auto Scaling for EC2.
- Learn RDS setup in Private Subnet.
