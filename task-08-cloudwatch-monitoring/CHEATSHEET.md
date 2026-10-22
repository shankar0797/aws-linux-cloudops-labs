\# Task 8 — CloudWatch Monitoring Cheatsheet



\## 1. Key Services



\- \*\*CloudWatch Metrics:\*\* Collects and displays monitoring data.

\- \*\*CloudWatch Dashboards:\*\* Combines metrics and alarm widgets.

\- \*\*CloudWatch Alarms:\*\* Evaluates metrics against configured thresholds.

\- \*\*CloudWatch Agent:\*\* Collects guest OS metrics and log files.

\- \*\*Systems Manager Run Command:\*\* Executes commands remotely on managed instances.

\- \*\*Parameter Store:\*\* Stores the CloudWatch Agent configuration.

\- \*\*SNS:\*\* Sends notifications to subscribed endpoints.

\- \*\*IAM:\*\* Controls permissions for the EC2 instance and monitoring services.



\## 2. Lab Configuration



| Setting | Value |

|---|---|

| Region | `ap-south-1` |

| Dashboard | `task8-operations-dashboard` |

| Custom namespace | `Task8/EC2` |

| Memory metric | `mem\_used\_percent` |

| Disk metric | `disk\_used\_percent` |

| Access logs | `/task8/nginx/access` |

| Error logs | `/task8/nginx/error` |

| Parameter | `AmazonCloudWatch-agent-task8-config` |



\## 3. IAM Policies



Instance role: `Task8CloudWatchAgentRole`



\- `AmazonSSMManagedInstanceCore` — supports Systems Manager management.

\- `CloudWatchAgentServerPolicy` — supports publishing agent data to CloudWatch and retrieving required configuration from supported sources.



\## 4. CloudWatch Agent Configuration



The Parameter Store JSON configuration defines:



\- Collection interval: 60 seconds.

\- Namespace: `Task8/EC2`.

\- Memory measurement: `mem\_used\_percent`.

\- Disk measurement: `used\_percent` for `/`.

\- Nginx access and error log collection.



\### Systems Manager documents



\- `AWS-ConfigureAWSPackage` — installs the CloudWatch Agent package.

\- `AmazonCloudWatch-ManageAgent` — configures and manages the agent.



Configuration settings used:



\- Action: `configure`

\- Mode: `ec2`

\- Source: `ssm`

\- Configuration name: `AmazonCloudWatch-agent-task8-config`

\- Restart: `yes`



\## 5. Troubleshooting Checklist



\### Missing memory or disk metrics

1\. Open CloudWatch → Metrics → `Task8/EC2`.

2\. Check the metric name and instance dimension.

3\. Verify the JSON configuration.

4\. Check the CloudWatch Agent status and logs.

5\. Verify IAM permissions and wait for the next collection interval.



\### Missing Nginx logs

1\. Confirm Nginx is writing to the configured files.

2\. Verify the file paths and read permissions.

3\. Check CloudWatch log groups and streams.

4\. Review CloudWatch Agent status and error logs.



\### Systems Manager command failure

1\. Verify the EC2 instance is a managed node.

2\. Check SSM Agent status and connectivity.

3\. Check the instance IAM role.

4\. Inspect the Run Command output for the specific failure.



\### Alarm or notification issue

1\. Verify the monitored metric and threshold.

2\. Check evaluation periods and alarm state.

3\. Verify the alarm action references the intended SNS topic.

4\. Confirm the email subscription.

5\. Test notification delivery and verify receipt.



\## 6. Interview Revision



\*\*Q: Why use the CloudWatch Agent?\*\*  

To collect operating-system metrics and application log files that standard EC2 monitoring does not provide by default.



\*\*Q: Why use Parameter Store?\*\*  

To centrally store and reuse the CloudWatch Agent configuration.



\*\*Q: What does Systems Manager Run Command do?\*\*  

It executes commands or AWS documents remotely on managed instances without requiring a separate SSH session.



\*\*Q: What is the difference between EC2 status checks and CPU utilization?\*\*  

Status checks help identify instance or underlying system health problems. CPU utilization measures processor usage.



\*\*Q: What is the purpose of SNS?\*\*  

It distributes notifications to subscribed endpoints, such as email, when configured alarm actions are invoked.



\*\*Q: Why are custom metrics useful?\*\*  

They provide visibility into operating-system and application conditions, such as memory and disk utilization.



\*\*Q: Does creating an alarm prove notifications work?\*\*  

No. Confirm the SNS subscription, alarm action, and actual notification delivery.



\## 7. Cleanup



Before removing lab resources, identify the dependencies. Delete or disable only resources no longer required, and check CloudWatch log retention and ongoing charges.

