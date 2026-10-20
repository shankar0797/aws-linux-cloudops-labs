# Task 2: Deploy an Nginx Web Server on Amazon Linux EC2

## Objective
Deploy Nginx on a Linux EC2 instance, serve a custom HTML page showing the engineer's name, environment, and instance hostname, enable Nginx at boot, verify with curl and a browser, and restrict inbound network access to required ports.

## Lab environment
- AWS EC2, Amazon Linux 2023
- Region: `ap-south-1` (Mumbai)
- Nginx; HTTP/TCP port 80
- Document root: `/usr/share/nginx/html`
- Page: `/usr/share/nginx/html/index.html`
- Hostname observed: `ip-172-31-14-0.ap-south-1.compute.internal`
- Public IP used in the lab: `13.235.77.113` (check the current address in the EC2 console before reuse)

## Steps performed

### 1. Identify OS and check Nginx
```bash
cat /etc/os-release
nginx -v
```
The OS check identified Amazon Linux 2023. Initially, `nginx -v` returned `command not found`, indicating Nginx was not installed.

### 2. Update packages and install Nginx
```bash
sudo dnf update -y
sudo dnf install nginx -y
```
`sudo` runs with administrative privileges, `dnf` is the Amazon Linux 2023 package manager, `update` installs available updates, `install nginx` installs Nginx, and `-y` automatically confirms prompts. In production, review updates and follow the approved patching process.

### 3. Start and enable Nginx
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```
`start` runs the service now; `enable` configures it to start during boot; `status` displays its state and recent logs. The lab output showed `Active: active (running)` and the service was enabled.

### 4. Verify locally
```bash
nginx -v
curl -I http://localhost
```
The lab returned `HTTP/1.1 200 OK` with `Server: nginx/1.30.4`. `curl -I` displays response headers. A successful local response does not by itself prove that external clients can reach the server.

### 5. Configure the EC2 security group
In AWS Console, open **EC2 → Instances → select the instance → Security → security group → Inbound rules**.
- Allow HTTP, TCP port 80, for the intended clients.
- Restrict SSH, TCP port 22, to a trusted client IP or approved access method.
- Remove unnecessary inbound rules.

The public browser test reached the default Nginx welcome page at `13.235.77.113`. `0.0.0.0/0` allows HTTP from any IPv4 address and should be used only when public access is intended. Do not expose SSH to the whole internet without an approved reason.

### 6. Inspect and back up the default page
```bash
sudo ls -l /usr/share/nginx/html/
sudo cat /usr/share/nginx/html/index.html
sudo cp /usr/share/nginx/html/index.html /usr/share/nginx/html/index.html.backup
```
The document root contained `index.html`, `404.html`, `50x.html`, and supporting image files. The original page was the standard Nginx welcome page.

### 7. Create the custom HTML page
```bash
sudo nano /usr/share/nginx/html/index.html
```
The page used in the lab:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EC2 Nginx Deployment</title>
</head>
<body>
    <h1>Linux EC2 Web Server</h1>
    <h2>Deployment Details</h2>
    <p><strong>Engineer Name:</strong> Shiva Shankar</p>
    <p><strong>Environment:</strong> AWS EC2 - Amazon Linux 2023</p>
    <p><strong>Instance Hostname:</strong> ip-172-31-14-0.ap-south-1.compute.internal</p>
    <p><strong>Web Server:</strong> Nginx</p>
    <p><strong>Status:</strong> Successfully Deployed</p>
</body>
</html>
```
The hostname was observed from this EC2 instance. For another instance, run `hostname` and use the actual output. In Nano, save with `Ctrl+O`, press `Enter`, and exit with `Ctrl+X`. A static HTML update does not normally require restarting Nginx.

### 8. Verify the custom page
```bash
hostname
curl http://localhost
curl -I http://localhost
```
`hostname` prints the instance hostname, `curl` displays the HTML body, and `curl -I` checks response headers. Then open `http://PUBLIC-IP` in a browser using the current public IPv4 address. The custom page displayed the engineer's name, environment, hostname, web server, and deployment status.

### 9. Verify after reboot
Before reboot:
```bash
sudo systemctl is-enabled nginx
sudo systemctl is-active nginx
```
Expected: `enabled` and `active`.

Reboot only when it is safe to interrupt the lab instance:
```bash
sudo reboot
```
Reconnect and run:
```bash
sudo systemctl is-enabled nginx
sudo systemctl is-active nginx
sudo systemctl status nginx
curl -I http://localhost
```
Expected: Nginx remains enabled, becomes active after boot, and responds with HTTP `200 OK`. Refresh the browser page too. Ensure you actually capture post-reboot evidence before claiming that this check is verified.

## Active service vs. network reachability
- **Service active:** `systemctl is-active nginx` reports whether systemd considers Nginx running.
- **Local HTTP response:** `curl -I http://localhost` checks HTTP response from inside the EC2 instance.
- **External reachability:** opening `http://PUBLIC-IP` tests the client-to-instance network path. This also depends on the current public IP, routes, security group, network ACLs, host firewall, and Nginx listener/configuration.

Nginx can be active while the website is unreachable externally if inbound TCP port 80 is blocked. A local HTTP 200 response does not prove external reachability.

## Troubleshooting
| Symptom | Checks |
|---|---|
| `nginx: command not found` | Confirm OS and install with `sudo dnf install nginx -y` on Amazon Linux 2023. |
| Nginx fails to start | `sudo systemctl status nginx`; `sudo journalctl -u nginx --no-pager -n 50`. |
| Local curl fails | Check service/config/listener; validate with `sudo nginx -t`. |
| Local curl works, browser times out | Check public IP, inbound TCP 80, routing, network ACLs, and host firewall. |
| Old page appears | Refresh browser and inspect `index.html`. |
| Nginx not active after reboot | Check `systemctl is-enabled`, `systemctl status`, and service logs. |
| Browser says “Not secure” | Expected for HTTP; TLS/HTTPS was not configured in this lab. |

## Security notes
- Restrict SSH to trusted sources or an approved management path.
- Open only required ports.
- Use HTTPS/TLS for production applications that need transport encryption.
- Never include passwords, private keys, tokens, or sensitive data in the HTML or repository.
- Consider removing or moving backup files from public document roots in production.

## Interview explanation
“I deployed Nginx on an Amazon Linux 2023 EC2 instance, enabled it to start at boot, and created a custom HTML page containing the engineer's name, environment, and instance hostname. I verified the page using curl locally and a browser externally, and configured the security group for required HTTP access.”

**Difference between active and reachable:** An active service means Nginx is running under systemd. Reachability means a client can reach the application over the network. Security groups, routing, network ACLs, firewalls, and the Nginx listener can block external access even when the service is active.

## Evidence screenshots
Save genuine lab screenshots under `screenshots/`:
1. `01-os-identification.png` — OS output and initial Nginx check.
2. `02-nginx-service-running.png` — service active and local curl response.
3. `03-nginx-browser-access.png` — default page reached using public IP.
4. `04-custom-website.png` — custom page details.
5. `05-nginx-after-reboot.png` — post-reboot service and curl checks.
6. `06-security-group-rules.png` — inbound rules.

## Completion checklist
- [ ] Amazon Linux EC2 instance running.
- [ ] Nginx installed and started.
- [ ] Nginx enabled at boot.
- [ ] Custom page shows engineer name, environment, and actual hostname.
- [ ] Local curl and browser checks succeed.
- [ ] Post-reboot service and HTTP checks pass.
- [ ] Security group allows only necessary inbound access.
- [ ] Real screenshots saved.
- [ ] No secrets/private keys included in GitHub.
