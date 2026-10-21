# Task 4: Host Nginx Behind an Application Load Balancer

## Objective
Expose the existing Nginx website through an AWS Application Load Balancer (ALB), rather than relying on direct public access to the EC2 web server.

## Architecture
```text
Browser / Users
      | HTTP :80
      v
Internet-facing ALB: task4-nginx-alb
Security group: task4-alb-sg
      | Listener HTTP :80
      v
Target group: task4-nginx-tg
Protocol: HTTP :80; health check path /
      |
      v
EC2: Develop-machine (i-06a150e7bc6dd5ed6)
Nginx on HTTP :80
```

## Environment and resources
- Region: `ap-south-1` (Mumbai)
- VPC: `vpc-01e04109fea01ca2f`
- EC2 instance: `i-06a150e7bc6dd5ed6` (`Develop-machine`)
- Linux hostname: `ip-172-31-14-0.ap-south-1.compute.internal`
- ALB: `task4-nginx-alb`
- ALB security group: `task4-alb-sg` (`sg-02e803ee1fc3163ca`)
- Target group: `task4-nginx-tg`
- Listener: HTTP port 80
- Target protocol/port: HTTP port 80
- Health check path: `/`
- Browser-tested ALB DNS: `task4-nginx-alb-2029992009.ap-south-1.elb.amazonaws.com`

This task reused the Nginx installation and custom web page from Task 2.

## Procedure

### 1. Verify existing Nginx
Run on the EC2 Linux terminal:
```bash
sudo systemctl status nginx --no-pager
curl -I http://localhost/
hostname
```
Observed: Nginx was `active (running)` and enabled at boot; `curl` returned `HTTP/1.1 200 OK`; hostname was `ip-172-31-14-0.ap-south-1.compute.internal`.

### 2. Create the ALB security group
Created `task4-alb-sg` (`sg-02e803ee1fc3163ca`) in VPC `vpc-01e04109fea01ca2f`.
Inbound rule: HTTP/TCP/port 80 from `0.0.0.0/0`, allowing public users to reach the internet-facing ALB.

### 3. Create the target group
Created `task4-nginx-tg`:
- Target type: Instances
- Protocol: HTTP
- Port: 80
- Protocol version: HTTP1
- Health check: HTTP path `/`
- Success code configured as `200`

Registered EC2 instance `i-06a150e7bc6dd5ed6` on port 80. Before the ALB was associated, the target showed `Unused`; this was expected because no load balancer was associated at that stage.

### 4. Create the Application Load Balancer
Created `task4-nginx-alb`:
- Type: Application Load Balancer
- Scheme: Internet-facing
- IP address type: IPv4
- VPC: `vpc-01e04109fea01ca2f`
- Two Availability Zones
- Security group: `task4-alb-sg`

Configured an HTTP listener on port 80 forwarding to `task4-nginx-tg`.

### 5. Verify target health and website
After associating the target group with the ALB, the target became `Healthy` (1 healthy target, 0 unhealthy, 0 unused).

Opened the ALB DNS name in a browser:
`http://task4-nginx-alb-2029992009.ap-south-1.elb.amazonaws.com`

The custom Nginx page loaded and showed the engineer name, AWS EC2/Amazon Linux 2023 environment, instance hostname, Nginx, and successful deployment status. This proves a request through the ALB reached the EC2 web server.

## Security group design
Recommended final inbound rules:
- **ALB SG (`task4-alb-sg`):** HTTP/TCP/80 from `0.0.0.0/0` for this public HTTP lab.
- **EC2 SG (`launch-wizard-4`):** HTTP/TCP/80 from ALB SG `sg-02e803ee1fc3163ca`; SSH/TCP/22 from a trusted administrator IP only.

The initial EC2 security group screenshot showed HTTP 80 and SSH 22 open to `0.0.0.0/0`. Verify the final rules are hardened before claiming this step is complete.

## Component roles
- **ALB:** stable public entry point that routes requests to targets.
- **Listener:** accepts connections on a configured protocol and port and applies routing rules.
- **Target group:** defines backend targets, protocol/port, and health checks.
- **Health check:** periodically requests the configured path. A successful response matching the success code (here, `200`) makes the target eligible to receive traffic.

## Troubleshooting an unhealthy target
1. Check Nginx:
   ```bash
   sudo systemctl status nginx --no-pager
   ```
2. Check local HTTP response:
   ```bash
   curl -I http://localhost/
   ```
3. Verify target protocol/port is HTTP/80 and health-check path is `/`.
4. Verify EC2 inbound TCP/80 permits traffic from the ALB security group.
5. Confirm target registration and VPC alignment.
6. Read the target's **Health status details** in the AWS Console.
7. Inspect Nginx logs if needed:
   ```bash
   sudo tail -n 50 /var/log/nginx/access.log
   sudo tail -n 50 /var/log/nginx/error.log
   ```

## Evidence screenshots
Save actual screenshots in `screenshots/`:
1. `01-alb-active.png`
2. `02-target-healthy.png`
3. `03-website-via-alb.png`
4. `04-listener-and-target-group.png`
5. `05-security-group-rules.png`

If needed, capture ALB and EC2 security groups separately as `05a-alb-security-group.png` and `05b-ec2-security-group.png`.

## Interview questions
1. **Why use an ALB instead of exposing EC2 directly?** It provides a stable entry point, listener-based routing, health checks, and can distribute requests across multiple targets.
2. **What is a listener?** It accepts connections on a protocol/port and applies routing rules.
3. **What is a target group?** It defines backend targets, protocol/port, and health-check settings.
4. **What does Healthy mean?** The target passes its configured health checks and can receive routed traffic.
5. **Why was the target Unused before ALB association?** The target group wasn't associated with a load balancer/listener yet.
6. **How do you troubleshoot Unhealthy?** Check Nginx, local HTTP response, target port, health-check path/success codes, security groups, and health-status details.
7. **Why allow backend HTTP only from the ALB security group?** It prevents arbitrary direct internet traffic to the EC2 web port while allowing the ALB to reach the target.

## Cost and cleanup
ALBs incur charges while running. When finished, delete only lab-created resources after checking dependencies. Remove the ALB first, then the target group and dedicated ALB security group once unused. Do not delete the reused EC2 instance or unrelated storage.

## Learning outcome
Reused an existing Nginx web server, placed an internet-facing ALB in front of it, configured an HTTP listener and target group health checks, and verified that the target became healthy and the website loaded through the ALB DNS name.
