# AWS Three-Tier Architecture

A hands-on AWS three-tier architecture project built using Amazon VPC, public and private subnets, an Application Load Balancer, EC2 application servers, a Bastion Host, and Amazon RDS for MySQL.

This project was built to understand how AWS networking, security, load balancing, compute, database connectivity, high availability, and troubleshooting work together in a real-world style architecture.

---

## 📌 Project Overview

The architecture follows a three-tier design:

- **Presentation Tier** – Internet-facing Application Load Balancer
- **Application Tier** – Two private EC2 application servers
- **Database Tier** – Private Amazon RDS MySQL database
- **Administration Layer** – Bastion Host for controlled SSH access

The application servers are distributed across two Availability Zones to demonstrate application-level high availability and failover.

---

## 🏗️ Architecture

```text
                                  INTERNET
                                      |
                                      |
                              Internet Gateway
                                      |
                                      |
                         +------------------------+
                         |    Public Subnets      |
                         |                        |
                         |  Application Load      |
                         |       Balancer         |
                         +-----------+------------+
                                     |
                                  HTTP : 80
                                     |
                    +----------------+----------------+
                    |                                 |
                    v                                 v
          +------------------+              +------------------+
          | App Subnet 1     |              | App Subnet 2     |
          | AZ: 2a           |              | AZ: 2b           |
          |                  |              |                  |
          | App Server 1     |              | App Server 2     |
          +---------+--------+              +---------+--------+
                    |                                 |
                    +----------------+----------------+
                                     |
                                  MySQL : 3306
                                     |
                                     v
                         +------------------------+
                         |    Database Subnets    |
                         |                        |
                         |     Amazon RDS         |
                         |       MySQL             |
                         +------------------------+


                    ADMINISTRATION ACCESS

                         Administrator
                               |
                            SSH : 22
                               |
                               v
                       +---------------+

                       | Bastion Host  |
                       | Public Subnet |
                       +-------+-------+
                               |
                            SSH : 22
                               |
                               v
                       Private App Servers

🌐 VPC Design

The project uses a dedicated VPC.

Component	Configuration
VPC Name	Three-Tier-VPC
CIDR	10.11.0.0/16
Region	ap-southeast-2 (Sydney)
Subnet Layout
Tier	Subnet	CIDR	Availability Zone
Public	public-subnet-1	10.11.1.0/24	ap-southeast-2a
Public	public-subnet-2	10.11.2.0/24	ap-southeast-2b
Application	app-subnet-1	10.11.11.0/24	ap-southeast-2a
Application	app-subnet-2	10.11.12.0/24	ap-southeast-2b
Database	database-subnet-1	10.11.21.0/24	ap-southeast-2a
Database	database-subnet-2	10.11.22.0/24	ap-southeast-2b

The six subnets are divided into public, application, and database tiers and distributed across two Availability Zones.

🛣️ Routing Design

Three route tables were created to separate routing between the different tiers.

Public Route Table
10.11.0.0/16 → local
0.0.0.0/0    → Internet Gateway

The public route table is associated with both public subnets.

This allows resources in the public tier to communicate with the internet when the resource has appropriate public addressing and security-group permissions.

Application Route Table
10.11.0.0/16 → local

The application route table is associated with both application subnets.

The application servers do not have a direct internet route.

Database Route Table
10.11.0.0/16 → local

The database route table is associated with both database subnets.

The database tier therefore has no direct internet route.

🔐 Security Group Architecture

Security groups were configured using a layered approach.

ALB Security Group
Inbound:
HTTP : 80
Source: 0.0.0.0/0

The Application Load Balancer accepts HTTP traffic from the internet.

Application Server Security Group
Inbound:
HTTP : 80
Source: alb-public-sg

SSH : 22
Source: bastion-sg

The application servers accept web traffic only from the ALB and administrative SSH traffic from the Bastion Host.

Database Security Group
Inbound:
MySQL : 3306
Source: app-server-sg

The database accepts MySQL traffic only from the application tier.

Bastion Security Group
Inbound:
SSH : 22
Source: Administrator's IP

SSH access to the Bastion Host is restricted to the administrator's public IP.

Traffic Flow
Internet
   |
   | HTTP : 80
   v
ALB
   |
   | HTTP : 80
   v
Application Servers
   |
   | MySQL : 3306
   v
RDS MySQL

Administrative traffic follows a separate path:

Administrator
      |
      | SSH : 22
      v
Bastion Host
      |
      | SSH : 22
      v
Private Application Servers
⚖️ Application Load Balancer

An internet-facing Application Load Balancer named:

three-tier-alb

was deployed across two public subnets in different Availability Zones.

Listener
Protocol: HTTP
Port: 80

The listener forwards requests to:

app-target-group
Target Group

The target group contains:

App Server 1
App Server 2

Configuration:

Protocol: HTTP
Port: 80
Health Check Path: /

The ALB uses health checks to determine whether registered targets are available to receive traffic.

🖥️ Application Tier

Two Amazon Linux EC2 instances were deployed in private subnets.

App Server 1
Subnet: app-subnet-1
Availability Zone: ap-southeast-2a
Private IP: 10.11.11.41
App Server 2
Subnet: app-subnet-2
Availability Zone: ap-southeast-2b
Private IP: 10.11.12.244

Both servers were deployed without public IP addresses.

This keeps the application tier private and prevents direct internet access to the EC2 instances.

🌐 Demonstration Application

A lightweight Python HTTP server was used for the application demonstration.

The application was intentionally kept simple because the primary focus of the project was the AWS infrastructure and networking architecture.

App Server 1 displays:

Three-Tier AWS Application
Served by App Server 1

App Server 2 displays:

Three-Tier AWS Application
Served by App Server 2

The application server was configured as a Linux systemd service.

This provides:

Automatic startup after EC2 reboot
Automatic restart if the process fails
Service management through systemctl
Production Consideration

The Python built-in HTTP server was used only as a lightweight demonstration application.

For a production environment, it would be replaced with an appropriate web/application stack such as Nginx with a production application server or the organization's actual application platform.

🗄️ Database Tier

The database tier uses Amazon RDS for MySQL.

Configuration
Setting	Value
Engine	MySQL
Instance Class	db.t3.micro
Storage	20 GiB gp3
Port	3306
Public Access	No
VPC	Three-Tier-VPC

The RDS instance uses a dedicated DB subnet group containing:

database-subnet-1
database-subnet-2

The database is not publicly accessible.

Only the application server security group is allowed to connect to the database on port 3306.

🔑 Bastion Host

A Bastion Host was deployed in a public subnet to provide controlled administrative access to the private application servers.

The Bastion Host has a public IP, while the application servers remain private.

Access Flow
Administrator
     |
     | SSH : 22
     v
Bastion Host
     |
     | SSH : 22
     v
Private App Servers

SSH access to the Bastion Host is restricted to the administrator's IP.

SSH agent forwarding was used so that the private key for the application servers did not need to be copied onto the Bastion Host.

🔄 High Availability and Failover

The application tier uses two EC2 instances deployed across different Availability Zones.

Normal State
                    ALB
                   /   \
                  /     \
                 v       v
        App Server 1   App Server 2
           Healthy        Healthy
Failover Test

One application server was intentionally stopped during testing.

The target group detected that the stopped instance was no longer available for traffic.

                    ALB
                     |
                     v
              App Server 1
                 Healthy

              App Server 2
             Stopped/Unused

The ALB continued serving the application through the remaining healthy application server.

The ALB DNS endpoint successfully returned:

Three-Tier AWS Application
Served by App Server 1

while App Server 2 was stopped.

This demonstrated application-level failover through the Application Load Balancer.

🔌 Application-to-Database Connectivity

Connectivity between the application tier and RDS was tested over TCP port 3306.

The TCP connection to the RDS endpoint was successfully established.

This verified the network path between:

Private App Subnet
       |
       | TCP : 3306
       v
Private RDS Database

The test validated network-level connectivity, including:

VPC routing
Subnet connectivity
Security group rules
RDS network accessibility

The test verified TCP connectivity and did not represent a successful MySQL authentication or SQL query execution.

🧪 Validation and Testing

The following tests were performed during the project.

1. VPC Validation

Verified the VPC CIDR and subnet structure.

VPC: 10.11.0.0/16
2. Multi-AZ Validation

Verified that the public, application, and database subnets were distributed across:

ap-southeast-2a
ap-southeast-2b
3. ALB Validation

Accessed the ALB DNS name from the internet and successfully received the application response.

4. Target Health Validation

Both application servers successfully registered with the target group and reached a healthy state.

5. Failover Validation

Stopped one application server and verified that the ALB continued serving traffic through the remaining available application server.

6. EC2 Reboot Validation

Rebooted the application servers and verified that the Python web service automatically started through systemd.

7. RDS Connectivity Validation

Successfully established TCP connectivity from the application tier to RDS on port 3306.

🛠️ Challenges and Troubleshooting

This project also involved troubleshooting several real-world infrastructure issues.

Private Application Servers

The application servers were deployed without public IP addresses.

Administration was performed through the Bastion Host instead of exposing the application servers directly to the internet.

SSH Agent Forwarding

SSH agent forwarding was configured to allow access from the Bastion Host to private EC2 instances without copying private keys onto the Bastion.

ALB Health Checks

Target group health status was monitored while starting, stopping, and rebooting EC2 instances.

This helped validate how the ALB handles available and unavailable targets.

Application Startup

Initially, the Python HTTP server was started manually.

A systemd service was later configured to provide automatic startup and process recovery.

Database Security

The database security group was intentionally restricted to the application server security group.

The Bastion Host was not granted direct database access because it was not required for the architecture.

This follows the principle of least privilege.

📸 Project Evidence

The project includes AWS console evidence covering:

VPC configuration
Six-subnet architecture
Multi-AZ deployment
Route tables
Security groups
Application Load Balancer
Target group health
EC2 infrastructure
RDS configuration
RDS connectivity
Application response
Failover testing
Bastion Host
systemd service configuration
📚 Key Learning Outcomes

Through this project, I gained hands-on experience with:

AWS VPC architecture
CIDR planning
Public and private subnet design
Availability Zones
Route tables
Internet Gateway
Security group design
Application Load Balancer
Target groups
Health checks
EC2 deployment
Amazon RDS
Bastion Host architecture
SSH agent forwarding
Linux systemd
Application-to-database networking
High availability concepts
Failover testing
AWS troubleshooting
Infrastructure security
AWS cost awareness
💰 Cost Considerations

The project was built with cost awareness in mind.

Resources were stopped when they were not required for testing.

However, stopping EC2 and RDS instances does not necessarily eliminate all AWS charges.

Depending on usage and configuration, charges may still apply for resources such as:

EBS storage
Application Load Balancer
Public IPv4 addresses
RDS storage and backups
Other associated AWS resources

Unused billable resources should therefore be stopped or removed when the project is not being used.

🚀 Future Improvements

The current project focuses on learning and demonstrating AWS infrastructure concepts.

Potential improvements include:

Auto Scaling Groups
HTTPS using AWS Certificate Manager
Route 53
CloudFront
AWS WAF
CloudWatch monitoring
Centralized logging
AWS Systems Manager Session Manager
Terraform
CloudFormation
CI/CD pipeline
AWS Secrets Manager
Multi-AZ RDS deployment
Automated application deployment
Containerization using ECS or EKS
Infrastructure monitoring and alerting
