# Task 9: Deploy a Dockerized Web Application on AWS EC2

## 1. Project Overview

This project demonstrates how to build a custom Docker image, run a containerized web application on Amazon EC2, and access the application through a public HTTP endpoint.

The application uses Nginx to serve a custom HTML page from inside a Docker container.

### Objectives

- Create a custom HTML web page.
- Build a Docker image using a Dockerfile.
- Run and manage a Docker container on EC2.
- Publish the container's HTTP port to the EC2 host.
- Configure network access using an EC2 security group.
- Validate local and public HTTP connectivity.
- Capture screenshots and document troubleshooting steps.

## 2. Environment and Technologies

| Component | Details |
|---|---|
| Cloud provider | AWS |
| Compute | Amazon EC2 |
| Operating system | Amazon Linux 2023 |
| Container runtime | Docker |
| Base image | `nginx:alpine` |
| Application | HTML and CSS |
| Web server | Nginx |
| Container port | 80 |
| EC2 host port | 8080 |
| Protocol | HTTP |
| Image name | `cloudops-webapp:1.0` |
| Container name | `cloudops-webapp` |

## 3. Architecture

```text
User Browser
     |
     | HTTP :8080
     v
EC2 Public IPv4
     |
     v
EC2 Security Group
     |
     v
EC2 Host Port 8080
     |
     v
Docker Port Mapping
     |
     v
Container Port 80
     |
     v
Nginx Web Server
     |
     v
/usr/share/nginx/html/index.html
```

The EC2 security group must allow the intended client to reach TCP port 8080. The instance must also have a valid public network path.

## 4. Project Structure

```text
task-09-docker-web-application/
├── app/
│   └── index.html
├── Dockerfile
├── README.md
├── CHEATSHEET.md
├── task-09-app.zip
└── screenshots/
    ├── 01-docker-image.png
    ├── 02-container-http-200.png
    └── 03-public-web-application.png
```

## 5. Application and Dockerfile

The custom page is stored in `app/index.html`. It displays the CloudOps Docker Demo heading, deployment status, and technologies used.

### Dockerfile

```dockerfile
FROM nginx:alpine
COPY app/index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

### Explanation

- `FROM nginx:alpine` selects a lightweight Nginx base image.
- `COPY` places the custom HTML page in Nginx's default document directory.
- `EXPOSE 80` documents the container port used by Nginx. It does not publish the port to the EC2 host by itself.

## 6. Implementation Steps

### Step 1: Connect to the EC2 instance

Connect to the intended Amazon Linux 2023 instance using an approved access method, such as AWS Systems Manager Session Manager or SSH.

Check the operating system and Docker installation:

```bash
cat /etc/os-release
docker --version
sudo docker info
```

If Docker is not running, inspect its service before making changes:

```bash
sudo systemctl status docker
```

### Step 2: Create the application files

Create the project directory and application directory:

```bash
mkdir -p ~/task-09-docker-web-application/app
cd ~/task-09-docker-web-application
```

Create `app/index.html` with the custom CloudOps Docker Demo page, then create the Dockerfile shown above.

### Step 3: Build the Docker image

```bash
sudo docker build -t cloudops-webapp:1.0 .
```

The command builds the image from the Dockerfile in the current directory and tags it as `cloudops-webapp:1.0`.

Verify the image:

```bash
sudo docker images
```

### Step 4: Start the container

```bash
sudo docker run -d \
  --name cloudops-webapp \
  --restart unless-stopped \
  -p 8080:80 \
  cloudops-webapp:1.0
```

Command breakdown:

- `-d`: run the container in the background.
- `--name cloudops-webapp`: assign a container name.
- `--restart unless-stopped`: configure automatic restart behavior.
- `-p 8080:80`: map EC2 host port 8080 to container port 80.
- `cloudops-webapp:1.0`: select the image to run.

### Step 5: Verify container status and logs

```bash
sudo docker ps
sudo docker logs cloudops-webapp
```

The container should be running, and Nginx should start without fatal errors.

### Step 6: Test HTTP from EC2

```bash
curl -i http://localhost:8080
```

**Observed result:** `HTTP/1.1 200 OK`, with the expected HTML content.

This confirms that the application responded successfully from the EC2 host.

### Step 7: Configure public HTTP access

In the AWS Console:

1. Open **EC2 → Instances**.
2. Select the instance running the container.
3. Open its attached security group.
4. Add an inbound TCP rule for port `8080`.
5. Set the source to the intended client IP/CIDR, preferably your current public IP for a personal test.
6. Confirm that the instance has a reachable public IPv4 address and an appropriate network route.

Open this URL in a browser:

```text
http://PUBLIC-IP:8080
```

Replace `PUBLIC-IP` with the instance's actual public IPv4 address.

**Observed result:** The browser displayed the CloudOps Docker Demo page, including “Deployment Successful!” and “Application status: Running.”

## 7. Validation Results

| Validation | Result |
|---|---|
| Docker image built | Successful |
| Container started | Successful |
| Container port mapping | Host 8080 → container 80 |
| Local HTTP test | `200 OK` |
| Public browser test | Successful at the time of testing |
| Custom application content | Displayed correctly |

## 8. Troubleshooting

### Container is not running

```bash
sudo docker ps -a
sudo docker logs cloudops-webapp
sudo docker inspect cloudops-webapp
```

Use the container state and logs to identify startup errors.

### Local HTTP test fails

```bash
sudo docker ps
sudo docker logs cloudops-webapp
curl -v http://localhost:8080
sudo ss -lntp
```

Check the container status, host port mapping, Nginx logs, and port availability.

### Public endpoint is unreachable

Check:

1. The correct EC2 public IPv4 address.
2. The security group's inbound TCP rule for port 8080.
3. The subnet route and internet gateway path, where applicable.
4. The container's published port.
5. The local `curl` response.

If local access works but public access fails, focus on the network path and security rules.

### Updated HTML does not appear

Rebuild the image and recreate the container if required. Building a new image does not automatically replace the image used by an existing container.

## 9. Security and Cost Considerations

- Restrict inbound port 8080 to trusted client addresses where possible.
- Avoid exposing demonstration services to the entire internet unnecessarily.
- HTTP is unencrypted; use an appropriate HTTPS design for production.
- Never commit credentials, private keys, or access tokens.
- Stop or terminate unused lab resources according to your retention needs.
- Confirm the impact before deleting or modifying resources.

## 10. Evidence

Screenshots captured for this lab:

- `screenshots/01-docker-image.png` — Docker image listing.
- `screenshots/02-container-http-200.png` — Running container and successful local HTTP response.
- `screenshots/03-public-web-application.png` — Application displayed through the public endpoint.

## 11. Learning Outcomes

This lab provided hands-on practice with Docker image creation, container lifecycle management, host-to-container port mapping, Nginx static content serving, HTTP testing, and EC2 security-group configuration.

