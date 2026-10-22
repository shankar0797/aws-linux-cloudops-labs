# Task 9: Docker Web Application — Cheatsheet

## 1. Docker Image Commands

```bash
# Build an image
sudo docker build -t cloudops-webapp:1.0 .

# List images
sudo docker images

# Inspect an image
sudo docker image inspect cloudops-webapp:1.0
```

## 2. Container Commands

```bash
# Run the application
sudo docker run -d \
  --name cloudops-webapp \
  --restart unless-stopped \
  -p 8080:80 \
  cloudops-webapp:1.0

# List running containers
sudo docker ps

# List all containers
sudo docker ps -a

# View logs
sudo docker logs cloudops-webapp

# Follow logs
sudo docker logs -f cloudops-webapp

# Inspect container configuration
sudo docker inspect cloudops-webapp

# Stop and start the container
sudo docker stop cloudops-webapp
sudo docker start cloudops-webapp
```

## 3. HTTP Validation

```bash
# Test from the EC2 host
curl -i http://localhost:8080

# Show response headers
curl -I http://localhost:8080

# Show detailed connection information
curl -v http://localhost:8080
```

Expected successful response:

```text
HTTP/1.1 200 OK
Content-Type: text/html
```

## 4. Port Mapping

```text
-p 8080:80
   |    |
   |    +-- Container port: 80
   +------- EC2 host port: 8080
```

Public endpoint:

```text
http://PUBLIC-IP:8080
```

Replace `PUBLIC-IP` with the actual public IPv4 address of the instance.

Remember: `EXPOSE 80` documents a container port; `-p 8080:80` publishes it through the Docker host.

## 5. Quick Troubleshooting

| Symptom | Commands and checks |
|---|---|
| Container stopped | `sudo docker ps -a` and `sudo docker logs cloudops-webapp` |
| Local HTTP fails | `curl -v http://localhost:8080`; check `sudo docker ps` |
| Port conflict | `sudo ss -lntp` and inspect existing container mappings |
| Wrong page appears | Verify `app/index.html`, rebuild the image, recreate the container |
| Public endpoint times out | Check public IP, security group, subnet routing, and container port mapping |
| Docker daemon unavailable | `sudo systemctl status docker` |

## 6. Rebuild After an HTML Change

```bash
# Rebuild the image
sudo docker build -t cloudops-webapp:1.0 .

# Replace the existing container
sudo docker stop cloudops-webapp
sudo docker rm cloudops-webapp

# Start a new container from the rebuilt image
sudo docker run -d \
  --name cloudops-webapp \
  --restart unless-stopped \
  -p 8080:80 \
  cloudops-webapp:1.0

# Validate the new container
sudo docker ps
curl -i http://localhost:8080
```

Only remove the intended container. Check its name and dependencies before executing removal commands.

## 7. Interview Revision

**What is a Docker image?**  
A template used to create containers.

**What is a Docker container?**  
An isolated running or stopped instance of an image.

**What does `-p 8080:80` do?**  
It maps host port 8080 to container port 80.

**Does `EXPOSE 80` publish the port?**  
No. It documents the intended container port.

**How do you troubleshoot a container that is not serving HTTP?**  
Check container status, logs, port mapping, local `curl`, and network/security-group rules.

**Why does local access work but public access fail?**  
The application may be running correctly, but the public IP, security group, routing, or external network path may be incorrect.

## 8. Security Reminders

- Restrict port 8080 to trusted source IPs.
- Use HTTPS for production workloads.
- Do not store secrets in the image or repository.
- Stop or remove lab resources when no longer required.

