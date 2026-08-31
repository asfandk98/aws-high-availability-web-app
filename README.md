# AWS Highly Available Web Application

A hands-on AWS networking and high-availability project demonstrating how to deploy a web application across multiple Availability Zones using an Application Load Balancer, private EC2 instances, VPC networking, security controls, and Route 53 private DNS.

---

## Overview

This project demonstrates how to build a **highly available web application architecture on AWS** using multiple Availability Zones.

The architecture separates the application into **public and private network layers** and uses an **Application Load Balancer (ALB)** to distribute incoming HTTP traffic across multiple EC2 application servers.

The project focuses on practical AWS networking concepts, including:

* Amazon VPC
* CIDR addressing
* Public and private subnets
* Availability Zones
* Internet Gateway
* NAT Gateways
* Route tables
* Security Groups
* Network ACLs
* Application Load Balancer
* Target Groups
* EC2
* Nginx
* IAM roles
* Route 53 Private Hosted Zones
* DNS resolution
* Health checks
* High availability
* Load balancing
* Network troubleshooting

---

## Architecture

![AWS Highly Available Web Application Architecture](./architecture/aws-architecture.png)

The architecture consists of a VPC spanning two Availability Zones.

Internet-facing infrastructure is placed in public subnets, while the application servers are deployed in private subnets.

The main components are:

* **Amazon VPC**
* **Public and private subnets**
* **Internet Gateway**
* **NAT Gateways**
* **Network ACLs**
* **Security Groups**
* **Application Load Balancer**
* **EC2 application servers**
* **Target Group**
* **Route 53 Private Hosted Zone**
* **IAM Role for EC2**
* **Amazon Linux**
* **Nginx**

---

## Network Design

### VPC

```text
10.0.0.0/16
```

The VPC provides the overall network boundary for the application.

The network is divided into public, private application, and private database subnets across two Availability Zones.

### Public Subnets

```text
10.0.1.0/24   us-east-1a
10.0.3.0/24   us-east-1b
```

The public subnets are used for internet-facing infrastructure such as:

* Application Load Balancer
* NAT Gateways

These subnets have routes to the Internet Gateway.

### Private Application Subnets

```text
10.0.11.0/24  us-east-1a
10.0.13.0/24  us-east-1b
```

The private application subnets contain the EC2 application servers.

The servers do not require public IP addresses and are not intended to be directly accessible from the internet.

### Private Database Subnets

```text
10.0.21.0/24  us-east-1a
10.0.22.0/24  us-east-1b
```

These subnets are reserved for a future database layer such as Amazon RDS Multi-AZ.

---

## Application Servers

Two EC2 application servers were deployed across separate Availability Zones.

### EC2-1

```text
Availability Zone: us-east-1a
Private subnet:    10.0.11.0/24
```

### EC2-2

```text
Availability Zone: us-east-1b
Private subnet:    10.0.13.0/24
```

Nginx was installed and configured on both servers.

Each server returns a different response so that load balancing can be verified.

Example response from the first server:

```text
Hello from APP SERVER 1

EC2-1 | Private Subnet | us-east-1a
```

Example response from the second server:

```text
Hello from APP SERVER 2

EC2-2 | Private Subnet | us-east-1b
```

---

## Load Balancing

An internet-facing **Application Load Balancer** was created to distribute HTTP requests between the two EC2 instances.

### Load Balancer

```text
aws-learning-alb
```

### Target Group

```text
aws-learning-app-tg
```

### Protocol

```text
HTTP : 80
```

Both EC2 instances were registered with the target group.

### Final Target Health

```text
2 Healthy
0 Unhealthy
```

The Application Load Balancer health checks confirmed that both application servers were available.

---

## DNS

A Route 53 private hosted zone was created:

```text
aws-learning.internal
```

The following DNS record was configured:

```text
app.aws-learning.internal
```

The record points to the Application Load Balancer.

DNS resolution from inside the VPC successfully returned the ALB addresses.

This allows clients inside the VPC to access the application using:

```text
http://app.aws-learning.internal
```

rather than directly accessing the EC2 instances.

---

## Load Balancing Test

The application was tested repeatedly to verify that requests were being distributed between both EC2 instances.

Example test:

```bash
for i in {1..20}; do
    curl -s http://app.aws-learning.internal | grep -E "Hello from|EC2-"
    echo "----------------"
done
```

The responses successfully came from both application servers.

Example:

```text
Hello from APP SERVER 1
EC2-1 | Private Subnet | us-east-1a
----------------
Hello from APP SERVER 2
EC2-2 | Private Subnet | us-east-1b
----------------
Hello from APP SERVER 1
EC2-1 | Private Subnet | us-east-1a
----------------
```

This confirms that the Application Load Balancer is distributing requests between the two healthy targets.

---

## Architecture Flow

```text
                         INTERNET
                            |
                            v
                  +-------------------+
                  |  Internet Gateway |
                  +---------+---------+
                            |
                            v
                  +-------------------+
                  | Application Load  |
                  |    Balancer       |
                  +---------+---------+
                            |
                     +------+------+
                     |             |
                     v             v
              +------------+  +------------+
              |   EC2 App 1|  |   EC2 App 2|
              | us-east-1a |  | us-east-1b |
              |  Private   |  |  Private   |
              |  Nginx :80 |  | Nginx :80 |
              +------------+  +------------+
                     |             |
                     +------+------+
                            |
                         VPC Network


                    Route 53 Private DNS
                            |
                            v
                 app.aws-learning.internal
                            |
                            v
                    Application ALB
```

---

## Security Model

The application security group allows HTTP traffic on port `80` **from the ALB security group**.

The EC2 instances are not intended to be directly exposed to the internet.

The desired traffic flow is:

```text
Internet
   |
   v
 ALB
   |
   v
Private EC2
```

Rather than exposing the EC2 instances directly:

```text
Internet
   |
   v
 EC2
```

This design reduces the public attack surface and separates the internet-facing load-balancing layer from the application layer.

---

## Network Security

The architecture uses multiple AWS networking and security controls.

### Security Groups

Security Groups control traffic at the instance and load-balancer level.

The application servers allow HTTP traffic from the ALB security group rather than allowing unrestricted internet traffic.

### Network ACLs

Network ACLs provide an additional subnet-level traffic control layer.

They can be used to control inbound and outbound traffic at the subnet boundary.

### Private Subnets

The application servers are deployed in private subnets and do not need to be directly reachable from the public internet.

### NAT Gateways

NAT Gateways provide outbound internet connectivity for resources in private subnets when required, without assigning public IP addresses to the application servers.

---

## Route Tables

The VPC uses separate routing behavior for public and private subnets.

### Public Subnet Routing

Public subnets have a route to the Internet Gateway:

```text
0.0.0.0/0
     |
     v
Internet Gateway
```

### Private Subnet Routing

Private application subnets can use a NAT Gateway for outbound internet access:

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

The NAT Gateway does not provide unsolicited inbound access to the private EC2 instances.

---

## IAM

An IAM role was associated with the EC2 instances to provide AWS permissions without requiring long-term access keys to be stored on the servers.

Using IAM roles is preferable to embedding AWS credentials directly into application servers.

---

## High Availability

The application servers are deployed across two Availability Zones:

```text
us-east-1a
    |
    |---- EC2 App 1
    |
    +---- Private Application Subnet


us-east-1b
    |
    |---- EC2 App 2
    |
    +---- Private Application Subnet
```

The Application Load Balancer distributes requests between the healthy instances.

If one application server becomes unhealthy, the load balancer health check can detect the failure and stop routing traffic to that target.

This provides a foundation for a highly available application architecture.

---

## Key AWS Concepts Demonstrated

This project demonstrates practical understanding of:

1. VPC architecture
2. CIDR addressing
3. Public vs private subnets
4. Availability Zones
5. Internet Gateway
6. NAT Gateway
7. Route tables
8. Security Groups
9. Network ACLs
10. IAM roles
11. EC2
12. Amazon Linux
13. Nginx
14. Application Load Balancer
15. Target Groups
16. Health Checks
17. Route 53 Private Hosted Zones
18. DNS resolution inside a VPC
19. High availability
20. Load balancing
21. Network troubleshooting

---

## Validation

The final architecture successfully demonstrated:

* Both EC2 instances running
* Nginx running on both instances
* Both target instances healthy
* ALB returning HTTP 200 responses
* Private DNS resolving inside the VPC
* Traffic reaching private EC2 instances through the ALB
* Requests being distributed between both application servers
* Application servers running across separate Availability Zones

---

## Troubleshooting Approach

The lab demonstrated that troubleshooting AWS networking requires checking each layer independently.

The troubleshooting process followed this general flow:

```text
DNS
 |
 v
Load Balancer
 |
 v
Target Group
 |
 v
Security Group
 |
 v
Network ACL
 |
 v
Route Table
 |
 v
EC2
 |
 v
Application
```

Testing each layer independently makes it much easier to identify where connectivity problems occur.

For example:

### DNS Problem

Check:

```text
Route 53
Private Hosted Zone
DNS Record
VPC Association
DNS Resolution
```

### ALB Problem

Check:

```text
ALB Status
Listener
Listener Port
Target Group
Health Checks
```

### EC2 Connectivity Problem

Check:

```text
Security Group
Network ACL
Route Table
NAT Gateway
Internet Gateway
```

### Application Problem

Check:

```text
EC2 Instance
Nginx Service
Port 80
Local Curl Test
Application Response
```

---

## Project Structure

```text
aws-high-availability-web-app/
│
├── architecture/
│   └── aws-architecture.png
│
└── README.md
```

The architecture diagram is stored in the `architecture` directory and displayed directly in this README.

---

## Future Improvements

The current architecture provides a strong foundation, but several improvements could be added.

### HTTPS

Add HTTPS using:

* AWS Certificate Manager
* HTTPS listener on the Application Load Balancer
* HTTP to HTTPS redirection

### Auto Scaling

Replace the manually created EC2 instances with an **Auto Scaling Group** so that instances can automatically launch or terminate based on demand and health.

### CloudWatch

Add monitoring and logging using Amazon CloudWatch for:

* EC2 metrics
* ALB metrics
* Application logs
* CPU utilization
* Health checks
* Alarms

### Systems Manager

Use AWS Systems Manager Session Manager for server administration instead of requiring a public SSH/bastion host.

### RDS Multi-AZ

Deploy a managed database using Amazon RDS with Multi-AZ capabilities.

The reserved database subnets can then be used for the database layer.

### AWS WAF

Add AWS WAF in front of the application to provide protection against common web-based attacks.

### CloudFront

Add Amazon CloudFront for:

* Content delivery
* Caching
* Reduced latency
* Additional edge-level protection

### Infrastructure as Code

Convert the architecture into Infrastructure as Code using:

* Terraform
* AWS CloudFormation

### CI/CD

Create an automated deployment pipeline using services such as:

* GitHub
* GitHub Actions
* AWS CodePipeline
* AWS CodeBuild

### Automated Recovery

Implement automated health checks and recovery mechanisms to improve application resilience.

---

## What This Project Demonstrates

This project demonstrates more than simply launching EC2 instances.

It shows how multiple AWS services work together to create a secure and highly available application environment:

```text
                    INTERNET
                       |
                       v
              +----------------+
              | Internet       |
              | Gateway        |
              +-------+--------+
                      |
                      v
              +----------------+
              | Application    |
              | Load Balancer  |
              +-------+--------+
                      |
              +-------+-------+
              |               |
              v               v
       +-------------+  +-------------+
       | EC2 App 1   |  | EC2 App 2   |
       | us-east-1a  |  | us-east-1b  |
       | Private     |  | Private     |
       +-------------+  +-------------+
              |               |
              +-------+-------+
                      |
                      v
                 Application
```

The architecture demonstrates the fundamental principle of placing the **load-balancing layer in front of private application servers**, while distributing workloads across multiple Availability Zones.

---

## Conclusion

This AWS lab successfully demonstrated the deployment and validation of a highly available web application architecture.

The final environment includes:

* A custom VPC
* Public and private subnet architecture
* Multiple Availability Zones
* Internet Gateway
* NAT Gateways
* Security Groups
* Network ACLs
* Application Load Balancer
* Target Group
* Two private EC2 application servers
* Nginx
* Route 53 Private Hosted Zone
* IAM role-based access
* ALB health checks
* Load balancing validation

The project also provided practical experience troubleshooting connectivity across the AWS networking stack.

The architecture can now be extended toward a more production-oriented environment using **Auto Scaling, HTTPS, CloudWatch, RDS Multi-AZ, WAF, CloudFront, Infrastructure as Code, and CI/CD**.
