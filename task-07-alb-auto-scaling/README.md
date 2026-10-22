# Task 7: Application Load Balancer and Auto Scaling

## Overview

This lab demonstrates how to deploy a web application on AWS using an Application Load Balancer (ALB), an EC2 Auto Scaling Group (ASG), and a CPU-based target-tracking scaling policy.

EC2 instances are launched using a launch template, Nginx is installed through user data, and the ALB routes HTTP requests to healthy targets.

## Architecture

```text
                         Users
                           |
                           v
              Application Load Balancer
                   task7-nginx-alb
                           |
                    HTTP Listener :80
                           |
                           v
                   Target Group :80
                   task7-nginx-tg
                     /          \
                    v            v
               EC2 Instance  EC2 Instance
               Amazon Linux  Amazon Linux
                  + Nginx       + Nginx
                     \           /
                      Auto Scaling
                          Group
                    task7-nginx-asg
                           |
                  CPU Target Tracking
                      Target: 50%
```

## AWS Resources

| Resource | Configuration |
|---|---|
| AWS Region | `ap-south-1` (Mumbai) |
| VPC | `vpc-01e04109fea01ca2f` |
| Application Load Balancer | `task7-nginx-alb` |
| Target Group | `task7-nginx-tg` |
| Auto Scaling Group | `task7-nginx-asg` |
| Launch Template | `task7-nginx-template` |
| Launch Template ID | `lt-0dc54d191a9dd384a` |
| Launch Template Version | `1` |
| Instance Type | `t3.micro` |
| Minimum Capacity | `2` |
| Desired Capacity | `2` |
| Maximum Capacity | `4` |
| Scaling Policy | `task7-cpu-target-tracking` |
| CPU Target | `50%` |
| Application Protocol | HTTP |
| Health Check Path | `/` |

### Security Groups

| Security Group | ID | Purpose |
|---|---|---|
| ALB security group | `sg-0f7f78709ea2a3ffa` | Allows inbound HTTP traffic on port 80 |
| EC2 security group | `sg-0a40c9c7532d43be2` | Allows inbound HTTP from the ALB security group |

The EC2 security group restricts web traffic to the ALB rather than allowing direct HTTP access from arbitrary internet sources.

## Implementation Steps

### 1. Create Security Groups

Created separate security groups for the ALB and EC2 instances.

- The ALB security group allows inbound HTTP on port 80.
- The EC2 security group allows inbound HTTP on port 80 from the ALB security group.

### 2. Create the Launch Template

Created `task7-nginx-template` using Amazon Linux 2023 and instance type `t3.micro`.

The user-data script installs Nginx, enables the service, and creates a custom web page displaying the instance hostname.

```bash
#!/bin/bash
dnf install -y nginx
systemctl enable --now nginx
echo "<h1>Task 7 - Auto Scaling</h1><p>Instance: $(hostname)</p>" > /usr/share/nginx/html/index.html
```

The hostname helps identify which EC2 instance responds to an HTTP request.

### 3. Create the Target Group

Created `task7-nginx-tg` with the following configuration:

- Protocol: HTTP
- Port: `80`
- Target type: `instance`
- Health check path: `/`
- Expected health check response: HTTP `200`

The target group checks target health and makes health information available to the load balancer.

### 4. Create the Application Load Balancer

Created the internet-facing ALB `task7-nginx-alb` across two Availability Zones.

Configured an HTTP listener on port 80 to forward requests to `task7-nginx-tg`.

### 5. Create the Auto Scaling Group

Created `task7-nginx-asg` using the launch template and configured it across two subnets in different Availability Zones.

Capacity settings:

- Minimum: `2`
- Desired: `2`
- Maximum: `4`

Configured Elastic Load Balancing health checks with a 180-second health check grace period.

The ASG maintains the desired capacity and can replace instances that fail applicable health checks.

### 6. Configure CPU-Based Scaling

Created the target-tracking policy `task7-cpu-target-tracking`.

- Metric: `ASGAverageCPUUtilization`
- Target value: `50%`

The Auto Scaling service adjusts capacity based on the configured target and observed metrics, subject to the group's minimum and maximum capacity.

A target value of 50% does not mean that an instance will be launched immediately whenever CPU crosses 50%. Scaling behavior depends on the metric history, scaling policy behavior, and available capacity.

## Validation and Results

The following checks were completed successfully:

- The ALB returned `HTTP/1.1 200 OK`.
- The Nginx page displayed `Task 7 - Auto Scaling`.
- Both EC2 instances registered with the target group and reached `healthy` status.
- The ASG reported two instances in `InService` state.
- The configured minimum, desired, and maximum capacities were verified as `2`, `2`, and `4`.
- The CPU target-tracking scaling policy was created successfully.
- Six HTTP requests were sent through the ALB. Responses showed hostnames from both EC2 instances.

**Testing limitation:** The CPU target-tracking policy was configured and verified, but an actual scale-out or scale-in event under load was not tested. The lab therefore demonstrates load balancing and scaling-policy configuration, not a verified scaling event.

## Screenshots

The `screenshots/` directory contains the following evidence:

1. `01-alb-details.png`
2. `02-target-group-healthy.png`
3. `03-asg-capacity.png`
4. `04-cpu-scaling-policy.png`
5. `05-website-http-200.png`
6. `06-load-balancing-responses.png`

## Key Learnings

- How an Application Load Balancer distributes HTTP traffic across registered targets.
- How target groups and health checks determine target eligibility.
- How launch templates standardize EC2 instance configuration.
- How Auto Scaling Groups maintain desired capacity across Availability Zones.
- How CPU target tracking can adjust capacity based on demand.
- Why separate ALB and EC2 security groups improve network access control.
- How to validate AWS deployments using the console, AWS CLI, and HTTP requests.
- Why creating a scaling policy is different from demonstrating an actual scaling event.

## Cost and Cleanup

The ALB, EC2 instances, and other associated resources may incur charges while running.

When the lab is complete, review the resources and remove those no longer required. Before deleting anything, confirm that it is dedicated to this lab.

Suggested cleanup order:

1. Delete the Auto Scaling Group and terminate its managed instances, if they are no longer needed.
2. Delete the ALB after confirming it is no longer needed.
3. Delete the listener if it still exists and is not removed with the ALB.
4. Delete the target group after it is no longer in use.
5. Delete the launch template.
6. Delete the dedicated security groups after checking their dependencies.

## Repository Structure

```text
task-07-alb-auto-scaling/
├── README.md
├── CHEATSHEET.md
└── screenshots/
    ├── 01-alb-details.png
    ├── 02-target-group-healthy.png
    ├── 03-asg-capacity.png
    ├── 04-cpu-scaling-policy.png
    ├── 05-website-http-200.png
    └── 06-load-balancing-responses.png
```

## Conclusion

This lab demonstrated an internet-facing ALB distributing requests across two healthy Nginx instances managed by an Auto Scaling Group. It also established a CPU target-tracking policy with a target of 50% and a capacity range of two to four instances.

The load-balancing behavior and policy configuration were validated. Actual scale-out and scale-in behavior remains untested.
