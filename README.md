\#  FavMart - AWS Solutions Architect Associate  Project



>A highly available, auto-scaling, multi-tier e-commerce infrastructure built on the AWS Free Tier, designed to demonstrate AWS architecture across networking, compute, data, and security layers.





\## Architecture Diagram



!\[FavMart AWS Architecture](docs/architecture-diagram.png)





\## AWS Services Used



| Service | Purpose |

|---|---|

| \*\*VPC + 6 Subnets\*\* | Custom network with public subnets for the load balancer and private subnets for application (EC2) and data layers (database) |

| \*\*IAM Roles\*\* | Allows EC2 instances to access AWS services securely without storing credentials |

| \*\*EC2 (t2.micro)\*\* | Application servers running in private subnets |

| \*\*Application Load Balancer\*\* | Distributes incoming HTTP traffic across EC2 instances |

| \*\*Auto Scaling Group\*\* | Automatically scales EC2 instances based on CPU utilization |

| \*\*RDS MySQL (db.t3.micro)\*\* | Managed relational database in private subnets |

| \*\*ElastiCache Redis\*\* | Shared session store for the application |

| \*\*EFS\*\* | Shared file system accessible by all EC2 instances |

| \*\*Route 53\*\* | Health checks and DNS monitoring for the application endpoint |

| \*\*Elastic Beanstalk\*\* | Separate managed environment used for staging deployments |

| \*\*NAT Gateway\*\* | Allows private instances to access the internet for updates and package installation |





\## Architecture Decisions



\*\*Why ALB and not NLB?\*\*  

The Application Load Balancer operates at Layer 7 (HTTP) and supports

features like path-based routing and sticky sessions. NLB operates at

Layer 4 (TCP) and is faster but lacks HTTP level capabilities. Since

FavMart serves web traffic, ALB provides the functionality needed for

routing and session handling.



\*\*Why ElastiCache Redis for sessions?\*\*  

Without a shared session store, a user who logs in on one EC2 instance

could appear logged out when the next request is routed to a different

instance. Redis provides a centralized session store so all instances

can access the same session data, allowing the application layer to

remain stateless.



\*\*Why private subnets for EC2?\*\*  

EC2 instances run in private subnets and have no public IP addresses,

which prevents direct access from the internet. All incoming traffic

passes through the Application Load Balancer, providing a controlled

and secure entry point to the application.



\*\*Why EFS and not EBS?\*\*  

EBS volumes attach to a single EC2 instance. In an auto-scaling

environment, new instances would not have access to files stored on

another instance. EFS can be mounted by multiple instances

simultaneously, allowing all application servers to access the same

shared file system.



\*\*Why RDS MySQL and not Aurora?\*\*  

Aurora does not offer a Free Tier eligible instance size. RDS MySQL on

db.t3.micro is a suitable option for development and learning

environments. Aurora would be a potential production upgrade for higher

performance and advanced replication capabilities.





\## Traffic Flow



```

User → Route 53 → Application Load Balancer

&#x20;       ↓

Auto Scaling Group (EC2 instances in private subnets)

&#x20;       ↓

Application interacts with:

• ElastiCache Redis (sessions)

• RDS MySQL (application data)

• EFS (shared files)



Response returns to the user through the ALB

```





\##  Security Architecture

```

Internet

&#x20;  ↓

ALB Security Group (80/443 from internet)

&#x20;  ↓

App Security Group (80 from ALB only)

&#x20;  ↓

RDS Security Group (3306 from app servers)

&#x20;  ↓

Redis Security Group (6379 from app servers)

```



Only the Application Load Balancer is exposed to the internet.

All other services accept traffic only from the specific resources that require access.





\## Problem Encountered



The ALB DNS kept returning \*\*504 Gateway Timeout\*\*, while the EC2 public IP loaded the application correctly.



After several hours checking security groups, health check paths, subnets, and target group ports, I discovered the issue: the \*\*outbound rule on the ALB security group was missing\*\*.



The load balancer could receive requests from users but had no rule allowing it to forward traffic to the EC2 instances.



Adding the outbound rule immediately resolved the issue and the target group became healthy.





\## How to Rebuild



1\. Create a VPC with public, private application, and private data subnets across two Availability Zones.

2\. Configure Internet Gateway, NAT Gateway, route tables, and security groups.

3\. Deploy RDS MySQL in private data subnets.

4\. Deploy ElastiCache Redis in private data subnets.

5\. Create an EFS file system with mount targets in the private application subnets.

6\. Launch an EC2 instance, install Apache, mount EFS, and pull the application from GitHub.

7\. Create a Golden AMI from the configured instance.

8\. Create an Application Load Balancer, Target Group, Launch Template, and Auto Scaling Group.

9\. Configure Route 53 health checks pointing to the ALB.

10\. Deploy a staging environment using Elastic Beanstalk.





\##  Proof of work

Screenshots of every phase are available in docs/screenshots/  from VPC creation to the live application, which loads via the ALB DNS name.



\---

\*Built as part of my AWS Solutions Architect Associate exam preparation\*









