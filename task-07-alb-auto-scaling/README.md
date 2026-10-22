\# Task 7: Application Load Balancer and Auto Scaling



\## Overview



This lab demonstrates how to deploy a highly available web application on AWS using an Application Load Balancer (ALB), an EC2 Auto Scaling Group (ASG), and a CPU-based target-tracking scaling policy.



EC2 instances are launched automatically from a launch template, Nginx is installed through user data, and the ALB routes HTTP requests to healthy targets.



\## Architecture



```text

&#x20;                        Users

&#x20;                          |

&#x20;                          v

&#x20;               Application Load Balancer

&#x20;                   task7-nginx-alb

&#x20;                          |

&#x20;                   HTTP Listener :80

&#x20;                          |

&#x20;                          v

&#x20;                  Target Group :80

&#x20;                  task7-nginx-tg

&#x20;                    /          \\

&#x20;                   v            v

&#x20;              EC2 Instance  EC2 Instance

&#x20;              Amazon Linux  Amazon Linux

&#x20;                 + Nginx       + Nginx

&#x20;                    \\          /

&#x20;                     Auto Scaling

&#x20;                        Group

&#x20;                 task7-nginx-asg

&#x20;                          |

&#x20;                 CPU Target Tracking

&#x20;                      Target: 50%

```



\## AWS Resources



| Resource | Configuration |

|---|---|

| AWS Region | ap-south-1 (Mumbai) |

| VPC | `vpc-01e04109fea01ca2f` |

| Application Load Balancer | `task7-nginx-alb` |

| Target Group | `task7-nginx-tg` |

| Auto Scaling Group | `task7-nginx-asg` |

| Launch Template | `task7-nginx-template` |

| Launch Template ID | `lt-0dc54d191a9dd384a` |

| Launch Template Version | 1 |

| Instance Type | `t3.micro` |

| Minimum Capacity | 2 |

| Desired Capacity | 2 |

| Maximum Capacity | 4 |

| Scaling Policy | `task7-cpu-target-tracking` |

| CPU Target | 50% |

| Application Protocol | HTTP |

| Health Check Path | `/` |



\### Security Groups



\- ALB security group: `sg-0f7f78709ea2a3ffa`

\- EC2 security group: `sg-0a40c9c7532d43be2`



The ALB security group allows inbound HTTP traffic on port 80. The EC2 security group allows inbound HTTP traffic from the ALB security group.



\## Implementation Steps



\### 1. Create Security Groups



Created separate security groups for the ALB and EC2 instances to control inbound traffic.



\### 2. Create the Launch Template



Created `task7-nginx-template` using Amazon Linux 2023 and instance type `t3.micro`.



User data installs and starts Nginx:



```bash

\#!/bin/bash

dnf install -y nginx

systemctl enable --now nginx

echo "<h1>Task 7 - Auto Scaling</h1><p>Instance: $(hostname)</p>" > /usr/share/nginx/html/index.html

```



\### 3. Create the Target Group



Created `task7-nginx-tg` with these settings:



\- Protocol: HTTP

\- Port: 80

\- Target type: Instance

\- Health check path: `/`

\- Expected health check response: HTTP 200



\### 4. Create the Application Load Balancer



Created an internet-facing ALB named `task7-nginx-alb` across two Availability Zones.



Configured an HTTP listener on port 80 to forward requests to `task7-nginx-tg`.



\### 5. Create the Auto Scaling Group



Created `task7-nginx-asg` using the launch template and configured it across two subnets in different Availability Zones.



Capacity settings:



\- Minimum: 2 instances

\- Desired: 2 instances

\- Maximum: 4 instances



Configured Elastic Load Balancing health checks with a 180-second health check grace period.



\### 6. Configure CPU-Based Scaling



Created the target-tracking policy `task7-cpu-target-tracking`.



\- Metric: `ASGAverageCPUUtilization`

\- Target value: 50%



The Auto Scaling service manages capacity adjustments based on the configured target and observed metrics, within the minimum and maximum capacity limits.



\## Validation and Results



The following checks were completed successfully:



\- The ALB returned `HTTP/1.1 200 OK`.

\- The Nginx welcome page displayed `Task 7 - Auto Scaling`.

\- Both EC2 instances registered with the target group and reached `healthy` status.

\- The ASG reported two instances in `InService` state.

\- The ASG capacity configuration was verified as minimum 2, desired 2, and maximum 4.

\- The CPU target-tracking scaling policy was created successfully.

\- Six HTTP requests were sent through the ALB. Responses showed hostnames from both EC2 instances.



\*\*Note:\*\* The CPU target-tracking policy was configured and verified, but an actual scale-out or scale-in event under load was not tested in this lab.



\## Screenshots



The `screenshots/` directory contains the following evidence:



1\. `01-alb-details.png`

2\. `02-target-group-healthy.png`

3\. `03-asg-capacity.png`

4\. `04-cpu-scaling-policy.png`

5\. `05-website-http-200.png`

6\. `06-load-balancing-responses.png`



\## Key Learnings



\- How an Application Load Balancer distributes HTTP traffic across registered targets.

\- How target groups and health checks determine whether instances can receive traffic.

\- How launch templates standardize EC2 instance configuration.

\- How Auto Scaling Groups maintain desired capacity across Availability Zones.

\- How CPU target tracking can adjust the number of instances based on demand.

\- Why separate ALB and EC2 security groups improve network access control.

\- How to validate an AWS deployment using the console, AWS CLI, and HTTP requests.



\## Cost and Cleanup



The ALB and EC2 instances may incur charges while running. Review the resources when the lab is complete and clean them up if they are no longer needed.



For safe cleanup, first remove the Auto Scaling Group and its instances, then remove the ALB and listener, target group, launch template, and dedicated security groups when no longer in use.



\## Repository Structure



```text

task-07-alb-auto-scaling/

├── README.md

├── CHEATSHEET.md

└── screenshots/

&#x20;   ├── 01-alb-details.png

&#x20;   ├── 02-target-group-healthy.png

&#x20;   ├── 03-asg-capacity.png

&#x20;   ├── 04-cpu-scaling-policy.png

&#x20;   ├── 05-website-http-200.png

&#x20;   └── 06-load-balancing-responses.png

```



