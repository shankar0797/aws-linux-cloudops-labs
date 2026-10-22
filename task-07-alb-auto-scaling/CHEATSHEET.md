# Task 7: ALB and Auto Scaling Cheatsheet

**AWS Region:** `ap-south-1` (Mumbai)

## Resource Names

| Resource | Name / ID |
|---|---|
| VPC | `vpc-01e04109fea01ca2f` |
| ALB | `task7-nginx-alb` |
| Target group | `task7-nginx-tg` |
| Auto Scaling Group | `task7-nginx-asg` |
| Launch template | `task7-nginx-template` |
| Launch template ID | `lt-0dc54d191a9dd384a` |
| ALB security group | `sg-0f7f78709ea2a3ffa` |
| EC2 security group | `sg-0a40c9c7532d43be2` |

## 1. Check the ALB

```bash
aws elbv2 describe-load-balancers \
  --names task7-nginx-alb \
  --region ap-south-1 \
  --output table
```

Get the ALB DNS name:

```bash
ALB_DNS=$(aws elbv2 describe-load-balancers \
  --names task7-nginx-alb \
  --region ap-south-1 \
  --query 'LoadBalancers[0].DNSName' \
  --output text)

echo "$ALB_DNS"
```

Test the application:

```bash
curl -i --max-time 15 "http://$ALB_DNS"
```

**Expected:** HTTP `200 OK` and the custom Task 7 HTML page.

## 2. Check Target Health

```bash
aws elbv2 describe-target-health \
  --target-group-arn arn:aws:elasticloadbalancing:ap-south-1:683604924610:targetgroup/task7-nginx-tg/31180be68a0da4e8 \
  --region ap-south-1 \
  --output table
```

**Expected:** Both registered EC2 targets show `healthy`.

Common target states:

- `initial`: Registration or initial health checks are in progress.
- `healthy`: The target passes health checks.
- `unhealthy`: The target fails health checks.
- `unused`: The target is not currently being used by the load balancer, for example because the target group is not associated with an active listener or the target is in an unavailable state.

## 3. Check the Auto Scaling Group

```bash
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names task7-nginx-asg \
  --region ap-south-1 \
  --output table
```

Check capacity and instance states:

```bash
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names task7-nginx-asg \
  --region ap-south-1 \
  --query 'AutoScalingGroups[0].{Min:MinSize,Desired:DesiredCapacity,Max:MaxSize,Instances:Instances[].{ID:InstanceId,Health:HealthStatus,State:LifecycleState}}' \
  --output json
```

Configured capacity:

- Minimum: `2`
- Desired: `2`
- Maximum: `4`

The `HealthStatus` and `LifecycleState` fields provide different information. For example, an instance can have an `InService` lifecycle state while its health is still being evaluated.

## 4. Check Scaling Policies

```bash
aws autoscaling describe-policies \
  --auto-scaling-group-name task7-nginx-asg \
  --region ap-south-1 \
  --output table
```

The configured target-tracking policy uses:

- Policy type: `TargetTrackingScaling`
- Metric: `ASGAverageCPUUtilization`
- Target: `50%`
- Policy name: `task7-cpu-target-tracking`

A configured target does not prove that a scaling event has occurred. Review scaling activities and instance counts to confirm actual behavior.

## 5. Check Scaling Activities

```bash
aws autoscaling describe-scaling-activities \
  --auto-scaling-group-name task7-nginx-asg \
  --region ap-south-1 \
  --max-items 10 \
  --output table
```

Use this command to review recent scaling decisions, instance launches, terminations, and failures.

## 6. Check the Launch Template

```bash
aws ec2 describe-launch-templates \
  --launch-template-names task7-nginx-template \
  --region ap-south-1 \
  --output table
```

To inspect launch-template version 1:

```bash
aws ec2 describe-launch-template-versions \
  --launch-template-id lt-0dc54d191a9dd384a \
  --versions 1 \
  --region ap-south-1 \
  --output json
```

## 7. Troubleshooting Quick Reference

### ALB returns 503 Service Unavailable

Check:

1. The target group has registered targets.
2. At least one target is healthy.
3. The ALB listener forwards to the correct target group.
4. The EC2 security group allows port 80 from the ALB security group.
5. Nginx is installed and running on the EC2 instances.
6. The target group port and health check configuration are correct.

### Targets remain unhealthy

Check:

1. The health check path is `/`.
2. Nginx is running and listening on port 80.
3. The EC2 security group allows inbound HTTP from the ALB security group.
4. The launch-template user data completed successfully.
5. The instance has completed startup and can return HTTP 200 locally.
6. The target group is configured for the correct port and protocol.

Useful commands to run on an instance:

```bash
sudo systemctl status nginx --no-pager
curl -I http://localhost
sudo journalctl -u nginx --no-pager -n 50
```

### ASG does not scale out

Check:

1. The target-tracking policy is attached to the correct ASG.
2. The metric is reporting CPU utilization.
3. CPU utilization and metric history are sufficient to trigger scaling.
4. The ASG has not reached its maximum capacity.
5. Scaling activities and CloudWatch metrics show no relevant errors.
6. EC2 capacity, subnet IP availability, and launch permissions are sufficient.

Do not expect an immediate launch whenever CPU briefly exceeds 50%. Target tracking evaluates metrics over time and manages scaling according to its policy behavior.

## 8. Important Concepts

- **ALB:** Distributes incoming HTTP/HTTPS requests across registered targets.
- **Target group:** Defines backend targets, protocol, port, and health checks.
- **Launch template:** Defines EC2 configuration such as AMI, instance type, security groups, and user data.
- **ASG:** Maintains desired capacity and can replace unhealthy instances.
- **Target tracking:** Adjusts capacity to keep a selected metric near a configured target.
- **Health check grace period:** Gives new instances time to initialize before ASG health-check replacement decisions.
- **Multi-AZ deployment:** Improves resilience to an Availability Zone disruption when capacity and dependencies are appropriately distributed.

## Cleanup Reminder

AWS resources can continue generating charges. Review the ALB, ASG, EC2 instances, and related resources when the lab is complete.

Before cleanup, verify that each resource belongs exclusively to this lab. Follow the cleanup order in the README to avoid dependency errors or accidental deletion of resources used by other projects.

## Interview Summary

**How does an ALB distribute requests?**

The ALB accepts requests through a listener and forwards them to a configured target group. It routes traffic to eligible healthy targets according to the listener rules and load-balancing behavior.

**What happens when an EC2 instance becomes unhealthy?**

The target group marks it unhealthy when it fails health checks. The ALB stops routing normal requests to that unhealthy target. If the ASG's applicable health checks identify the instance as unhealthy, the ASG can replace it.

**How does CPU target tracking work?**

The ASG adjusts capacity to keep average CPU utilization near the configured target, subject to the minimum and maximum capacity limits.

**Did this lab prove automatic scaling?**

The target-tracking policy was configured and verified, but a real scale-out or scale-in event under load was not tested. The demonstrated results are successful load balancing across two healthy instances and verification of the scaling configuration.
