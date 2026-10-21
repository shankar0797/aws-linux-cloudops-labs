\# Task 05 — Multi-Tier VPC Cheatsheet



\## 1. VPC and Subnets



\- \*\*VPC CIDR:\*\* `10.0.0.0/16`

\- \*\*Public subnets:\*\* `10.0.0.0/20`, `10.0.16.0/20`

\- \*\*App private subnets:\*\* `10.0.32.0/20`, `10.0.48.0/20`

\- \*\*DB private subnets:\*\* `10.0.64.0/20`, `10.0.80.0/20`

\- \*\*Region:\*\* `ap-south-1` (Mumbai)

\- \*\*Availability Zones:\*\* `ap-south-1a`, `ap-south-1b`



\## 2. Route Tables



\### Public route table



\- Route table: `task5-public-rt`

\- Route: `10.0.0.0/16` → `local`

\- Route: `0.0.0.0/0` → Internet Gateway



\### App private route table



\- Route table: `task5-app-private-rt`

\- Final route: `10.0.0.0/16` → `local`

\- Temporary NAT default route was removed after Nginx installation.



\### DB private route table



\- Route table: `task5-db-private-rt`

\- Route: `10.0.0.0/16` → `local`

\- No default internet route.



\## 3. Security Groups



| Security group | Inbound rule |

|---|---|

| `task5-alb-sg` | HTTP TCP 80 from `0.0.0.0/0` |

| `task5-app-sg` | HTTP TCP 80 from the ALB security group |

| `task5-db-sg` | PostgreSQL TCP 5432 from the app security group |



The DB security group also currently has a MySQL TCP 3306 rule from the app security group. It is unnecessary for this PostgreSQL-only lab and can be removed.



\## 4. Application Load Balancer



\- ALB: `task5-app-alb`

\- Target group: `task5-app-tg`

\- Listener: HTTP TCP 80

\- Target: `i-028e01bd3aa36ae30`

\- Target port: 80

\- Health-check path: `/`

\- Expected HTTP code: `200`



\### Check target health



```bash

aws elbv2 describe-target-health \\

&#x20; --region ap-south-1 \\

&#x20; --target-group-arn "$(aws elbv2 describe-target-groups \\

&#x20;   --region ap-south-1 \\

&#x20;   --names task5-app-tg \\

&#x20;   --query 'TargetGroups\[0].TargetGroupArn' \\

&#x20;   --output text)" \\

&#x20; --output table

```



\### Test the ALB



```bash

ALB\_DNS=$(aws elbv2 describe-load-balancers \\

&#x20; --region ap-south-1 \\

&#x20; --names task5-app-alb \\

&#x20; --query 'LoadBalancers\[0].DNSName' \\

&#x20; --output text)



echo "http://$ALB\_DNS"

curl -I --max-time 15 "http://$ALB\_DNS"

```



\## 5. Nginx Commands (run on the EC2 instance)



```bash

sudo dnf makecache

sudo dnf install nginx -y

sudo systemctl enable --now nginx

sudo systemctl status nginx --no-pager

curl -I http://localhost

```



Expected local response: HTTP 200 OK.



\## 6. RDS PostgreSQL



\- Instance: `task5-test-db`

\- Database: `task5db`

\- Engine: PostgreSQL

\- Port: `5432`

\- DB subnet group: `task5-db-subnet-group`

\- Publicly accessible: `False`

\- Storage encrypted: `False`



\### Check RDS status



```bash

aws rds describe-db-instances \\

&#x20; --region ap-south-1 \\

&#x20; --db-instance-identifier task5-test-db \\

&#x20; --query 'DBInstances\[0].\[DBInstanceStatus,PubliclyAccessible,StorageEncrypted,Endpoint.Address,Endpoint.Port]' \\

&#x20; --output table

```



The database master password is managed by AWS Secrets Manager. Do not store credentials in this repository.



\## 7. Temporary Resource Cleanup



The temporary NAT Gateway and EC2 Instance Connect Endpoint were deleted after setup.



\### Check app-private routes



```bash

aws ec2 describe-route-tables \\

&#x20; --region ap-south-1 \\

&#x20; --route-table-ids rtb-09d78e4b994afde2d \\

&#x20; --query 'RouteTables\[0].Routes\[\*].\[DestinationCidrBlock,GatewayId,State]' \\

&#x20; --output table

```



Expected final route: `10.0.0.0/16` → `local`.



\## 8. Troubleshooting Notes



\### ALB target unhealthy



1\. Verify Nginx is running.

2\. Test `curl -I http://localhost` on the instance.

3\. Confirm the target group uses HTTP port 80 and path `/`.

4\. Confirm the app security group allows TCP 80 from the ALB security group.

5\. Confirm the ALB listener forwards to the correct target group.



\### Private EC2 cannot access the internet



The temporary NAT Gateway was removed. The app private route table intentionally has no default internet route. Do not add a public IP to the app instance just to solve outbound access.



\### SSM session unavailable



SSM requires appropriate IAM permissions and connectivity to the Systems Manager endpoints. In this lab, temporary access was obtained through an EC2 Instance Connect Endpoint, which was deleted after use.



\## 9. Interview Summary



\*\*Explain the traffic flow:\*\* Users connect to the public ALB. The ALB forwards requests to Nginx on a private EC2 instance. The application tier can connect to PostgreSQL through a security-group rule on port 5432. The RDS instance is private and is not directly accessible from the internet.



\*\*Why use private subnets?\*\* They reduce direct exposure of application and database resources. Access is controlled through routing and security groups.



\*\*Why two Availability Zones?\*\* They provide subnet distribution across separate zones and support resilient architecture. This lab uses one application EC2 and one RDS instance, so it does not by itself provide full application or database high availability.



\*\*Security caveat:\*\* RDS storage encryption is disabled in this lab. Enable encryption for a security-hardened deployment.



