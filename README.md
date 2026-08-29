\# AWS Highly Available Web Application



\## Overview



This project demonstrates how to build a highly available web application architecture on AWS using multiple Availability Zones.



The architecture separates the application into public and private network layers and uses an Application Load Balancer to distribute traffic across multiple EC2 application servers.



\## Architecture



The application uses:



\* Amazon VPC

\* Public and private subnets

\* Internet Gateway

\* NAT Gateways

\* Network ACLs

\* Security Groups

\* Application Load Balancer

\* EC2 application servers

\* Target Group

\* Route 53 Private Hosted Zone

\* IAM Role for EC2

\* Amazon Linux

\* Nginx



\## Network Design



VPC:



```text

10.0.0.0/16

```



\### Public Subnets



```text

10.0.1.0/24   us-east-1a

10.0.3.0/24   us-east-1b

```



Used for internet-facing infrastructure such as the Application Load Balancer and NAT Gateways.



\### Private Application Subnets



```text

10.0.11.0/24  us-east-1a

10.0.13.0/24  us-east-1b

```



Used for EC2 application servers.



\### Private Database Subnets



```text

10.0.21.0/24  us-east-1a

10.0.22.0/24  us-east-1b

```



Reserved for the database layer.



\## Application Servers



Two EC2 instances were deployed:



```text

EC2-1

Availability Zone: us-east-1a

Private subnet: 10.0.11.0/24



EC2-2

Availability Zone: us-east-1b

Private subnet: 10.0.13.0/24

```



Nginx was installed on both servers.



Each server returns a different response so that load balancing can be verified.



\## Load Balancing



An internet-facing Application Load Balancer was created:



```text

aws-learning-alb

```



Target group:



```text

aws-learning-app-tg

```



Protocol:



```text

HTTP : 80

```



Both EC2 instances are registered as targets.



Final target health:



```text

2 Healthy

0 Unhealthy

```



\## DNS



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



\## Load Balancing Test



The application was tested repeatedly using:



```bash

for i in {1..20}; do

&#x20; curl -s http://app.aws-learning.internal | grep -E "Hello from|EC2-"

&#x20; echo "----------------"

done

```



The responses successfully came from both application servers.



Example:



```text

Hello from APP SERVER 1

EC2-1 | Private Subnet | us-east-1a

```



and:



```text

Hello from APP SERVER 2

EC2-2 | Private Subnet | us-east-1b

```



This confirms that the Application Load Balancer is distributing requests between the two healthy targets.



\## Architecture Flow



```text

&#x20;                   Internet

&#x20;                      |

&#x20;                      v

&#x20;             +----------------+

&#x20;             | Internet       |

&#x20;             | Gateway        |

&#x20;             +-------+--------+

&#x20;                     |

&#x20;                     v

&#x20;             +----------------+

&#x20;             | Application    |

&#x20;             | Load Balancer  |

&#x20;             +-------+--------+

&#x20;                     |

&#x20;             +-------+-------+

&#x20;             |               |

&#x20;             v               v

&#x20;      +-------------+  +-------------+

&#x20;      | EC2 App 1   |  | EC2 App 2   |

&#x20;      | us-east-1a  |  | us-east-1b  |

&#x20;      | Private     |  | Private     |

&#x20;      | Nginx :80   |  | Nginx :80   |

&#x20;      +-------------+  +-------------+



&#x20;             Route 53

&#x20;                |

&#x20;                v

&#x20;     app.aws-learning.internal

&#x20;                |

&#x20;                v

&#x20;         Application ALB

```



\## Security Model



The application security group allows HTTP traffic on port 80 from the ALB security group.



The EC2 instances are not intended to be directly exposed to the internet.



The architecture therefore follows the principle:



```text

Internet

&#x20;  |

&#x20;  v

ALB

&#x20;  |

&#x20;  v

Private EC2

```



rather than:



```text

Internet

&#x20;  |

&#x20;  v

EC2

```



\## Key AWS Concepts Demonstrated



This lab demonstrates practical understanding of:



1\. VPC architecture

2\. CIDR addressing

3\. Public vs private subnets

4\. Availability Zones

5\. Internet Gateway

6\. NAT Gateway

7\. Route tables

8\. Security Groups

9\. Network ACLs

10\. IAM roles

11\. EC2

12\. Nginx

13\. Application Load Balancer

14\. Target Groups

15\. Health Checks

16\. Route 53 Private Hosted Zones

17\. DNS resolution inside a VPC

18\. High availability

19\. Load balancing

20\. Basic troubleshooting



\## Validation



The final architecture successfully demonstrated:



\* Both EC2 instances running

\* Nginx running on both instances

\* Both target instances healthy

\* ALB returning HTTP 200

\* Private DNS resolving inside the VPC

\* Traffic reaching private EC2 instances through the ALB

\* Requests being distributed between both application servers



\## Lessons Learned



The lab demonstrated that troubleshooting AWS networking requires checking each layer independently:



```text

DNS

&#x20;↓

Load Balancer

&#x20;↓

Target Group

&#x20;↓

Security Group

&#x20;↓

Network ACL

&#x20;↓

Route Table

&#x20;↓

EC2

&#x20;↓

Application

```



Testing each layer independently makes it much easier to identify where connectivity problems occur.



\## Next Steps



Future improvements to this architecture could include:



\* HTTPS with ACM

\* Auto Scaling Group

\* CloudWatch monitoring

\* Bastion-free administration using Systems Manager

\* RDS Multi-AZ

\* WAF

\* CloudFront

\* Infrastructure as Code with Terraform

\* CI/CD deployment pipeline

\* Automated health checks and recovery



