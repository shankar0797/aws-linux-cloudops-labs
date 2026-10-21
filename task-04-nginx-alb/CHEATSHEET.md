# Task 4 — Nginx Behind an ALB Cheatsheet

## Lab resources
- Region: `ap-south-1`
- VPC: `vpc-01e04109fea01ca2f`
- EC2: `i-06a150e7bc6dd5ed6` (`Develop-machine`)
- ALB: `task4-nginx-alb`
- ALB SG: `task4-alb-sg` / `sg-02e803ee1fc3163ca`
- Target group: `task4-nginx-tg`
- Listener: HTTP :80
- Target: HTTP :80
- Health check path: `/`
- ALB DNS: `task4-nginx-alb-2029992009.ap-south-1.elb.amazonaws.com`

## Verify Nginx on EC2
```bash
sudo systemctl status nginx --no-pager
curl -I http://localhost/
hostname
```
Expected local response: `HTTP/1.1 200 OK`.

## Traffic flow
```text
Browser -> ALB listener HTTP:80 -> target group HTTP:80 -> EC2 Nginx
```

## Component roles
- **ALB:** public entry point; routes requests to targets.
- **Listener:** accepts connections on a configured protocol/port and applies rules.
- **Target group:** defines backend targets, protocol/port, and health checks.
- **Health check:** probes path `/`; expected success code is `200`.

## Recommended security groups
- **ALB SG:** inbound TCP 80 from `0.0.0.0/0` for this public HTTP lab.
- **EC2 SG:** inbound TCP 80 from ALB SG `sg-02e803ee1fc3163ca`; SSH TCP 22 from a trusted admin IP only.

## Troubleshooting
1. `sudo systemctl status nginx --no-pager`
2. `curl -I http://localhost/`
3. Confirm target registered on HTTP port 80.
4. Confirm health check path `/` and success code `200`.
5. Confirm EC2 inbound TCP 80 allows the ALB security group.
6. Read target **Health status details** in the AWS Console.
7. Inspect logs:
   ```bash
   sudo tail -n 50 /var/log/nginx/access.log
   sudo tail -n 50 /var/log/nginx/error.log
   ```

## Verification results
- ALB: Active, internet-facing, two Availability Zones.
- Target group: 1 Healthy target.
- Browser: custom Nginx page loads through ALB DNS.
- Listener: HTTP :80 forwards to `task4-nginx-tg`.

**Security reminder:** The initial EC2 SG allowed HTTP 80 and SSH 22 from `0.0.0.0/0`. Verify hardening before stating that direct access has been restricted.

## Cost note
An ALB incurs charges while running. Remove only resources created for this lab after verifying dependencies; do not delete the reused EC2 instance or unrelated storage.
