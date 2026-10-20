# Task 2 Cheat Sheet — Nginx on AWS EC2

## Goal
Install Nginx on Amazon Linux 2023, serve a custom page with the engineer's name, environment, and hostname, enable Nginx at boot, and verify local and external access.

## 1. Identify OS and Nginx
```bash
cat /etc/os-release
nginx -v
```

## 2. Install Nginx
```bash
sudo dnf update -y
sudo dnf install nginx -y
```
`sudo` = administrative privileges; `dnf` = package manager; `-y` = automatically confirm prompts.

## 3. Start, enable, and check service
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
sudo systemctl is-enabled nginx
sudo systemctl is-active nginx
```
`start` starts now; `enable` configures boot startup; `status` shows service state. Expected quick checks: `enabled`, `active`.

## 4. Verify local HTTP
```bash
nginx -v
curl -I http://localhost
curl http://localhost
```
`curl -I` shows response headers; `curl` displays the page body. Expected HTTP response: `200 OK`.

## 5. Document root and backup
```bash
sudo ls -l /usr/share/nginx/html/
sudo cat /usr/share/nginx/html/index.html
sudo cp /usr/share/nginx/html/index.html /usr/share/nginx/html/index.html.backup
sudo nano /usr/share/nginx/html/index.html
```
The document root in this lab is `/usr/share/nginx/html/`. In Nano save with `Ctrl+O`, `Enter`, then exit with `Ctrl+X`. A static HTML edit does not normally require an Nginx restart.

Page requirements: engineer name, environment (`AWS EC2 - Amazon Linux 2023`), actual hostname from `hostname`, web server and deployment status.

## 6. Security group
AWS Console → EC2 → Instances → select instance → Security → security group → Inbound rules.
- HTTP: TCP 80, allow intended clients.
- SSH: TCP 22, restrict to trusted IP/approved access.
- Remove unneeded inbound rules.
`0.0.0.0/0` means all IPv4 addresses; use only when public access is intended.

## 7. Browser test
Open `http://PUBLIC-IP` using the current EC2 public IPv4 address. “Not secure” is expected because this lab uses HTTP and does not configure HTTPS/TLS.

## 8. Reboot test
```bash
sudo systemctl is-enabled nginx
sudo systemctl is-active nginx
sudo reboot
```
Reconnect, then run:
```bash
sudo systemctl is-enabled nginx
sudo systemctl is-active nginx
sudo systemctl status nginx
curl -I http://localhost
```
Refresh the browser too. Reboot only when safe to interrupt the lab.

## 9. Interview concept
- **Active:** systemd reports Nginx is running.
- **Local response:** `curl -I http://localhost` gets a response from inside EC2.
- **External reachability:** a client can reach `http://PUBLIC-IP`. This depends on IP, routes, security group, network ACLs, host firewall, and Nginx configuration.

Nginx can be active while the website is unreachable if port 80 is blocked.

## 10. Troubleshooting
| Problem | First checks |
|---|---|
| Nginx not installed | `sudo dnf install nginx -y` |
| Service failed | `sudo systemctl status nginx`; `sudo journalctl -u nginx --no-pager -n 50` |
| Config issue | `sudo nginx -t` |
| Browser timeout | Public IP, inbound TCP 80, routes, network ACLs, host firewall |
| Old page | Refresh and inspect `index.html` |
| Not active after reboot | `systemctl is-enabled`, `systemctl status`, service logs |

## Screenshot checklist
1. `01-os-identification.png`
2. `02-nginx-service-running.png`
3. `03-nginx-browser-access.png`
4. `04-custom-website.png`
5. `05-nginx-after-reboot.png`
6. `06-security-group-rules.png`

Use real screenshots only. Never commit credentials, private keys, or secrets.
