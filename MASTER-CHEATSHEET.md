# AWS Linux & CloudOps Labs — Master Cheatsheet

A consolidated revision guide for nine hands-on labs covering Linux administration, AWS infrastructure, networking, security, load balancing, auto scaling, monitoring, Docker deployment, and troubleshooting.

## Table of Contents

1. Linux Users & SSH
2. Nginx on EC2
3. Encrypted EBS Volumes
4. Application Load Balancer
5. Multi-Tier VPC
6. IAM & Private S3
7. ALB & Auto Scaling
8. CloudWatch & SNS
9. Docker Web Application on EC2
10. Common CloudOps Troubleshooting Workflow
11. Interview Quick Revision
12. Security and Cleanup
13. Revision Strategy

---

## Task 1 — Linux Users & SSH Authentication

### Key concepts

- Linux users and groups
- `sudo` privileges
- SSH public-key authentication
- `authorized_keys` permissions
- Secure file permissions

### Important commands

```bash
# Create users
sudo useradd -m devuser
sudo useradd -m opsuser

# Create sudoers configuration
sudo visudo -f /etc/sudoers.d/opsuser

# Validate sudoers syntax
sudo visudo -c

# Check user information
id devuser
id opsuser

# Check sudo privileges
sudo -l -U opsuser

# Check SSH permissions for the current user
ls -ld ~/.ssh
ls -l ~/.ssh/authorized_keys

# Check SSH service
sudo systemctl status sshd
```

### Permissions to understand

- `.ssh` directory: commonly `700`
- `authorized_keys`: commonly `600`
- Private SSH key: protect it; commonly `600` on Linux

Use the correct home directory for the target user when checking SSH files. Never upload private keys to GitHub.

### Troubleshooting

| Symptom | Checks |
|---|---|
| SSH permission denied | Username, public key, `authorized_keys`, permissions, SSH logs |
| User cannot use sudo | Sudoers configuration, syntax, permitted commands |
| SSH service unavailable | Service status, instance reachability, security group, network path |

---

## Task 2 — Nginx on EC2

### Key concepts

- EC2 connectivity
- Linux package management
- Nginx service management
- HTTP port 80
- Security groups

### Important commands

```bash
# Install Nginx on Amazon Linux 2023
sudo dnf install nginx -y

# Start and enable Nginx
sudo systemctl enable --now nginx

# Check service status
sudo systemctl status nginx

# Check listening ports
sudo ss -lntp

# Test locally
curl -I http://localhost

# View service logs
sudo journalctl -u nginx --no-pager -n 50

# View Nginx logs
sudo tail -n 50 /var/log/nginx/access.log
sudo tail -n 50 /var/log/nginx/error.log
```

### Troubleshooting

1. Confirm the EC2 instance is running and status checks pass.
2. Confirm Nginx is installed and running.
3. Check the security group permits inbound HTTP from the intended source.
4. Check network routes, public IP/DNS, and NACLs when relevant.
5. Inspect Nginx logs and the local HTTP response.

---

## Task 3 — Encrypted EBS Volume

### Key concepts

- EBS volumes and EC2 attachment
- Encryption at rest
- Linux block devices
- Filesystems and mount points
- Data protection

### Important commands

```bash
# List block devices and mount points
lsblk

# Show filesystem information
df -hT

# Show mounted filesystems
findmnt

# Show disk usage
df -h

# Review kernel messages for storage issues
sudo dmesg | tail -n 50
```

### Troubleshooting

- Confirm the volume is attached to the intended instance and Availability Zone.
- Check whether the device appears in `lsblk`.
- Verify the filesystem and mount configuration before mounting.
- Confirm encryption in the EBS volume details in the AWS Console.
- Avoid formatting or overwriting a volume containing data you need.

**Important:** A newly attached volume may need partitioning and formatting before first use. Never run formatting commands unless the device is confirmed to be the intended empty volume.

---

## Task 4 — Nginx Application Load Balancer

### Key concepts

- ALB listeners and rules
- Target groups
- Health-check paths and ports
- Target health
- Security group traffic flows

### Troubleshooting checklist

1. Check the ALB state and listener configuration.
2. Check target registration and target health reasons.
3. Confirm the application listens on the configured target port.
4. Verify the health-check protocol and path return the expected response.
5. Check the ALB security group permits client traffic.
6. Check the target security group permits traffic from the ALB security group.
7. Inspect application logs and network routes.

### Common symptoms

| Symptom | Possible area to investigate |
|---|---|
| Target unhealthy | Application service, health-check path/port, security groups |
| HTTP 503 | No usable healthy targets, listener/rule configuration |
| Connection timeout | Routing, NACLs, security groups, application listener |
| HTTP 403 | Application response, listener rules, WAF or access controls if configured |

Do not assume every HTTP 503 is caused by unhealthy targets. Inspect the ALB and application configuration.

---

## Task 5 — Multi-Tier VPC Architecture

### Key concepts

- VPC CIDR blocks
- Public and private subnets
- Route tables
- Internet Gateway
- NAT Gateway where required
- Security groups and Network ACLs

### Traffic-flow revision

**Public subnet:** A route to an Internet Gateway can provide an internet route, but a resource also needs suitable addressing and security rules.

**Private subnet:** Typically has no direct inbound internet route. Outbound internet access may be provided by a NAT Gateway if the architecture requires it.

**Security group:** A stateful virtual firewall associated with resources or network interfaces.

**Network ACL:** A stateless subnet-level traffic filter. Return traffic may require explicit rules.

### Troubleshooting

- Verify subnet CIDR and Availability Zone.
- Verify the route table associated with the subnet.
- Check Internet Gateway or NAT Gateway routes where applicable.
- Check security group inbound and outbound rules.
- Check NACL rules for both request and return traffic.
- Verify DNS resolution and the destination service's listening port.

---

## Task 6 — IAM & Private S3 Access

### Key concepts

- IAM users, roles, and policies
- Identity-based and resource-based policies
- S3 bucket policies
- Least privilege
- Explicit deny and access evaluation

### Troubleshooting `AccessDenied`

1. Identify the caller and the AWS action being attempted.
2. Verify the caller's identity-based permissions.
3. Review the S3 bucket policy.
4. Check for explicit denies, permissions boundaries, session policies, and applicable organization policies.
5. Verify the bucket/object ARN and resource scope.
6. Check encryption permissions, such as KMS permissions, if the object uses SSE-KMS.
7. Confirm the request uses the intended role or credentials.

### Security principles

- Grant only the actions and resources required.
- Keep buckets private unless public access is explicitly intended.
- Avoid embedding long-lived access keys in scripts.
- Use IAM roles for supported AWS workloads.

---

## Task 7 — ALB & Auto Scaling

### Lab configuration

| Setting | Value |
|---|---|
| Region | `ap-south-1` |
| ALB | `task7-nginx-alb` |
| Target group | `task7-nginx-tg` |
| Auto Scaling Group | `task7-nginx-asg` |
| Launch template | `task7-nginx-template` |
| Desired capacity | 2 |
| Maximum capacity | 4 |
| CPU target tracking | 50% |

### Key concepts

- Launch templates define instance launch settings.
- Auto Scaling Groups maintain desired capacity within configured limits.
- ALB health checks assess whether targets can receive traffic.
- Target tracking adjusts capacity based on a target metric.

### Useful commands

```bash
# Inspect target health
aws elbv2 describe-target-health \
  --target-group-arn <target-group-arn>

# Review scaling activities
aws autoscaling describe-scaling-activities \
  --auto-scaling-group-name task7-nginx-asg

# Review Auto Scaling Group configuration
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names task7-nginx-asg
```

Replace placeholders with the correct resource identifiers.

### Troubleshooting

1. Check Auto Scaling Group activity history.
2. Review launch template AMI, instance type, IAM profile, and security groups.
3. Check instance status checks and bootstrap/user data logs.
4. Check target group health and health-check settings.
5. Review CloudWatch metrics and scaling policy configuration.
6. Confirm maximum capacity and subnet capacity allow additional instances.

**Validation note:** HTTP 200 responses and healthy targets were verified in the lab. Claim a successful scale-out only when the corresponding scaling activity has actually been observed.

---

## Task 8 — CloudWatch Monitoring & SNS

### Lab configuration

| Setting | Value |
|---|---|
| Region | `ap-south-1` |
| Dashboard | `task8-operations-dashboard` |
| Custom namespace | `Task8/EC2` |
| Parameter Store parameter | `AmazonCloudWatch-agent-task8-config` |
| Access log group | `/task8/nginx/access` |
| Error log group | `/task8/nginx/error` |

### Important metrics

- `CPUUtilization` — EC2 CPU usage.
- `StatusCheckFailed` — instance/system status-check failures.
- `mem_used_percent` — memory utilization collected by the CloudWatch Agent.
- `disk_used_percent` — disk utilization collected by the CloudWatch Agent.

EC2 does not publish guest operating system memory usage by default. The CloudWatch Agent is used to collect this information.

### Systems Manager documents

- `AWS-ConfigureAWSPackage` — used to install the CloudWatch Agent package.
- `AmazonCloudWatch-ManageAgent` — used to configure and manage the agent.

### Useful commands

```bash
# Check CloudWatch alarms
aws cloudwatch describe-alarms

# Check SSM managed instances
aws ssm describe-instance-information

# Check CloudWatch Agent status on Linux
sudo systemctl status amazon-cloudwatch-agent

# Review CloudWatch Agent logs
sudo tail -n 100 /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

### Troubleshooting

**Missing custom metrics**

1. Check the `Task8/EC2` namespace.
2. Verify the metric name and instance dimension.
3. Review the Parameter Store configuration.
4. Check agent status, logs, IAM permissions, and collection interval.

**Missing Nginx logs**

1. Confirm the configured log file paths exist.
2. Confirm Nginx writes to the expected files.
3. Check CloudWatch log groups and streams.
4. Inspect the agent's status and error logs.

**SNS notification not received**

1. Verify the alarm action references the correct topic.
2. Confirm the SNS subscription is confirmed.
3. Verify the alarm enters the expected state.
4. Test notification delivery and confirm receipt.

Creating an alarm and connecting an SNS topic does not, by itself, prove end-to-end email delivery works.

---

## Task 9 — Docker Web Application on EC2

### Key concepts

- Docker images and containers
- Dockerfile instructions
- Nginx static web serving
- Container lifecycle management
- Host-to-container port mapping
- HTTP response validation
- EC2 security groups and public connectivity

### Lab configuration

| Setting | Value |
|---|---|
| Operating system | Amazon Linux 2023 |
| Base image | `nginx:alpine` |
| Docker image | `cloudops-webapp:1.0` |
| Container name | `cloudops-webapp` |
| Container port | `80` |
| EC2 host port | `8080` |
| Protocol | HTTP |

### Dockerfile

```dockerfile
FROM nginx:alpine
COPY app/index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

- `FROM nginx:alpine` selects the base image.
- `COPY` places the custom HTML page in Nginx's default web root.
- `EXPOSE 80` documents the container port; it does not publish the port on the host by itself.

### Build and run

```bash
# Build the custom image
sudo docker build -t cloudops-webapp:1.0 .

# List local images
sudo docker images

# Run the container
sudo docker run -d \
  --name cloudops-webapp \
  --restart unless-stopped \
  -p 8080:80 \
  cloudops-webapp:1.0

# Check running containers
sudo docker ps

# Check all containers
sudo docker ps -a

# Review logs
sudo docker logs cloudops-webapp
```

### Understand port mapping

```text
-p 8080:80
   |    |
   |    +-- Container port 80 (Nginx)
   +------- EC2 host port 8080
```

Request path:

```text
Browser
  -> EC2 public IP:8080
  -> Security group and network path
  -> Docker host port 8080
  -> Container port 80
  -> Nginx
  -> Custom index.html
```

### Validation

```bash
# Test from the EC2 host
curl -i http://localhost:8080

# Show response headers
curl -I http://localhost:8080

# Inspect published ports
sudo docker port cloudops-webapp

# Review recent logs
sudo docker logs --tail 100 cloudops-webapp
```

**Observed lab result:** The local HTTP request returned `HTTP/1.1 200 OK` and the expected HTML. The public browser test also displayed the application successfully during the lab.

Public endpoint format:

```text
http://PUBLIC-IP:8080
```

Replace `PUBLIC-IP` with the instance's actual public IPv4 address.

### Troubleshooting

| Symptom | Checks |
|---|---|
| Container stopped | `sudo docker ps -a`; `sudo docker logs cloudops-webapp` |
| Local HTTP fails | `curl -v http://localhost:8080`; check container status and published ports |
| Port conflict | `sudo ss -lntp`; inspect existing Docker port mappings |
| Old HTML appears | Rebuild the image and recreate the container |
| Public endpoint times out | Check public IP, security group, subnet route, and public network path |
| Local curl works but public access fails | Focus on external connectivity and AWS network rules |

### Security reminders

- Restrict inbound TCP port 8080 to trusted source IP addresses where possible.
- Avoid exposing demonstration services to the entire internet unnecessarily.
- HTTP is unencrypted; use an appropriate HTTPS configuration for production.
- Never store credentials or private keys in the image or repository.
- Stop or remove unused lab resources when appropriate.

### Evidence

- `task-09-docker-web-application/screenshots/01-docker-image.png`
- `task-09-docker-web-application/screenshots/02-container-http-200.png`
- `task-09-docker-web-application/screenshots/03-public-web-application.png`

Detailed documentation: [Task 9 README](./task-09-docker-web-application/README.md)

---

## Common CloudOps Troubleshooting Workflow

Use this sequence when investigating a production issue:

1. **Understand impact:** Identify affected users, services, scope, and start time.
2. **Check monitoring:** Review CloudWatch metrics, alarms, logs, and recent changes.
3. **Validate resource health:** Check EC2 status checks, application services, target health, and dependencies.
4. **Check connectivity:** Review DNS, routes, security groups, NACLs, and listening ports.
5. **Collect evidence:** Capture relevant errors, timestamps, command outputs, and log entries.
6. **Identify the root cause:** Distinguish symptoms from the underlying failure.
7. **Apply an approved fix:** Follow change and incident procedures.
8. **Validate recovery:** Confirm service health and user-facing behavior.
9. **Document and prevent recurrence:** Record root cause, resolution, and preventive actions.

### General Linux commands

```bash
# CPU and processes
top

# Memory
free -h

# Disk capacity and inodes
df -h
df -i

# Block devices
lsblk

# Service status
sudo systemctl status <service>

# Recent service logs
sudo journalctl -u <service> --since "30 minutes ago"

# Test TCP connectivity
nc -vz <host> <port>

# DNS resolution
dig <domain>
```

### Useful AWS CLI commands

```bash
# Confirm AWS identity
aws sts get-caller-identity

# Check EC2 instance status
aws ec2 describe-instance-status \
  --instance-ids <instance-id>

# Inspect target health
aws elbv2 describe-target-health \
  --target-group-arn <target-group-arn>

# Review CloudWatch alarms
aws cloudwatch describe-alarms

# Review recent CloudTrail events
aws cloudtrail lookup-events

# List S3 objects
aws s3 ls s3://<bucket-name>/
```

Replace placeholders with actual resource identifiers. Confirm the AWS account, profile, and region before making changes.

---

## Interview Quick Revision

**Q: What is the difference between a security group and a NACL?**

Security groups are stateful and associated with resources or network interfaces. NACLs are stateless and operate at the subnet boundary.

**Q: What causes an ALB target to become unhealthy?**

The health check may fail because the application is stopped, the port or path is incorrect, or network rules prevent the ALB from reaching the target.

**Q: What should you check if SSH fails?**

Verify instance state, network path, security group, username, SSH key, `authorized_keys`, file permissions, and SSH service.

**Q: Why use an Auto Scaling Group?**

It maintains desired capacity and can add or replace instances according to its configuration and scaling policies.

**Q: Why might CloudWatch memory metrics be missing?**

The CloudWatch Agent may be missing, stopped, misconfigured, or unable to publish the metric.

**Q: What is least privilege?**

Granting only the permissions needed for a specific task, resource, or workload.

**Q: What is the difference between a metric and an alarm?**

A metric is measured data. An alarm evaluates a metric against configured conditions and can change state or invoke actions.

**Q: What is the purpose of an incident RCA?**

To identify the underlying cause of an incident and define corrective and preventive actions.

**Q: What does Docker port mapping `-p 8080:80` mean?**

It maps host port 8080 to container port 80.

**Q: Does Dockerfile `EXPOSE 80` publish the container port?**

No. It documents the intended container port. Publishing is configured separately, such as with `-p 8080:80`.

**Q: How do you troubleshoot a Docker web application that is unreachable publicly?**

Check container status and logs, test locally with `curl`, verify port mapping, and then inspect the EC2 public IP, security group, and network path.

---

## Security and Cleanup

- Never commit private keys, AWS credentials, tokens, or sensitive configuration.
- Check IAM permissions before granting broader access.
- Confirm the intended device before formatting or modifying storage.
- Review log retention and CloudWatch costs.
- Remove temporary resources after lab validation.
- Restrict public network access to the minimum required.
- Keep documentation aligned with actual test results.

## Revision Strategy

1. Read the individual task README to understand the implementation.
2. Use that task's `CHEATSHEET.md` for detailed commands and troubleshooting.
3. Use this master cheatsheet for quick revision across all nine projects.
4. Practice explaining one issue, the investigation sequence, evidence, root cause, resolution, and validation for each major service.
5. Distinguish completed hands-on validation from scenarios that have not been tested.
6. Follow approved change procedures when applying these commands in production.
