# Task 05 — Multi-Tier VPC Cheatsheet

## 1. VPC and Subnets

- **Region:** `ap-south-1` (Mumbai)
- **VPC:** `multitier-vpc`
- **VPC CIDR:** `10.0.0.0/16`

| Tier | CIDR | Availability Zone |
|---|---|---|
| Public A | `10.0.0.0/20` | `ap-south-1a` |
| Public B | `10.0.16.0/20` | `ap-south-1b` |
| App Private A | `10.0.32.0/20` | `ap-south-1a` |
| App Private B | `10.0.48.0/20` | `ap-south-1b` |
| DB Private A | `10.0.64.0/20` | `ap-south-1a` |
| DB Private B | `10.0.80.0/20` | `ap-south-1b` |

## 2. Route Tables

### Public route table

- **Name:** `task5-public-rt`
- `10.0.0.0/16` → `local`
- `0.0.0.0/0` → Internet Gateway

### App private route table

- **Name:** `task5-app-private-rt`
- `10.0.0.0/16` → `local`
- No default NAT route after temporary NAT cleanup.

### DB private route table

- **Name:** `task5-db-private-rt`
- `10.0.0.0/16` → `local`
- No default internet route.

## 3. Security Groups

| Security group | Inbound rule |
|---|---|
| `task5-alb-sg` | HTTP TCP 80 from `0.0.0.0/0` |
| `task5-app-sg` | HTTP TCP 80 from the ALB security group |
| `task5-db-sg` | PostgreSQL TCP 5432 from the app security group |

**Cleanup note:** The DB security group also had TCP 3306 from the app security group. Remove it after verifying it is not required by another workload.

## 4. Application Load Balancer

- **ALB:** `task5-app-alb`
- **Target group:** `task5-app-tg`
- **Listener:** HTTP TCP 80
- **Target instance:** `i-028e01bd3aa36ae30`
- **Target port:** `80`
- **Health-check path:** `/`
- **Expected HTTP code:** `200`

### Check target health

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

### Test the ALB

```bash
ALB_DNS=$(aws elbv2 describe-load-balancers \
  --region ap-south-1 \
  --names task5-app-alb \
  --query 'LoadBalancers[0].DNSName' \
  --output text)

echo "http://$ALB_DNS"
curl -I --max-time 15 "http://$ALB_DNS"
```

## 5. Nginx Commands

Run on the application EC2 instance:

```bash
# Install
sudo dnf makecache
sudo dnf install nginx -y

# Start and enable
sudo systemctl enable --now nginx

# Check service
sudo systemctl status nginx --no-pager

# Test local HTTP
curl -I http://localhost

# Check listening ports
sudo ss -lntp

# Inspect logs
sudo journalctl -u nginx --no-pager -n 50
```

Expected local response: `HTTP 200 OK`.

## 6. RDS PostgreSQL

- **Instance:** `task5-test-db`
- **Database:** `task5db`
- **Engine:** PostgreSQL
- **Port:** `5432`
- **Subnet group:** `task5-db-subnet-group`
- **Publicly accessible:** `False`
- **Storage encrypted:** `False`

### Check RDS status

```bash
aws rds describe-db-instances \
  --region ap-south-1 \
  --db-instance-identifier task5-test-db \
  --query 'DBInstances[0].[DBInstanceStatus,PubliclyAccessible,StorageEncrypted,Endpoint.Address,Endpoint.Port]' \
  --output table
```

The database master password is managed by AWS Secrets Manager. Do not store credentials in GitHub.

## 7. Temporary Resource Cleanup

The temporary NAT Gateway and EC2 Instance Connect Endpoint were deleted after setup. The NAT default route was removed.

### Check app-private routes

```bash
aws ec2 describe-route-tables \
  --region ap-south-1 \
  --route-table-ids rtb-09d78e4b994afde2d \
  --query 'RouteTables[0].Routes[*].[DestinationCidrBlock,GatewayId,State]' \
  --output table
```

Expected final route: `10.0.0.0/16` → `local`, with no default NAT route.

## 8. Troubleshooting Notes

### ALB target unhealthy

1. Verify Nginx is running.
2. Run `curl -I http://localhost` on the application instance.
3. Confirm the target group uses HTTP port 80 and health-check path `/`.
4. Confirm the app security group allows TCP 80 from the ALB security group.
5. Confirm the ALB listener forwards to the correct target group.
6. Review the target health reason using `describe-target-health`.

### ALB returns HTTP 503

1. Check target health and target registration.
2. Verify listener rules and target group association.
3. Confirm Nginx is listening on port 80.
4. Check the security group rules between the ALB and application instance.

### Private EC2 cannot access the internet

The temporary NAT Gateway was removed. The application-private route table intentionally has no default internet route.

Do not assign a public IP to the application instance merely to restore outbound access. If outbound connectivity is required, select an approved network design such as a NAT Gateway or appropriate VPC endpoints.

### SSM session unavailable

SSM requires suitable IAM permissions and connectivity to Systems Manager endpoints. Temporary access in this lab was obtained through an EC2 Instance Connect Endpoint, which was deleted after use.

### RDS connection fails

1. Confirm RDS is available.
2. Verify the database endpoint and port.
3. Confirm the DB security group allows TCP 5432 from the app security group.
4. Check VPC routing and name resolution.
5. Validate credentials securely through the approved secrets mechanism.

### Security improvement

RDS storage encryption was disabled in this lab. Enable encryption when designing a security-hardened deployment. An existing unencrypted RDS instance may require a migration or snapshot-and-restore approach to enable encryption; evaluate the impact before changing it.

## 9. Interview Summary

**Explain the traffic flow:** Users connect to the public ALB. The ALB forwards requests to Nginx on a private EC2 instance. The application tier can connect to PostgreSQL through a security-group rule on port 5432. The RDS instance is not publicly accessible.

**Why use private subnets?** They reduce direct exposure of application and database resources. Access is controlled through routing and security groups.

**Why two Availability Zones?** They distribute subnets across separate zones and support resilient architecture. This lab has one registered application EC2 instance and one RDS instance, so it does not by itself provide full application or database high availability.

**Why is the application instance private?** It receives application traffic through the ALB rather than requiring direct internet access to the instance.

**Why remove the temporary NAT Gateway?** To avoid ongoing NAT Gateway charges when the temporary outbound path is no longer required. The trade-off is that the application-private subnet no longer has that default outbound internet route.

**Security caveat:** RDS storage encryption was disabled in this lab. Enable encryption for a security-hardened deployment.

