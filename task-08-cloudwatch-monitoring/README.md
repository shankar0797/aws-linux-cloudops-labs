\# Task 8: AWS CloudWatch Monitoring and SNS Notifications



\## Overview



This project demonstrates how to implement centralized monitoring for Amazon EC2 instances using Amazon CloudWatch, the CloudWatch Agent, AWS Systems Manager, Parameter Store, and Amazon SNS.



The lab extends the previous EC2 and Auto Scaling project by collecting operating-system metrics and Nginx logs, creating monitoring dashboards and alarms, and configuring notification delivery.



\## Objectives



\- Build a CloudWatch dashboard for EC2 operational monitoring.

\- Configure CPU and instance status-check alarms.

\- Install and configure the CloudWatch Agent using Systems Manager Run Command.

\- Store the agent configuration in Systems Manager Parameter Store.

\- Publish memory and disk utilization metrics to CloudWatch.

\- Collect Nginx access and error logs centrally.

\- Configure SNS as the alarm notification destination.



\## AWS Services Used



| Service | Purpose |

|---|---|

| Amazon EC2 | Hosts the Linux instances and Nginx |

| Amazon CloudWatch | Metrics, dashboards, logs, and alarms |

| CloudWatch Agent | Collects guest OS metrics and application logs |

| AWS Systems Manager Run Command | Remotely installs and configures the agent |

| Systems Manager Parameter Store | Stores the CloudWatch Agent JSON configuration |

| Amazon SNS | Delivers alarm notifications to subscribers |

| IAM | Grants the instances permission to use Systems Manager and publish monitoring data |



\## Environment



\- \*\*AWS Region:\*\* Asia Pacific (Mumbai), `ap-south-1`

\- \*\*EC2 instance IDs:\*\*

&#x20; - `i-0b004c0e7a2c1c095`

&#x20; - `i-0cc6d76fc75033ad7`

\- \*\*CloudWatch dashboard:\*\* `task8-operations-dashboard`

\- \*\*Custom metric namespace:\*\* `Task8/EC2`

\- \*\*Parameter Store parameter:\*\* `AmazonCloudWatch-agent-task8-config`



\## Implementation



\### 1. Configure IAM permissions



Created and attached the `Task8CloudWatchAgentRole` instance role with:



\- `AmazonSSMManagedInstanceCore`

\- `CloudWatchAgentServerPolicy`



These permissions support Systems Manager management and CloudWatch Agent publishing.



\### 2. Install the CloudWatch Agent



Used Systems Manager Run Command with the `AWS-ConfigureAWSPackage` document to install the `AmazonCloudWatchAgent` package on the EC2 instances.



\### 3. Store the agent configuration



Stored the agent configuration in Systems Manager Parameter Store using the parameter name:



`AmazonCloudWatch-agent-task8-config`



The configuration specifies a 60-second collection interval, the `Task8/EC2` namespace, memory and disk metrics, and Nginx log files.



\### 4. Configure the CloudWatch Agent



Used the `AmazonCloudWatch-ManageAgent` Systems Manager document with the following settings:



\- Action: `configure`

\- Mode: `ec2`

\- Configuration source: `ssm`

\- Configuration location: `AmazonCloudWatch-agent-task8-config`

\- Restart agent: `yes`



\### 5. Collect custom EC2 metrics



The agent publishes:



\- `mem\_used\_percent`

\- `disk\_used\_percent`



The metrics are available in CloudWatch under the `Task8/EC2` namespace, with instance dimensions for identifying the source EC2 instance.



\### 6. Centralize Nginx logs



Configured the agent to collect:



| Log file | CloudWatch log group |

|---|---|

| `/var/log/nginx/access.log` | `/task8/nginx/access` |

| `/var/log/nginx/error.log` | `/task8/nginx/error` |



The log streams use the EC2 instance ID. Access log events were verified in CloudWatch Logs.



\### 7. Create dashboards and alarms



Created the CloudWatch dashboard `task8-operations-dashboard` to consolidate operational monitoring.



Configured CPU utilization alarms and instance status-check alarms for the EC2 instances. The status-check alarms monitor `StatusCheckFailed`, using the Maximum statistic and a threshold of 1.



The alarms were configured with SNS notification destinations. The actual email notification delivery and alarm-triggering behavior should be tested separately before being considered fully validated.



\## Verification



\- Confirmed that custom memory and disk metrics appeared in CloudWatch.

\- Verified that the Nginx access and error log groups were available.

\- Inspected Nginx access log events in CloudWatch Logs.

\- Reviewed the CloudWatch dashboard and alarm configuration.

\- Used Systems Manager Run Command to install and configure the CloudWatch Agent.



\## Screenshots



The `screenshots/` directory contains four evidence files:



1\. `01-cloudwatch-dashboard.png`

2\. `02-cloudwatch-alarms.png`

3\. `03-memory-disk-metrics.png`

4\. `04-nginx-logs.png`



\## Operational Troubleshooting



| Issue | Checks and corrective action |

|---|---|

| Memory metric missing | Verify the agent configuration includes `mem\_used\_percent`, then check agent status and logs. |

| Disk metric missing | Verify the disk measurement, configured mount points, IAM permissions, and agent status. |

| Logs missing | Check the configured file paths, Nginx log file permissions, log group and stream names, and agent status. |

| Run Command fails | Check the instance's Systems Manager connectivity, SSM Agent status, IAM role, and command output. |

| Alarm does not trigger | Check the metric, threshold, evaluation period, datapoints to alarm, and alarm state. |

| Email notification not received | Verify the SNS subscription is confirmed, the alarm action points to the correct topic, and test notification delivery. |



\## Security and Cost Considerations



\- Follow least-privilege IAM practices.

\- Do not store credentials or secrets in monitoring configuration.

\- Set appropriate CloudWatch log retention periods.

\- Monitor custom metric, log ingestion, dashboard, and alarm charges.

\- Remove unused lab resources when testing is complete.



\## Outcome



This lab demonstrates centralized EC2 observability using CloudWatch metrics, dashboards, alarms, and logs, with Systems Manager for remote configuration and SNS for notification integration.



The lab provides a foundation for investigating resource utilization, instance health, and application-level issues in an AWS CloudOps environment.

