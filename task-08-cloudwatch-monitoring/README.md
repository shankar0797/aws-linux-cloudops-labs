# Task 8: AWS CloudWatch Monitoring and SNS Notifications

## Overview

This lab demonstrates centralized monitoring for Amazon EC2 instances using Amazon CloudWatch, the CloudWatch Agent, AWS Systems Manager, Systems Manager Parameter Store, and Amazon SNS.

The lab covers operating-system metrics, Nginx log collection, monitoring dashboards, CPU and instance status-check alarms, and SNS notification configuration.

## Objectives

- Build a CloudWatch dashboard for EC2 operational monitoring.
- Configure CPU utilization and instance status-check alarms.
- Install and configure the CloudWatch Agent using Systems Manager Run Command.
- Store the agent configuration in Systems Manager Parameter Store.
- Publish memory and disk utilization metrics to CloudWatch.
- Collect Nginx access and error logs centrally.
- Configure SNS as the alarm notification destination.

## AWS Services Used

| Service | Purpose |
|---|---|
| Amazon EC2 | Hosts Linux instances and Nginx |
| Amazon CloudWatch | Metrics, dashboards, logs, and alarms |
| CloudWatch Agent | Collects guest OS metrics and application logs |
| AWS Systems Manager Run Command | Remotely installs and configures the agent |
| Systems Manager Parameter Store | Stores the CloudWatch Agent JSON configuration |
| Amazon SNS | Delivers alarm notifications to subscribers |
| AWS IAM | Grants instances permission to use Systems Manager and publish monitoring data |

## Environment

- **AWS Region:** `ap-south-1` (Mumbai)
- **EC2 instance 1:** `i-0b004c0e7a2c1c095`
- **EC2 instance 2:** `i-0cc6d76fc75033ad7`
- **CloudWatch dashboard:** `task8-operations-dashboard`
- **Custom metric namespace:** `Task8/EC2`
- **Parameter Store parameter:** `AmazonCloudWatch-agent-task8-config`

## Implementation

### 1. Configure IAM permissions

Created and attached the `Task8CloudWatchAgentRole` instance role with these managed policies:

- `AmazonSSMManagedInstanceCore`
- `CloudWatchAgentServerPolicy`

These policies support Systems Manager management and CloudWatch Agent publishing. The instance must also have the required connectivity to Systems Manager endpoints.

### 2. Install the CloudWatch Agent

Used Systems Manager Run Command with the `AWS-ConfigureAWSPackage` document to install the `AmazonCloudWatchAgent` package on the EC2 instances.

### 3. Store the agent configuration

Stored the CloudWatch Agent configuration in Systems Manager Parameter Store using this parameter name:

`AmazonCloudWatch-agent-task8-config`

The configuration specifies a 60-second collection interval, the `Task8/EC2` namespace, memory and disk metrics, and Nginx log files.

### 4. Configure the CloudWatch Agent

Used the `AmazonCloudWatch-ManageAgent` Systems Manager document with the following settings:

- Action: `configure`
- Mode: `ec2`
- Configuration source: `ssm`
- Configuration location: `AmazonCloudWatch-agent-task8-config`
- Restart agent: `yes`

### 5. Collect memory and disk metrics

The agent publishes the following custom metrics:

- `mem_used_percent`
- `disk_used_percent`

The metrics are available in CloudWatch under the `Task8/EC2` namespace, with instance dimensions to identify the source EC2 instance.

### 6. Centralize Nginx logs

Configured the agent to collect Nginx access and error logs.

| Log file | CloudWatch log group |
|---|---|
| `/var/log/nginx/access.log` | `/task8/nginx/access` |
| `/var/log/nginx/error.log` | `/task8/nginx/error` |

Log streams use the EC2 instance ID. Nginx access log events were verified in CloudWatch Logs.

### 7. Create a CloudWatch dashboard

Created the `task8-operations-dashboard` dashboard to consolidate operational monitoring.

The dashboard provides a central view of configured monitoring data. The specific widgets and metrics should be checked against the dashboard configuration and screenshots.

### 8. Configure CloudWatch alarms and SNS

Configured CPU utilization alarms and instance status-check alarms for the EC2 instances.

The status-check alarms monitor `StatusCheckFailed`, using the `Maximum` statistic and a threshold of `1`.

Configured SNS notification destinations for the alarms.

**Validation limitation:** Alarm configuration was reviewed, but actual alarm triggering and end-to-end email delivery were not established by the evidence described in this lab. Verify these separately before claiming notification delivery is fully tested.

## Verification Results

- Confirmed that custom memory and disk metrics appeared in CloudWatch.
- Verified that the Nginx access and error log groups were available.
- Inspected Nginx access log events in CloudWatch Logs.
- Reviewed the CloudWatch dashboard and alarm configuration.
- Used Systems Manager Run Command to install and configure the CloudWatch Agent.

## Screenshots

The `screenshots/` directory contains four evidence files:

1. `01-cloudwatch-dashboard.png`
2. `02-cloudwatch-alarms.png`
3. `03-memory-disk-metrics.png`
4. `04-nginx-logs.png`

## Operational Troubleshooting

| Issue | Checks and corrective action |
|---|---|
| Memory metric missing | Verify the agent configuration includes `mem_used_percent`; check agent status, logs, and metric dimensions. |
| Disk metric missing | Verify the disk measurement, configured mount points, IAM permissions, and agent status. |
| Nginx logs missing | Check configured file paths, file permissions, log group and stream names, and agent status. |
| Run Command fails | Check Systems Manager connectivity, SSM Agent status, the instance IAM role, and command output. |
| Alarm does not trigger | Check the metric, threshold, evaluation period, datapoints to alarm, and alarm state. |
| Email notification not received | Verify the SNS subscription is confirmed, the alarm action points to the correct topic, and notification delivery is tested. |

## Security and Cost Considerations

- Follow least-privilege IAM practices.
- Do not store credentials or secrets in monitoring configuration.
- Configure appropriate CloudWatch log retention periods.
- Monitor custom metric, log ingestion, dashboard, and alarm charges.
- Remove unused lab resources when testing is complete.

## Key Learnings

- CloudWatch provides metrics, dashboards, alarms, and centralized logs.
- The CloudWatch Agent collects guest operating-system metrics that EC2 basic monitoring does not provide by default.
- Systems Manager Run Command enables remote software installation and configuration.
- Parameter Store centralizes the CloudWatch Agent configuration.
- CloudWatch alarms can invoke SNS actions when their configured state transitions occur.
- Missing metrics, failed commands, and undelivered notifications require separate troubleshooting.
- Configuring an alarm and SNS destination is not the same as validating end-to-end notification delivery.

## Conclusion

This lab demonstrates centralized EC2 observability using CloudWatch metrics, dashboards, alarms, and logs, with Systems Manager for remote configuration and SNS for notification integration.

The lab provides a foundation for investigating resource utilization, instance health, and application-level issues in an AWS CloudOps environment.
