# Task 05 — AWS Multi-Tier VPC Architecture

## 1. Overview

Designed and deployed a multi-tier AWS network architecture in the Mumbai region (`ap-south-1`). The architecture separates public-facing load balancing, private application compute, and private database resources across two Availability Zones.

The Application Load Balancer (ALB) receives HTTP requests and forwards them to an Nginx web server running on a private Amazon EC2 instance. A private Amazon RDS for PostgreSQL database represents the database tier.

### Objectives

- Design a VPC with public, application-private, and database-private subnets.
- Configure route tables and Internet Gateway routing.
- Deploy an internet-facing Application Load Balancer.
- Run Nginx on a private EC2 instance without a public IP.
- Restrict communication between tiers using security groups.
- Deploy a private RDS PostgreSQL database.
- Validate connectivity, target health, and database accessibility.
- Remove temporary resources after completing setup.

## 2. Architecture

```text
                       Internet Users
                             |
                             | HTTP :80
                             v
                  Internet-facing ALB
                    task5-app-alb
                             |
                             v
                   Target Group :80
                    task5-app-tg
                             |
                             v
                Private Application EC2
                   Amazon Linux 2023
                         Nginx
                             |
                             | PostgreSQL :5432
                             v
                  Private RDS PostgreSQL
                    task5-test-db
```

The ALB resides in public subnets. The application server and database are placed in private subnets. Security groups restrict traffic between the tiers.

**Important:** This lab uses two Availability Zones for subnet distribution, but it has one registered application EC2 target and one RDS instance. The design does not, by itself, demonstrate full application or database high availability.

## 3. AWS Resources

| Resource | Name / Configuration |
|---|---|
| Region | `ap-south-1` |
| VPC | `multitier-vpc` |
| VPC CIDR | `10.0.0.0/16` |
| Public route table | `task5-public-rt` |
| App private route table | `task5-app-private-rt` |
| DB private route table | `task5-db-private-rt` |
| Application Load Balancer | `task5-app-alb` |
| Target group | `task5-app-tg` |
| Application EC2 | `i-028e01bd3aa36ae30` |
| Application security group | `task5-app-sg` |
| ALB security group | `task5-alb-sg` |
| Database security group | `task5-db-sg` |
| RDS database | `task5-test-db` |
| RDS engine | PostgreSQL |
| RDS port | `5432` |
| RDS database name | `task5db` |
| DB subnet group | `task5-db-subnet-group` |

## 4. Subnet Design

The VPC uses six `/20` subnets across two Availability Zones.

| Tier | Subnet CIDR | Availability Zone |
|---|---|---|
| Public A | `10.0.0.0/20` | `ap-south-1a` |
| Public B | `10.0.16.0/20` | `ap-south-1b` |
| App Private A | `10.0.32.0/20` | `ap-south-1a` |
| App Private B | `10.0.48.0/20` | `ap-south-1b` |
| DB Private A | `10.0.64.0/20` | `ap-south-1a` |
| DB Private B | `10.0.80.0/20` | `ap-south-1b` |

The subnets separate the public entry point, application compute, and database resources into distinct network tiers.

## 5. Routing Configuration

### Public route table

Route table: `task5-public-rt`

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | Internet Gateway |

The default route allows resources with suitable public addressing and security rules to communicate through the Internet Gateway.

### Application private route table

Route table: `task5-app-private-rt`

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |

A temporary NAT Gateway was used during setup to provide outbound connectivity for installing Nginx. The NAT Gateway was subsequently deleted, and the default NAT route was removed.

### Database private route table

Route table: `task5-db-private-rt`

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |

The database route table has no default internet route.

### Routing observations

- The ALB is internet-facing.
- The application EC2 instance has no public IP address.
- The application-private subnet has no default internet route after cleanup.
- The database-private subnets have no direct internet route.
- Removing the NAT Gateway reduces ongoing NAT charges but also removes that outbound internet path.

## 6. Security Groups

| Security group | Inbound rule |
|---|---|
| `task5-alb-sg` | HTTP TCP 80 from `0.0.0.0/0` |
| `task5-app-sg` | HTTP TCP 80 from the ALB security group |
| `task5-db-sg` | PostgreSQL TCP 5432 from the application security group |

The database security group also had a MySQL TCP 3306 rule from the application security group. This rule is unnecessary for the PostgreSQL-only lab and should be removed after confirming that no other workload depends on it.

### Security considerations

- The ALB accepts HTTP from the internet in this lab.
- The application tier accepts HTTP only from the ALB security group.
- The database accepts PostgreSQL connections from the application security group.
- RDS is configured as not publicly accessible.
- RDS storage encryption was disabled in this lab. Enable encryption for a security-hardened deployment.

## 7. Application Load Balancer

| Setting | Value |
|---|---|
| ALB | `task5-app-alb` |
| Target group | `task5-app-tg` |
| Listener | HTTP |
| Listener port | `80` |
| Registered target | `i-028e01bd3aa36ae30` |
| Target port | `80` |
| Health-check path | `/` |
| Expected success code | `200` |

### Check target health

Run in a terminal with AWS CLI credentials and access to the `ap-south-1` region:

```bash
aws elbv2 describe-target-health \
  --region ap-south-1 \
  --target-group-arn "$(
    aws elbv2 describe-target-groups \
      --region ap-south-1 \
      --names task5-app-tg \
      --query 'TargetGroups[0].TargetGroupArn' \
      --output text
  )" \
  --output table
```

Check that the registered target's state is `healthy`.

### Test the ALB endpoint

```bash
ALB_DNS=$(aws elbv2 describe-load-balancers \
  --region ap-south-1 \
  --names task5-app-alb \
  --query 'LoadBalancers[0].DNSName' \
  --output text)

echo "http://$ALB_DNS"
curl -I --max-time 15 "http://$ALB_DNS"
```

Expected result: an HTTP success response, typically `HTTP/1.1 200 OK`, when the ALB and application are healthy.

## 8. Nginx on the Application EC2

Run these commands on the application EC2 instance:

```bash
# Refresh package metadata
sudo dnf makecache

# Install Nginx
sudo dnf install nginx -y

# Enable and start Nginx
sudo systemctl enable --now nginx

# Check service status
sudo systemctl status nginx --no-pager

# Test local HTTP
curl -I http://localhost
```

**Observed lab result:** The local Nginx test returned `HTTP 200 OK`.

### Troubleshooting Nginx

```bash
sudo systemctl status nginx --no-pager
sudo journalctl -u nginx --no-pager -n 50
sudo ss -lntp
curl -I http://localhost
```

If Nginx is healthy locally but the ALB target is unhealthy, check the target group health-check path and port, the ALB listener, and the application security group's inbound rule.

## 9. RDS PostgreSQL

| Setting | Value |
|---|---|
| DB instance | `task5-test-db` |
| Database | `task5db` |
| Engine | PostgreSQL |
| Port | `5432` |
| DB subnet group | `task5-db-subnet-group` |
| Publicly accessible | `False` |
| Storage encrypted | `False` |

### Check RDS status

```bash
aws rds describe-db-instances \
  --region ap-south-1 \
  --db-instance-identifier task5-test-db \
  --query 'DBInstances[0].[DBInstanceStatus,PubliclyAccessible,StorageEncrypted,Endpoint.Address,Endpoint.Port]' \
  --output table
```

Confirm that the instance is available and `PubliclyAccessible` is `False`.

The database master password is managed by AWS Secrets Manager. Never store credentials in this repository.

**Note:** RDS availability and private configuration were validated. Do not claim a successful application-to-database query unless you have actually tested connectivity and authentication.

## 10. Temporary Resource Cleanup

The temporary NAT Gateway and EC2 Instance Connect Endpoint were deleted after setup. The NAT default route was also removed from the application private route table.

### Check the application-private routes

```bash
aws ec2 describe-route-tables \
  --region ap-south-1 \
  --route-table-ids rtb-09d78e4b994afde2d \
  --query 'RouteTables[0].Routes[*].[DestinationCidrBlock,GatewayId,State]' \
  --output table
```

Expected final route: `10.0.0.0/16` to `local`, with no default NAT route.

### Cost awareness

The ALB, EC2 instance, and RDS database may continue to incur charges while running. Review the resources and remove those no longer needed.

## 11. Validation Summary

| Validation | Result |
|---|---|
| VPC and subnet layout | Configured |
| Public and private route tables | Configured |
| Nginx local HTTP response | HTTP 200 OK |
| ALB target health | Verified healthy during the lab |
| RDS PostgreSQL | Created and available |
| RDS public accessibility | `False` |
| Temporary NAT Gateway | Deleted after setup |
| EC2 Instance Connect Endpoint | Deleted after setup |
| RDS storage encryption | Disabled; improvement identified |

## 12. Screenshots

Evidence is stored in the `screenshots/` directory:

- `01-vpc-details.png`
- `02-vpc-subnets.png`
- `03-public-route-table.png`
- `04-app-private-route-table.png`
- `05-db-private-route-table.png`
- `06-alb-target-healthy.png`
- `07-alb-website.png`
- `08-rds-private-status.png`

## 13. Key Learning Outcomes

- Designed public, application-private, and database-private subnet tiers.
- Configured route tables and Internet Gateway routing.
- Used an ALB to expose an application without assigning a public IP to the application server.
- Applied security-group references to restrict tier-to-tier traffic.
- Provisioned a private RDS PostgreSQL database.
- Practiced Nginx and ALB troubleshooting.
- Used temporary NAT and EC2 Instance Connect resources during setup, then removed them.
- Identified security improvements, including removing unnecessary database ports and enabling RDS storage encryption.

