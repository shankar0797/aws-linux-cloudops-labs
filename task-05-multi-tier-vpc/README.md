\# Task 05 — AWS Multi-Tier VPC Architecture



\## Overview



Designed and deployed a multi-tier AWS network architecture in the Mumbai region (`ap-south-1`). The architecture separates public-facing load balancing, private application compute, and private database resources across two Availability Zones.



\## Architecture



\- \*\*VPC:\*\* `10.0.0.0/16`

\- \*\*Public tier:\*\* Two public subnets across `ap-south-1a` and `ap-south-1b`

\- \*\*Application tier:\*\* Two private application subnets

\- \*\*Database tier:\*\* Two private database subnets

\- \*\*Load balancing:\*\* Internet-facing Application Load Balancer (ALB)

\- \*\*Application server:\*\* Amazon Linux 2023 EC2 running Nginx

\- \*\*Database:\*\* Private Amazon RDS for PostgreSQL

\- \*\*Security:\*\* Security groups restrict traffic between tiers



\## AWS Resources



| Resource | Name / Configuration |

|---|---|

| Region | `ap-south-1` |

| VPC | `multitier-vpc` |

| VPC CIDR | `10.0.0.0/16` |

| Public route table | `task5-public-rt` |

| App route table | `task5-app-private-rt` |

| DB route table | `task5-db-private-rt` |

| ALB | `task5-app-alb` |

| Target group | `task5-app-tg` |

| App EC2 | `i-028e01bd3aa36ae30` |

| App security group | `task5-app-sg` |

| ALB security group | `task5-alb-sg` |

| DB security group | `task5-db-sg` |

| RDS database | `task5-test-db` |

| RDS engine | PostgreSQL |

| RDS port | `5432` |



\## Subnet Design



| Tier | Subnet CIDR | Availability Zone |

|---|---|---|

| Public A | `10.0.0.0/20` | `ap-south-1a` |

| Public B | `10.0.16.0/20` | `ap-south-1b` |

| App Private A | `10.0.32.0/20` | `ap-south-1a` |

| App Private B | `10.0.48.0/20` | `ap-south-1b` |

| DB Private A | `10.0.64.0/20` | `ap-south-1a` |

| DB Private B | `10.0.80.0/20` | `ap-south-1b` |



\## Traffic Flow



1\. Users access the internet-facing ALB over HTTP on port 80.

2\. The ALB forwards requests to the registered application EC2 instance.

3\. The application security group permits HTTP on port 80 from the ALB security group.

4\. The application tier can access the database tier through the database security group on PostgreSQL port 5432.

5\. The RDS instance is not publicly accessible.

6\. The DB subnets have no direct internet route.



\## Routing



\- \*\*Public route table:\*\* `10.0.0.0/16` local route and `0.0.0.0/0` through the Internet Gateway.

\- \*\*Application private route table:\*\* local route only after temporary NAT cleanup.

\- \*\*Database private route table:\*\* local route only.



A temporary NAT Gateway was used during setup to provide outbound connectivity for installing Nginx. It was deleted after setup, and its default route was removed from the application private route table.



\## Validation



\- Installed and started Nginx on the private EC2 instance.

\- Verified Nginx locally using `curl -I http://localhost`; received HTTP 200 OK.

\- Registered the application EC2 instance with `task5-app-tg`.

\- Verified the target became healthy.

\- Created the RDS PostgreSQL instance in the private DB subnet group.

\- Verified RDS became available and `PubliclyAccessible` was `False`.

\- Deleted the temporary NAT Gateway and EC2 Instance Connect Endpoint after access and setup.



\## Security Notes



\- The application EC2 instance has no public IP address.

\- The ALB security group accepts HTTP port 80 from the internet.

\- The application security group accepts HTTP port 80 only from the ALB security group.

\- The DB security group allows PostgreSQL port 5432 from the application security group.

\- RDS storage encryption was \*\*disabled\*\* in this lab and should be enabled for a security-hardened implementation.



\## Cost Considerations



The NAT Gateway and EC2 Instance Connect Endpoint were temporary resources and have been deleted. The ALB, EC2 instance, and RDS database may still incur charges while running. Delete resources that are no longer needed.



\## Screenshots



Evidence is stored in the `screenshots/` directory:



\- `01-vpc-details.png`

\- `02-vpc-subnets.png`

\- `03-public-route-table.png`

\- `04-app-private-route-table.png`

\- `05-db-private-route-table.png`

\- `06-alb-target-healthy.png`

\- `07-alb-website.png`

\- `08-rds-private-status.png`



\## Key Learning Outcomes



\- Designed public, application-private, and database-private subnet tiers.

\- Configured route tables and an Internet Gateway.

\- Used an ALB to expose an application without assigning a public IP to the app server.

\- Applied security-group references to restrict tier-to-tier traffic.

\- Provisioned a private RDS PostgreSQL database.

\- Troubleshot SSM access, configured temporary access using EC2 Instance Connect Endpoint, and removed temporary resources.



