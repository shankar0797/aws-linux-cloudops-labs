# ☁️ AWS Linux & CloudOps Labs

**Hands-on AWS infrastructure | Linux Administration | Networking | Security | Monitoring**

A portfolio of eight practical labs focused on building, configuring, validating, and troubleshooting AWS infrastructure.

[Projects](#-projects) · [Technical Skills](#-technical-skills) · [Repository Structure](#-repository-structure) · [Learning Outcomes](#-learning-outcomes)

---

## 👋 About This Portfolio

This repository documents hands-on exercises relevant to **AWS Cloud Operations, Cloud Support Engineering, L2 Cloud Engineering, and Linux Administration**.

Each project includes implementation documentation, key configuration details, validation steps, troubleshooting guidance, and screenshots where available.

## 🧰 Technical Skills

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Administration-black?logo=linux&logoColor=white)
![Networking](https://img.shields.io/badge/Networking-VPC%20%7C%20ALB-blue)
![Security](https://img.shields.io/badge/Security-IAM%20%7C%20Encryption-green)
![Monitoring](https://img.shields.io/badge/Monitoring-CloudWatch-purple)

- **Compute:** Amazon EC2
- **Linux:** Users, groups, sudo, SSH authentication, Nginx
- **Networking:** VPC, subnets, route tables, security groups, Network ACLs
- **Storage:** EBS volumes, encryption, Amazon S3
- **Security:** IAM roles, policies, least privilege, private resource access
- **Availability:** Application Load Balancer, target groups, health checks
- **Scaling:** Launch templates, Auto Scaling Groups, target tracking
- **Monitoring:** CloudWatch metrics, dashboards, alarms, logs, SNS
- **Operations:** Systems Manager Run Command, Parameter Store, troubleshooting

## 🚀 Projects

| # | Project | What It Covers | Documentation |
|---|---|---|---|
| 01 | Linux Users & SSH Authentication | Linux accounts, sudo privileges, SSH keys, file permissions | [README](./task-01-linux-users-ssh/README.md) · [Cheatsheet](./task-01-linux-users-ssh/CHEATSHEET.md) |
| 02 | Nginx on EC2 | Linux web server setup, service management, HTTP validation | [README](./task-02-nginx-ec2/README.md) |
| 03 | Encrypted EBS Volume | Block storage, encryption, volume operations | [README](./task-03-encrypted-ebs-volume/README.md) |
| 04 | Nginx Application Load Balancer | Listeners, target groups, health checks | [README](./task-04-nginx-alb/README.md) |
| 05 | Multi-Tier VPC Architecture | Public/private subnet design, routing, network segmentation | [README](./task-05-multi-tier-vpc/README.md) |
| 06 | IAM & Private S3 Access | IAM permissions, S3 access control, least privilege | [README](./task-06-iam-private-s3-access/README.md) |
| 07 | ALB & Auto Scaling | Launch templates, Auto Scaling, health checks, target tracking | [README](./task-07-alb-auto-scaling/README.md) |
| 08 | CloudWatch Monitoring & SNS | Custom metrics, dashboards, alarms, logs, notifications | [README](./task-08-cloudwatch-monitoring/README.md) |

### What Each Project Demonstrates

**01 — Linux Users & SSH Authentication**

Created Linux accounts, configured controlled sudo access, set up SSH public-key authentication, and validated permissions and login behavior.

**02 — Nginx on EC2**

Practiced deploying a Linux web server, managing the Nginx service, configuring network access, and verifying HTTP availability.

**03 — Encrypted EBS Volume**

Explored EBS storage, encryption, volume operations, and Linux storage administration.

**04 — Nginx Application Load Balancer**

Configured load-balancing components and explored how listeners, target groups, and health checks determine target availability.

**05 — Multi-Tier VPC Architecture**

Worked with VPC networking, subnet segmentation, routing, and network security controls.

**06 — IAM & Private S3 Access**

Practiced IAM permissions and controlled access to private S3 resources.

**07 — ALB & Auto Scaling**

Configured an Application Load Balancer, target group, launch template, Auto Scaling Group, and CPU target-tracking policy. Verified HTTP responses and target health.

**08 — CloudWatch Monitoring & SNS**

Configured CloudWatch dashboards and alarms, installed the CloudWatch Agent through Systems Manager, stored its configuration in Parameter Store, and verified custom memory/disk metrics and Nginx logs.

---

## 📂 Repository Structure

```text
aws-linux-cloudops-labs/
├── README.md
├── MASTER-CHEATSHEET.md
├── task-01-linux-users-ssh/
│   ├── README.md
│   ├── CHEATSHEET.md
│   └── screenshots/
├── task-02-nginx-ec2/
├── task-03-encrypted-ebs-volume/
├── task-04-nginx-alb/
├── task-05-multi-tier-vpc/
├── task-06-iam-private-s3-access/
├── task-07-alb-auto-scaling/
└── task-08-cloudwatch-monitoring/
```

Each task folder contains its own project documentation and may include a cheatsheet, screenshots, or configuration files.

## 🧪 Validation & Troubleshooting

The labs emphasize validating actual behavior, not just creating AWS resources.

Examples include:

- Checking Linux authentication, sudo permissions, and SSH configuration.
- Verifying Nginx service health and HTTP responses.
- Inspecting load balancer target health and application connectivity.
- Investigating IAM access-denied errors and network restrictions.
- Reviewing CloudWatch metrics, alarms, and application logs.
- Identifying root causes and documenting corrective actions.

Implementation results and untested scenarios are distinguished in the task-specific documentation.

## 🔐 Security & Cost Awareness

- Follow least-privilege access principles.
- Never commit private keys, credentials, or access tokens.
- Restrict network access to required sources and ports.
- Use encryption where appropriate.
- Review CloudWatch log retention and monitoring charges.
- Remove unused lab resources to control AWS costs.

## 📚 Learning Outcomes

- Practical exposure to AWS infrastructure and Linux operations.
- Understanding of network traffic flow and access controls.
- Familiarity with load balancing and Auto Scaling fundamentals.
- Experience with CloudWatch monitoring and centralized logging.
- A structured approach to operational validation and troubleshooting.
- Project documentation designed for technical revision and interview discussions.

## 📝 Quick Revision

Use [`MASTER-CHEATSHEET.md`](./MASTER-CHEATSHEET.md) for consolidated revision across the portfolio when available.

Each project's own `CHEATSHEET.md` should contain its detailed commands, configuration notes, and task-specific troubleshooting steps.

---

**Repository:** [aws-linux-cloudops-labs](https://github.com/shankar0797/aws-linux-cloudops-labs)
