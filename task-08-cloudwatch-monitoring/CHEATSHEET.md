# Task 8: CloudWatch Monitoring and SNS Cheatsheet

## AWS Environment

- **Region:** `ap-south-1`
- **EC2 instance 1:** `i-0b004c0e7a2c1c095`
- **EC2 instance 2:** `i-0cc6d76fc75033ad7`
- **Dashboard:** `task8-operations-dashboard`
- **Custom metric namespace:** `Task8/EC2`
- **Parameter Store parameter:** `AmazonCloudWatch-agent-task8-config`
- **Instance role:** `Task8CloudWatchAgentRole`

## 1. Verify the AWS Identity

```bash
aws sts get-caller-identity
```

Confirms the active AWS account and identity.

## 2. Check EC2 Instances

```bash
aws ec2 describe-instances \
  --region ap-south-1 \
  --instance-ids \
    i-0b004c0e7a2c1c095 \
    i-0cc6d76fc75033ad7 \
  --query 'Reservations[].Instances[].{ID:InstanceId,State:State.Name,PrivateIP:PrivateIpAddress}' \
  --output table
```

## 3. Check the CloudWatch Agent Package

Run this command on each EC2 instance:

```bash
sudo systemctl status amazon-cloudwatch-agent --no-pager
```

Restart the agent if required:

```bash
sudo systemctl restart amazon-cloudwatch-agent
```

Inspect agent logs:

```bash
sudo tail -n 100 /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

The service status and logs help identify configuration, startup, and publishing issues.

## 4. Check the Parameter Store Configuration

```bash
aws ssm get-parameter \
  --region ap-south-1 \
  --name AmazonCloudWatch-agent-task8-config \
  --query 'Parameter.{Name:Name,Type:Type,Version:Version}' \
  --output table
```

This checks the parameter metadata without printing its value.

To inspect the configuration when needed:

```bash
aws ssm get-parameter \
  --region ap-south-1 \
  --name AmazonCloudWatch-agent-task8-config \
  --with-decryption \
  --query 'Parameter.Value' \
  --output text
```

Review the output carefully before sharing it. Do not expose credentials or other secrets if the parameter contains them.

## 5. Check Systems Manager Run Command

List recent commands:

```bash
aws ssm list-commands \
  --region ap-south-1 \
  --max-results 10 \
  --output table
```

Check invocation details for a known command ID and instance:

```bash
aws ssm get-command-invocation \
  --region ap-south-1 \
  --command-id COMMAND_ID \
  --instance-id i-0b004c0e7a2c1c095
```

Replace `COMMAND_ID` with the actual command ID.

Common statuses include `Pending`, `InProgress`, `Success`, `Failed`, and `TimedOut`.

## 6. Verify Custom Memory and Disk Metrics

List available metrics in the custom namespace:

```bash
aws cloudwatch list-metrics \
  --region ap-south-1 \
  --namespace Task8/EC2 \
  --output table
```

Filter for the memory metric:

```bash
aws cloudwatch list-metrics \
  --region ap-south-1 \
  --namespace Task8/EC2 \
  --metric-name mem_used_percent \
  --output table
```

Filter for the disk metric:

```bash
aws cloudwatch list-metrics \
  --region ap-south-1 \
  --namespace Task8/EC2 \
  --metric-name disk_used_percent \
  --output table
```

**Note:** `list-metrics` only lists metrics CloudWatch recognizes as available. Newly published metrics can take time to appear, and metrics without recent data may not be listed indefinitely.

## 7. Check Nginx Log Groups

```bash
aws logs describe-log-groups \
  --region ap-south-1 \
  --log-group-name-prefix /task8/nginx/ \
  --query 'logGroups[].{Name:logGroupName,Retention:retentionInDays}' \
  --output table
```

Expected log groups:

- `/task8/nginx/access`
- `/task8/nginx/error`

List log streams for the access log group:

```bash
aws logs describe-log-streams \
  --region ap-south-1 \
  --log-group-name /task8/nginx/access \
  --order-by LastEventTime \
  --descending \
  --max-items 10 \
  --output table
```

Read recent access log events from a known stream:

```bash
aws logs get-log-events \
  --region ap-south-1 \
  --log-group-name /task8/nginx/access \
  --log-stream-name STREAM_NAME \
  --limit 20
```

Replace `STREAM_NAME` with an actual stream name returned by `describe-log-streams`.

## 8. Check the CloudWatch Dashboard

```bash
aws cloudwatch get-dashboard \
  --region ap-south-1 \
  --dashboard-name task8-operations-dashboard
```

This returns the dashboard body and widget configuration.

## 9. Check CloudWatch Alarms

List alarms:

```bash
aws cloudwatch describe-alarms \
  --region ap-south-1 \
  --query 'MetricAlarms[].{Name:AlarmName,State:StateValue,Metric:MetricName,Namespace:Namespace}' \
  --output table
```

Inspect one alarm:

```bash
aws cloudwatch describe-alarms \
  --region ap-south-1 \
  --alarm-names ALARM_NAME \
  --output json
```

Replace `ALARM_NAME` with the actual alarm name.

For instance status checks, the metric is `StatusCheckFailed`. The `Maximum` statistic and threshold of `1` can be used to detect a failed status check, depending on the alarm's configured evaluation settings.

## 10. Check SNS Topics and Subscriptions

List SNS topics:

```bash
aws sns list-topics \
  --region ap-south-1 \
  --output table
```

List subscriptions:

```bash
aws sns list-subscriptions \
  --region ap-south-1 \
  --output table
```

Inspect an individual topic:

```bash
aws sns get-topic-attributes \
  --region ap-south-1 \
  --topic-arn TOPIC_ARN
```

Replace `TOPIC_ARN` with the actual topic ARN.

For email subscriptions, confirm that the subscription status is `Confirmed`. A configured topic or alarm action alone does not prove that email delivery works.

## 11. Troubleshooting Quick Reference

### Memory or disk metrics are missing

1. Confirm the agent service is running.
2. Inspect the agent configuration and logs.
3. Verify the metric names and namespace.
4. Check instance dimensions and collection settings.
5. Confirm the instance role permits CloudWatch metric publishing.
6. Allow time for newly published metrics to appear.

### Nginx logs are missing

1. Confirm Nginx is writing to the configured log files.
2. Check the file paths and permissions.
3. Verify the agent log collection configuration.
4. Check the agent logs for file or publishing errors.
5. Confirm the log groups and streams exist in the expected Region.

### Systems Manager command fails

1. Verify the instance is online in Systems Manager.
2. Check SSM Agent status.
3. Verify the instance role includes the required permissions.
4. Confirm network connectivity to required Systems Manager endpoints.
5. Inspect the command output and invocation status.

### CloudWatch alarm does not trigger

1. Confirm the metric exists and is reporting data.
2. Verify the namespace, metric name, and dimensions.
3. Check the threshold, statistic, evaluation period, and datapoints to alarm.
4. Review the current alarm state and its history.
5. Confirm the alarm action is configured for the intended SNS topic.

### SNS email notification is not received

1. Verify the SNS topic ARN configured in the alarm.
2. Confirm the email subscription is `Confirmed`.
3. Check the alarm action configuration.
4. Test SNS delivery using an authorized test publication when appropriate.
5. Review SNS delivery status and available monitoring information.

## 12. Important Concepts

- **CloudWatch metrics:** Numerical measurements used to monitor resource and application behavior.
- **CloudWatch Agent:** Collects guest OS metrics and log files.
- **CloudWatch Logs:** Centralizes log events from configured sources.
- **CloudWatch dashboard:** Presents monitoring data through configurable widgets.
- **CloudWatch alarm:** Evaluates a metric against configured conditions and changes state accordingly.
- **Systems Manager Run Command:** Runs supported commands remotely on managed instances.
- **Parameter Store:** Stores configuration values centrally.
- **SNS:** Distributes messages to configured subscribers and endpoints.

## Interview Summary

**Why install the CloudWatch Agent?**

To collect guest operating-system metrics such as memory and disk utilization, along with application logs that are not provided by standard EC2 metrics alone.

**How do you troubleshoot missing CloudWatch metrics?**

Check the agent service, configuration, logs, IAM permissions, metric namespace, dimensions, and collection interval.

**What is the difference between a metric and an alarm?**

A metric is a measurement. An alarm evaluates a metric against configured thresholds and evaluation rules and can invoke actions when its state changes.

**Does configuring SNS prove email delivery?**

No. The subscription must be confirmed, the alarm must be configured with the correct action, and notification delivery should be tested.

## Cleanup Reminder

Review CloudWatch dashboards, alarms, log groups, SNS topics, Parameter Store parameters, and any lab-specific resources before cleanup. Confirm that resources are not shared with other workloads, and retain logs or screenshots needed for your portfolio.
