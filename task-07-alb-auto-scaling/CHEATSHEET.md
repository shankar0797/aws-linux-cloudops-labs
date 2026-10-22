\# Task 7: ALB and Auto Scaling Cheatsheet



\*\*Region:\*\* `ap-south-1`



\## Resource Names



| Resource | Name / ID |

|---|---|

| VPC | `vpc-01e04109fea01ca2f` |

| ALB | `task7-nginx-alb` |

| Target group | `task7-nginx-tg` |

| ASG | `task7-nginx-asg` |

| Launch template | `task7-nginx-template` |

| Launch template ID | `lt-0dc54d191a9dd384a` |

| ALB security group | `sg-0f7f78709ea2a3ffa` |

| EC2 security group | `sg-0a40c9c7532d43be2` |



\## 1. Check the ALB



```bash

aws elbv2 describe-load-balancers \\

&#x20; --names task7-nginx-alb \\

&#x20; --region ap-south-1 \\

&#x20; --output table

```



Get the DNS name:



```bash

aws elbv2 describe-load-balancers \\

&#x20; --names task7-nginx-alb \\

&#x20; --region ap-south-1 \\

&#x20; --query 'LoadBalancers\[0].DNSName' \\

&#x20; --output text

```



Test the application:



```bash

curl -i http://<ALB-DNS-NAME>

```



Expected response: `HTTP 200 OK`.



\## 2. Check Target Health



```bash

aws elbv2 describe-target-health \\

&#x20; --target-group-arn arn:aws:elasticloadbalancing:ap-south-1:683604924610:targetgroup/task7-nginx-tg/31180be68a0da4e8 \\

&#x20; --region ap-south-1 \\

&#x20; --output table

```



Expected target state: `healthy`.



Common states:



\- `initial`: Initial registration or health checks are in progress.

\- `healthy`: The target passes health checks.

\- `unhealthy`: The target fails health checks.

\- `unused`: The target is not currently being used by the load balancer, or the target group is not configured for traffic.



\## 3. Check the Auto Scaling Group



```bash

aws autoscaling describe-auto-scaling-groups \\

&#x20; --auto-scaling-group-names task7-nginx-asg \\

&#x20; --region ap-south-1 \\

&#x20; --output table

```



Check capacity and instance states:



```bash

aws autoscaling describe-auto-scaling-groups \\

&#x20; --auto-scaling-group-names task7-nginx-asg \\

&#x20; --region ap-south-1 \\

&#x20; --query 'AutoScalingGroups\[0].{Min:MinSize,Desired:DesiredCapacity,Max:MaxSize,Instances:Instances\[].{ID:InstanceId,Health:HealthStatus,State:LifecycleState}}' \\

&#x20; --output json

```



Configured capacity:



\- Minimum: `2`

\- Desired: `2`

\- Maximum: `4`



\## 4. Check Scaling Policies



```bash

aws autoscaling describe-policies \\

&#x20; --auto-scaling-group-name task7-nginx-asg \\

&#x20; --region ap-south-1 \\

&#x20; --output table

```



Policy configuration:



\- Type: `TargetTrackingScaling`

\- Metric: `ASGAverageCPUUtilization`

\- Target: `50%`



A configured target is not proof that a scaling event has occurred. Verify scaling activities and instance counts to confirm actual scale-out or scale-in behavior.



\## 5. Check Scaling Activities



```bash

aws autoscaling describe-scaling-activities \\

&#x20; --auto-scaling-group-name task7-nginx-asg \\

&#x20; --region ap-south-1 \\

&#x20; --max-items 10 \\

&#x20; --output table

```



Use this command to review recent scaling decisions, launches, terminations, and failures.



\## 6. Check the Launch Template



```bash

aws ec2 describe-launch-templates \\

&#x20; --launch-template-names task7-nginx-template \\

&#x20; --region ap-south-1 \\

&#x20; --output table

```



\## 7. Troubleshooting Quick Reference



\### ALB returns 503



Check:



1\. Target group has registered targets.

2\. Targets are healthy.

3\. ALB listener forwards to the correct target group.

4\. EC2 security group permits port 80 from the ALB security group.

5\. Nginx is installed and running.



\### Targets remain unhealthy



Check:



1\. Target group health check path is `/`.

2\. Nginx is running on port 80.

3\. The EC2 security group permits inbound HTTP from the ALB security group.

4\. The launch template user data completed successfully.

5\. The instance has completed startup and health checks.



\### ASG does not scale out



Check:



1\. The target-tracking policy is enabled.

2\. CPU utilization is sufficiently above target for long enough to trigger scaling.

3\. The ASG has not reached maximum capacity.

4\. CloudWatch metrics and scaling activities show no errors.

5\. EC2 capacity and launch permissions are available.



\## 8. Important Concepts



\- \*\*ALB:\*\* Distributes incoming HTTP/HTTPS requests to registered targets.

\- \*\*Target group:\*\* Defines the backend targets, protocol, port, and health checks.

\- \*\*Launch template:\*\* Defines the AMI, instance type, security groups, and startup configuration.

\- \*\*ASG:\*\* Maintains the desired number of EC2 instances and replaces unhealthy instances.

\- \*\*Target tracking:\*\* Adjusts capacity to maintain a metric near a configured target.

\- \*\*Health check grace period:\*\* Gives new instances time to initialize before ASG health-check replacement decisions.

\- \*\*Multi-AZ deployment:\*\* Helps maintain availability when an Availability Zone has a disruption.



\## Cleanup Reminder



AWS resources can continue generating charges. Review the ALB, ASG, and EC2 resources when finished. Follow the lab README's cleanup order to avoid dependencies and accidental deletion of resources belonging to other projects.



