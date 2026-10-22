\# AWS Linux \& CloudOps Hands-On Labs



\*\*Hands-on AWS infrastructure, Linux administration, security, networking, monitoring, and troubleshooting portfolio\*\*



This repository contains eight practical labs built to develop and demonstrate skills relevant to AWS Cloud Operations, Cloud Support Engineering, L2 Infrastructure Support, and Linux Administration.



Each lab focuses on a specific operational requirement, from managing Linux users and securing storage to deploying load-balanced applications, implementing auto scaling, and centralizing monitoring with Amazon CloudWatch.



The repository documents the implementation approach, configuration decisions, validation steps, troubleshooting guidance, and supporting screenshots.



\## Table of Contents



1\. \[Portfolio Overview](#portfolio-overview)

2\. \[Technical Skills](#technical-skills)

3\. \[Hands-On Projects](#hands-on-projects)

4\. \[Repository Structure](#repository-structure)

5\. \[How Each Lab Is Documented](#how-each-lab-is-documented)

6\. \[Validation and Troubleshooting Approach](#validation-and-troubleshooting-approach)

7\. \[Security and Cost Management](#security-and-cost-management)

8\. \[Learning Outcomes](#learning-outcomes)

9\. \[Master Cheatsheet](#master-cheatsheet)



\## Portfolio Overview



The labs are organized around common AWS infrastructure and CloudOps responsibilities:



\- Linux user administration and secure SSH authentication.

\- EC2 web server deployment and service validation.

\- Encrypted block storage.

\- Application Load Balancer configuration and health checks.

\- Multi-tier VPC networking and subnet segmentation.

\- IAM permissions and private S3 access.

\- Load balancing and Auto Scaling.

\- Centralized monitoring, custom metrics, application logs, and alerting.



The objective is to understand how the infrastructure works, validate the configuration, identify operational issues, and document practical troubleshooting procedures.



\## Technical Skills



| Category | Technologies and concepts |

|---|---|

| Compute | Amazon EC2 |

| Linux administration | User management, groups, sudo privileges, SSH keys, file permissions, Nginx |

| Networking | VPC, subnets, routing, security groups, Network ACLs, load balancing |

| Storage | EBS volumes, encryption, Amazon S3 |

| Security | IAM roles, policies, least privilege, SSH authentication, private resource access |

| High availability | Application Load Balancer, target groups, health checks |

| Scaling | Launch templates, Auto Scaling Groups, target tracking |

| Monitoring | CloudWatch metrics, dashboards, alarms, log groups and log streams |

| Operations | Systems Manager Run Command, Parameter Store, SNS notification integration |

| Documentation | Markdown, implementation guides, cheatsheets, screenshots, troubleshooting notes |



\## Hands-On Projects



\### Task 1 — Linux Users and SSH Authentication



\*\*Project:\*\* \[Linux Users \& SSH Authentication](./task-01-linux-users-ssh/)



\*\*Objective:\*\* Practice Linux account administration, controlled privilege escalation, and key-based SSH authentication on an Amazon Linux EC2 instance.



\*\*What was implemented:\*\*



\- Created the Linux users `devuser` and `opsuser`.

\- Configured sudo access for `opsuser` through `/etc/sudoers.d/opsuser`.

\- Validated the sudoers configuration with `visudo`.

\- Created an Ed25519 SSH key pair and configured the authorized public key.

\- Applied restrictive permissions to the SSH directory and `authorized\_keys`.

\- Verified privileged access and checked that an unauthorized key was rejected.



\*\*Key learning outcomes:\*\*



Linux user and privilege management, SSH public-key authentication, secure file permissions, sudoers validation, and access troubleshooting.



\*\*Documentation:\*\* \[Task 1 README](./task-01-linux-users-ssh/README.md) · \[Task 1 Cheatsheet](./task-01-linux-users-ssh/CHEATSHEET.md)



\### Task 2 — Nginx Web Server on EC2



\*\*Project:\*\* \[Nginx on EC2](./task-02-nginx-ec2/)



\*\*Objective:\*\* Deploy and validate a Linux-hosted web server on Amazon EC2.



\*\*Project focus:\*\*



\- Provision and access an EC2 Linux instance.

\- Install and configure Nginx.

\- Manage the web server service.

\- Configure network access for HTTP traffic on TCP port 80.

\- Validate web server availability through a browser or HTTP request.

\- Document the deployment and verification process.



\*\*Key learning outcomes:\*\*



EC2 administration, Linux package management, service management, security group configuration, HTTP connectivity, and basic web server troubleshooting.



\*\*Documentation:\*\* \[Task 2 README](./task-02-nginx-ec2/README.md) · \[Task 2 Cheatsheet](./task-02-nginx-ec2/CHEATSHEET.md)



\### Task 3 — Encrypted EBS Volume



\*\*Project:\*\* \[Encrypted EBS Volume](./task-03-encrypted-ebs-volume/)



\*\*Objective:\*\* Explore AWS block storage and the security implications of EBS encryption.



\*\*Project focus:\*\*



\- Understand the relationship between EC2 instances and EBS volumes.

\- Examine volume encryption and its role in protecting stored data.

\- Work with volume attachment and Linux disk operations as documented in the task.

\- Validate the resulting storage configuration.

\- Record relevant checks and troubleshooting steps.



\*\*Key learning outcomes:\*\*



EBS volume lifecycle management, storage security, encryption concepts, Linux disk administration, and storage troubleshooting.



\*\*Documentation:\*\* \[Task 3 README](./task-03-encrypted-ebs-volume/README.md) · \[Task 3 Cheatsheet](./task-03-encrypted-ebs-volume/CHEATSHEET.md)



\### Task 4 — Nginx Application Load Balancer



\*\*Project:\*\* \[Nginx Application Load Balancer](./task-04-nginx-alb/)



\*\*Objective:\*\* Learn how an Application Load Balancer distributes HTTP requests to registered targets and uses health checks to determine target availability.



\*\*Project focus:\*\*



\- Configure an Application Load Balancer.

\- Create and associate a target group.

\- Configure listener and target health-check settings.

\- Review security group rules required for traffic between the load balancer and targets.

\- Validate the application endpoint and investigate target health when necessary.



\*\*Key learning outcomes:\*\*



ALB listeners, target groups, HTTP health checks, security group traffic flows, and troubleshooting unhealthy targets.



\*\*Documentation:\*\* \[Task 4 README](./task-04-nginx-alb/README.md) · \[Task 4 Cheatsheet](./task-04-nginx-alb/CHEATSHEET.md)



\### Task 5 — Multi-Tier VPC Architecture



\*\*Project:\*\* \[Multi-Tier VPC Architecture](./task-05-multi-tier-vpc/)



\*\*Objective:\*\* Understand how AWS network components are organized to separate application tiers and control network traffic.



\*\*Project focus:\*\*



\- Build a VPC-based network environment.

\- Organize resources into public and private subnet tiers.

\- Configure route tables and network paths.

\- Apply security group and network access controls.

\- Review how traffic should flow between application tiers and external networks.



\*\*Key learning outcomes:\*\*



VPC design, subnet segmentation, route tables, network security, traffic flow, and troubleshooting connectivity between resources.



\*\*Documentation:\*\* \[Task 5 README](./task-05-multi-tier-vpc/README.md) · \[Task 5 Cheatsheet](./task-05-multi-tier-vpc/CHEATSHEET.md)



\### Task 6 — IAM and Private S3 Access



\*\*Project:\*\* \[IAM \& Private S3 Access](./task-06-iam-private-s3-access/)



\*\*Objective:\*\* Explore AWS identity and access management and how permissions control access to private S3 resources.



\*\*Project focus:\*\*



\- Review IAM identities, roles, and policy permissions.

\- Configure access according to the lab's requirements.

\- Work with a private S3 bucket and controlled access.

\- Validate the expected access behavior.

\- Document permission troubleshooting and least-privilege considerations.



\*\*Key learning outcomes:\*\*



IAM policy evaluation, access control, S3 permissions, private resource protection, and troubleshooting access-denied errors.



\*\*Documentation:\*\* \[Task 6 README](./task-06-iam-private-s3-access/README.md) · \[Task 6 Cheatsheet](./task-06-iam-private-s3-access/CHEATSHEET.md)



\### Task 7 — Application Load Balancer and Auto Scaling



\*\*Project:\*\* \[ALB \& Auto Scaling](./task-07-alb-auto-scaling/)



\*\*Objective:\*\* Deploy a load-balanced web application using an EC2 Auto Scaling Group and understand how health checks and scaling policies support application availability.



\*\*What was implemented:\*\*



\- Configured the `task7-nginx-alb` Application Load Balancer.

\- Created the `task7-nginx-tg` target group.

\- Configured the `task7-nginx-asg` Auto Scaling Group with a desired capacity of two instances and a maximum capacity of four.

\- Used the `task7-nginx-template` launch template.

\- Configured Nginx on instances launched by the group.

\- Validated target health and HTTP responses through the load balancer.

\- Configured CPU target tracking with a 50% target.



\*\*Key learning outcomes:\*\*



Launch templates, Auto Scaling Groups, ALB target health, health-check troubleshooting, scaling policies, and application availability.



\*\*Validation note:\*\* The load balancer returned HTTP 200 responses, and requests reached both instances. A complete scale-out event should be documented separately if and when it is tested.



\*\*Documentation:\*\* \[Task 7 README](./task-07-alb-auto-scaling/README.md) · \[Task 7 Cheatsheet](./task-07-alb-auto-scaling/CHEATSHEET.md)



\### Task 8 — CloudWatch Monitoring and SNS Notifications



\*\*Project:\*\* \[CloudWatch Monitoring](./task-08-cloudwatch-monitoring/)



\*\*Objective:\*\* Build centralized operational monitoring for EC2 instances using CloudWatch metrics, dashboards, alarms, application logs, and notification integration.



\*\*What was implemented:\*\*



\- Created the `task8-operations-dashboard` CloudWatch dashboard.

\- Configured CPU utilization and instance status-check alarms.

\- Created the `Task8CloudWatchAgentRole` instance role with the required Systems Manager and CloudWatch Agent policies.

\- Used Systems Manager Run Command to install and configure the CloudWatch Agent.

\- Stored the agent configuration in Parameter Store as `AmazonCloudWatch-agent-task8-config`.

\- Published custom memory and disk metrics under the `Task8/EC2` namespace.

\- Configured collection of Nginx access and error logs.

\- Verified the custom metrics and inspected Nginx access log events in CloudWatch.

\- Configured SNS as an alarm notification destination.



\*\*Key learning outcomes:\*\*



CloudWatch dashboards, standard and custom metrics, alarms, guest OS monitoring, centralized logs, Systems Manager remote management, Parameter Store, IAM permissions, and SNS notification integration.



\*\*Validation note:\*\* The custom memory and disk metrics and Nginx log events were verified. SNS email delivery and actual alarm triggering should be reported as tested only after successful end-to-end verification.



\*\*Documentation:\*\* \[Task 8 README](./task-08-cloudwatch-monitoring/README.md) · \[Task 8 Cheatsheet](./task-08-cloudwatch-monitoring/CHEATSHEET.md)



\## Repository Structure



```text

aws-linux-cloudops-labs/

├── README.md

├── MASTER-CHEATSHEET.md

├── task-01-linux-users-ssh/

│   ├── README.md

│   ├── CHEATSHEET.md

│   └── screenshots/

├── task-02-nginx-ec2/

│   ├── README.md

│   ├── CHEATSHEET.md

│   └── screenshots/

├── task-03-encrypted-ebs-volume/

├── task-04-nginx-alb/

├── task-05-multi-tier-vpc/

├── task-06-iam-private-s3-access/

├── task-07-alb-auto-scaling/

└── task-08-cloudwatch-monitoring/

```



Some task directories may contain additional configuration or evidence files. This tree illustrates the intended documentation structure; it is not a guarantee that every optional file currently exists.



\## How Each Lab Is Documented



Each task's README should describe:



1\. The problem and objective.

2\. The AWS architecture and resources involved.

3\. The implementation steps and important configuration decisions.

4\. Validation commands, console checks, or application responses.

5\. Troubleshooting scenarios and root causes encountered.

6\. Security considerations and cleanup instructions.

7\. Screenshots that support the reported results.



The corresponding cheatsheet should make it easy to revise the important commands, concepts, and troubleshooting checks without rereading the full implementation guide.



\## Validation and Troubleshooting Approach



The labs emphasize verifying outcomes instead of relying solely on successful resource creation.



Examples include:



\- Confirming that SSH authentication and sudo access behave as expected.

\- Checking web server service status and HTTP responses.

\- Reviewing load balancer target health and application availability.

\- Inspecting IAM permissions when access is denied.

\- Checking CloudWatch metrics, alarms, and log events.

\- Reviewing command output and service status when troubleshooting fails.



Where a scenario has not been tested, the documentation should identify it as a recommended test rather than claim a successful result.



\## Security and Cost Management



\- Use least-privilege IAM permissions.

\- Protect private keys and never commit credentials or secrets.

\- Restrict inbound network access to the required sources and ports.

\- Use encryption where appropriate for stored data.

\- Set appropriate log retention policies.

\- Review the cost of load balancers, EC2 instances, EBS volumes, CloudWatch ingestion, and other billable resources.

\- Delete temporary lab resources when they are no longer required.



\## Learning Outcomes



This portfolio demonstrates practical exposure to:



\- Linux administration and secure remote access.

\- AWS compute, storage, and networking.

\- IAM and private resource access.

\- Load balancing and Auto Scaling fundamentals.

\- Monitoring, logging, and alerting.

\- Cloud infrastructure troubleshooting and operational documentation.



The projects are intended to support technical learning and interview discussions. The actual scope of each implementation and validation is described in its task-specific documentation.



\## Master Cheatsheet



For quick revision across all labs, see \[`MASTER-CHEATSHEET.md`](./MASTER-CHEATSHEET.md) once that file has been created.



For detailed commands and task-specific troubleshooting, use the `CHEATSHEET.md` inside the corresponding project directory.



\---



\*\*Portfolio repository:\*\* \[aws-linux-cloudops-labs](https://github.com/shankar0797/aws-linux-cloudops-labs)

