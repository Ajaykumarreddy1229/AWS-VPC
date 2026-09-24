# AWS VPC 

## 1. What is VPC?

**VPC (Virtual Private Cloud)** is a logically isolated network inside AWS.

A VPC allows us to control:

* IP address ranges
* Public and private subnets
* Routing
* Internet connectivity
* Network security
* Communication between AWS resources

Example:

```text
AWS
 |
 +-- VPC
      |
      +-- Public Subnet
      |
      +-- Private Subnet
```

---

# 2. VPC CIDR

CIDR defines the IP address range of the VPC.

Example:

```text
10.0.0.0/16
```

The VPC CIDR can be divided into smaller subnet ranges.

Example:

```text
VPC
10.0.0.0/16
    |
    +-- Public Subnet
    |   10.0.0.0/24
    |
    +-- Private Subnet
        10.0.1.0/24
```

---

# 3. Subnets

A subnet is a smaller network inside a VPC.

There are two common types:

## Public Subnet

A public subnet has a route to an Internet Gateway.

Typical resources:

* Application Load Balancer
* Bastion Host
* Public-facing resources

Example:

```text
Public Subnet
10.0.0.0/24
      |
      +-- EC2
      +-- ALB
```

## Private Subnet

A private subnet does not have a direct route to the Internet Gateway.

Typical resources:

* Application Servers
* Backend Servers
* Databases

Example:

```text
Private Subnet
10.0.1.0/24
      |
      +-- EC2
      +-- Application Server
      +-- Database
```

---

# 4. Internet Gateway

**Internet Gateway (IGW)** provides internet connectivity for resources in public subnets.

Example:

```text
Internet
    |
    v
Internet Gateway
    |
    v
Public Subnet
    |
    v
EC2
```

Public route:

```text
Destination       Target

0.0.0.0/0         Internet Gateway
```

---

# 5. Route Tables

A route table controls where network traffic is sent.

## Public Route Table

Example:

```text
Destination       Target
--------------------------------
10.0.0.0/16       local
0.0.0.0/0         Internet Gateway
```

Traffic:

```text
EC2
 |
 v
Public Route Table
 |
 v
Internet Gateway
 |
 v
Internet
```

## Private Route Table

Example:

```text
Destination       Target
--------------------------------
10.0.0.0/16       local
0.0.0.0/0         NAT Gateway
```

Traffic:

```text
Private EC2
     |
     v
Private Route Table
     |
     v
NAT Gateway
     |
     v
Internet Gateway
     |
     v
Internet
```

---

# 6. NAT Gateway

**NAT Gateway (Network Address Translation Gateway)** allows resources in private subnets to access the internet for outbound connections.

Example:

```text
Private EC2
     |
     v
NAT Gateway
     |
     v
Internet Gateway
     |
     v
Internet
```

Examples of why a private server may need outbound internet access:

* Download packages
* Install software
* Update packages
* Download application dependencies

### Important

NAT Gateway is normally created in a **public subnet**.

---

# 7. Security Groups

A **Security Group** acts as a virtual firewall for AWS resources such as EC2.

It controls network traffic using rules.

Example:

```text
Web Server Security Group

HTTP   → 80
HTTPS  → 443
SSH    → 22
```

For better security, SSH access should normally be restricted to trusted sources.

---

# 8. Security Group Architecture

A multi-tier application can use separate security groups.

```text
Internet
   |
   v
ALB Security Group
   |
   v
Application Security Group
   |
   v
Database Security Group
```

Example:

```text
ALB
 |
 | HTTP/HTTPS
 v
Application Server
 |
 | Database Port
 v
RDS
```

The database should normally allow traffic from the application tier rather than directly from the public internet.

---

# 9. Bastion Host

A **Bastion Host**, also called a Jump Server, can be used to access private servers for administration.

Example:

```text
Administrator
      |
      | SSH
      v
Bastion Host
Public Subnet
      |
      | SSH
      v
Private EC2
```

The private EC2 does not need to have a public IP for this administration pattern.

---

# 10. VPC Endpoints

VPC Endpoints allow private resources to access supported AWS services without requiring internet connectivity through a NAT Gateway.

Example:

```text
Private EC2
     |
     v
VPC Endpoint
     |
     v
Amazon S3
```

VPC endpoints can be useful for private workloads that need access to AWS services.

---

# 11. Availability Zones

AWS Regions contain multiple Availability Zones.

For highly available architectures, resources can be distributed across multiple Availability Zones.

Example:

```text
                 VPC
                  |
        +---------+---------+
        |                   |
        v                   v
      AZ-1                 AZ-2
        |                   |
   Public Subnet       Public Subnet
        |                   |
       ALB                 ALB
        |                   |
   Private Subnet      Private Subnet
        |                   |
      EC2                  EC2
```

Using multiple Availability Zones helps reduce dependency on a single Availability Zone.

---

# 12. VPC Setup Steps

## Step 1 — Create VPC

Example:

```text
VPC Name: DevOps-VPC
CIDR: 10.0.0.0/16
```

## Step 2 — Create Internet Gateway

Create an Internet Gateway and attach it to the VPC.

## Step 3 — Create Public Subnet

```text
10.0.0.0/24
```

## Step 4 — Create Private Subnet

```text
10.0.1.0/24
```

## Step 5 — Create NAT Gateway

Create the NAT Gateway in the public subnet and associate an Elastic IP.

## Step 6 — Configure Public Route Table

```text
0.0.0.0/0 → Internet Gateway
```

Associate the route table with the public subnet.

## Step 7 — Configure Private Route Table

```text
0.0.0.0/0 → NAT Gateway
```

Associate the route table with the private subnet.

## Step 8 — Create Security Groups

Example:

```text
webserver-sg
appserver-sg
database-sg
```

## Step 9 — Launch EC2

Example:

```text
Public Subnet
    |
    +-- Bastion Host

Private Subnet
    |
    +-- Application Server
    +-- Database
```

---

# 13. Three-Tier Architecture

A common AWS application architecture contains three tiers.

```text
                 Internet
                    |
                    v
             Load Balancer
                    |
          +---------+---------+
          |                   |
          v                   v
      App Server          App Server
       Private              Private
          |                   |
          +---------+---------+
                    |
                    v
                   RDS
                Database
```

### Web Tier

Handles incoming user requests.

### Application Tier

Runs application and business logic.

### Database Tier

Stores application data.

---

# 14. Public vs Private Subnet

| Feature                | Public Subnet           | Private Subnet                         |
| ---------------------- | ----------------------- | -------------------------------------- |
| Route to IGW           | Yes                     | No                                     |
| Direct internet access | Possible                | No direct access                       |
| Typical resources      | ALB, Bastion            | Application, Database                  |
| NAT Gateway            | Can contain NAT Gateway | Uses NAT Gateway for outbound internet |
| Public IP              | May be used             | Usually avoided                        |

---

# 15. Complete VPC Architecture

```text
                         INTERNET
                            |
                            v
                  +-------------------+
                  | Internet Gateway  |
                  +---------+---------+
                            |
                 +----------+----------+
                 |        VPC          |
                 |    10.0.0.0/16      |
                 |                     |
                 |  PUBLIC SUBNET      |
                 |  10.0.0.0/24        |
                 |                     |
                 |  +-------------+    |
                 |  | Bastion/ALB |    |
                 |  +------+------+    |
                 |         |            |
                 |  +------+------+     |
                 |  | NAT Gateway |     |
                 |  +------+------+     |
                 |         |            |
                 |  PRIVATE SUBNET     |
                 |  10.0.1.0/24       |
                 |                     |
                 |  +-------------+    |
                 |  | App Server  |    |
                 |  +------+------+    |
                 |         |            |
                 |         v            |
                 |  +-------------+    |
                 |  |     RDS     |    |
                 |  |  Database   |    |
                 |  +-------------+    |
                 |                     |
                 +---------------------+
```

---

# 16. Important Concepts Learned

During my AWS VPC practice, I learned:

* VPC
* CIDR
* Subnets
* Public Subnet
* Private Subnet
* Internet Gateway
* NAT Gateway
* Route Tables
* Security Groups
* Bastion Host
* VPC Endpoints
* Availability Zones
* EC2 Networking
* Network Segmentation
* Private Server Access
* Three-Tier Architecture

---

# 17. Key Takeaway

The main VPC architecture I practiced:

```text
VPC
 |
 +-- Public Subnet
 |      |
 |      +-- ALB
 |      +-- Bastion Host
 |      +-- NAT Gateway
 |
 +-- Private Subnet
        |
        +-- Application Server
        |
        +-- Database
```

### Traffic Flow

```text
User
 |
 v
Internet
 |
 v
Internet Gateway
 |
 v
Public Resources
 |
 v
Private Application
 |
 v
Database
```

### Simple Summary

```text
VPC
 ↓
Subnet
 ↓
Route Table
 ↓
Internet Gateway / NAT Gateway
 ↓
Security Group
 ↓
EC2 / Application / Database
```

## My Learning Outcome

This hands-on practice helped me understand how AWS networking works and how public and private resources can be separated inside a VPC.

I also learned the foundation required for building AWS architectures using:

**VPC + EC2 + Load Balancer + Auto Scaling + NAT Gateway + RDS + Security Groups**

These concepts will help me build more advanced AWS cloud and DevOps projects.
