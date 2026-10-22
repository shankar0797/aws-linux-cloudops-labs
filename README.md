# AWS Linux & CloudOps Labs

A hands-on portfolio demonstrating AWS infrastructure, Linux administration, cloud networking, security, load balancing, auto scaling, monitoring, Docker deployment, and operational troubleshooting.

## About This Repository

This repository contains practical cloud and Linux labs designed to strengthen AWS CloudOps, Cloud Support, Infrastructure Operations, and troubleshooting skills.

Each project includes implementation documentation and may include a task-specific cheatsheet, configuration files, and screenshots demonstrating the results.

## Technology Stack

- **Cloud Platform:** Amazon Web Services (AWS)
- **Operating Systems:** Amazon Linux 2023, Linux
- **Compute:** Amazon EC2
- **Storage:** Amazon EBS, Amazon S3
- **Networking:** VPC, subnets, route tables, Internet Gateway, NAT Gateway, Security Groups, Network ACLs
- **Load Balancing and Scaling:** Application Load Balancer, Auto Scaling Groups
- **Identity and Security:** IAM, SSH authentication, EBS encryption, private S3 access
- **Monitoring:** Amazon CloudWatch, CloudWatch Agent, Amazon SNS
- **Containers:** Docker, Nginx
- **Validation Tools:** AWS CLI, Linux commands, curl
- **Documentation:** Markdown, screenshots, troubleshooting guides

## Hands-On Projects

| Task | Project | Key Skills | Documentation |
|---|---|---|---|
| 1 | Linux Users & SSH Authentication | Linux users, sudo permissions, SSH keys, secure file permissions | [Task 1](./task-01-linux-users-ssh/README.md) |
| 2 | Nginx on EC2 | Package installation, systemd, HTTP validation, security groups | [Task 2](./task-02-nginx-ec2/README.md) |
| 3 | Encrypted EBS Volume | EBS encryption, block devices, XFS, mounting, persistent storage | [Task 3](./task-03-encrypted-ebs-volume/README.md) |
| 4 | Nginx Application Load Balancer | ALB listeners, target groups, health checks, connectivity troubleshooting | [Task 4](./task-04-nginx-alb/README.md) |
| 5 | Multi-Tier VPC Architecture | Public and private subnets, routing, Internet Gateway, NAT, network security | [Task 5](./task-05-multi-tier-vpc/README.md) |
| 6 | IAM & Private S3 Access | IAM policies, bucket policies, least privilege, access-denied troubleshooting | [Task 6](./task-06-iam-private-s3-access/README.md) |
| 7 | ALB & Auto Scaling | Launch templates, target groups, scaling policies, health checks | [Task 7](./task-07-alb-auto-scaling/README.md) |
| 8 | CloudWatch Monitoring & SNS | Dashboards, alarms, CloudWatch Agent, memory and disk metrics, logs, notifications | [Task 8](./task-08-cloudwatch-monitoring/README.md) |
| 9 | Docker Web Application on EC2 | Docker image builds, container lifecycle, Nginx, port mapping, public HTTP access | [Task 9](./task-09-docker-web-application/README.md) |

## Project Highlights

### Task 1 — Linux Users & SSH Authentication

Created Linux users, configured sudo access, and implemented SSH public-key authentication with appropriate file permissions.

### Task 2 — Nginx on EC2

Installed and configured Nginx on an EC2 instance, managed the service with systemd, and verified HTTP access.

### Task 3 — Encrypted EBS Volume

Created and attached an encrypted EBS data volume, configured a filesystem, and practiced Linux storage and mount management.

### Task 4 — Nginx Application Load Balancer

Configured an Application Load Balancer and target group, then reviewed health checks, listener configuration, and application connectivity.

### Task 5 — Multi-Tier VPC Architecture

Practiced public and private subnet design, routing, Internet Gateway and NAT Gateway concepts, and network access controls.

### Task 6 — IAM & Private S3 Access

Worked with IAM permissions and private S3 access, focusing on least privilege and troubleshooting `AccessDenied` errors.

### Task 7 — ALB & Auto Scaling

Configured an ALB, target group, launch template, and Auto Scaling Group. Practiced target health checks and scaling configuration.

### Task 8 — CloudWatch Monitoring & SNS

Configured CloudWatch monitoring components, dashboards, alarms, custom operating-system metrics, log collection, and SNS notifications.

### Task 9 — Docker Web Application on AWS EC2

Built a custom Docker image and deployed a static web application inside an Nginx container on Amazon EC2.

**Implementation:**
- Created a custom HTML/CSS web page.
- Built the `cloudops-webapp:1.0` Docker image using `nginx:alpine`.
- Started the `cloudops-webapp` container with restart configuration.
- Mapped EC2 host port `8080` to container port `80`.
- Verified the application using `curl` and received an HTTP `200 OK` response.
- Opened the public HTTP endpoint in a browser and confirmed the application displayed successfully.
- Captured screenshots of the Docker image, container/HTTP response, and public web page.

**Task 9 resources:**

- [Project README](./task-09-docker-web-application/README.md)
- [Docker Cheatsheet](./task-09-docker-web-application/CHEATSHEET.md)
- [Dockerfile](./task-09-docker-web-application/Dockerfile)
- [Application HTML](./task-09-docker-web-application/app/index.html)
- [Evidence screenshots](./task-09-docker-web-application/screenshots/)

## Repository Structure

```text
aws-linux-cloudops-labs/
├── README.md
├── MASTER-CHEATSHEET.md
├── task-01-linux-users-ssh/
├── task-02-nginx-ec2/
├── task-03-encrypted-ebs-volume/
├── task-04-nginx-alb/
├── task-05-multi-tier-vpc/
├── task-06-iam-private-s3-access/
├── task-07-alb-auto-scaling/
├── task-08-cloudwatch-monitoring/
└── task-09-docker-web-application/
    ├── app/
    │   └── index.html
    ├── Dockerfile
    ├── README.md
    ├── CHEATSHEET.md
    └── screenshots/
```

Each task directory contains its own project documentation and may include a cheatsheet, screenshots, or configuration files.

## Validation & Troubleshooting

The labs emphasize verifying actual behavior, not just creating AWS resources.

Examples include:

- Checking Linux authentication, sudo permissions, and SSH configuration.
- Verifying Nginx service health and HTTP responses.
- Inspecting EBS devices, encryption, filesystems, and mount configuration.
- Investigating ALB target health and application connectivity.
- Troubleshooting VPC routes, security groups, and NACLs.
- Diagnosing IAM access-denied errors.
- Reviewing Auto Scaling activity and CloudWatch metrics.
- Validating Docker containers, port mappings, logs, and HTTP responses.
- Documenting root causes, resolutions, and validation results.

Implementation results and untested scenarios are distinguished in the task-specific documentation.

## Security & Cost Awareness

- Follow least-privilege access principles.
- Never commit private keys, credentials, or access tokens.
- Restrict network access to required sources and ports.
- Use encryption where appropriate.
- Use HTTPS for production web applications.
- Review CloudWatch log retention and monitoring charges.
- Remove unused lab resources to control AWS costs.
- Validate impact before changing or deleting infrastructure.

## Learning Outcomes

- Practical exposure to AWS infrastructure and Linux operations.
- Understanding of network traffic flows and access controls.
- Familiarity with load balancing and Auto Scaling fundamentals.
- Experience with CloudWatch monitoring and centralized logging.
- Hands-on Docker image creation and container deployment.
- A structured approach to operational validation and troubleshooting.
- Documentation suitable for technical revision and interview preparation.

## Quick Revision

Use the [Master Cheatsheet](./MASTER-CHEATSHEET.md) for consolidated revision across all nine projects.

Each project's own `CHEATSHEET.md` contains detailed commands, configuration notes, and task-specific troubleshooting steps.

---

**Repository:** [aws-linux-cloudops-labs](https://github.com/shankar0797/aws-linux-cloudops-labs)
