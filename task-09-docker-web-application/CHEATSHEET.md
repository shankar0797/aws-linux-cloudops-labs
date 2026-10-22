\# Task 9: Docker Web Application Cheatsheet



\## Build and Image Management



```bash

\# Build an image from the current directory

sudo docker build -t cloudops-webapp:1.0 .



\# List local images

sudo docker images



\# Inspect image details

sudo docker image inspect cloudops-webapp:1.0

```



\## Container Management



```bash

\# Start the container

sudo docker run -d --name cloudops-webapp --restart unless-stopped -p 8080:80 cloudops-webapp:1.0



\# List running containers

sudo docker ps



\# List all containers, including stopped containers

sudo docker ps -a



\# View application logs

sudo docker logs cloudops-webapp



\# Follow logs in real time

sudo docker logs -f cloudops-webapp



\# Stop the container

sudo docker stop cloudops-webapp



\# Start an existing stopped container

sudo docker start cloudops-webapp



\# Remove a stopped container

sudo docker rm cloudops-webapp

```



\## HTTP Testing



```bash

\# Test from the EC2 instance

curl -i http://localhost:8080



\# Fetch only response headers

curl -I http://localhost:8080



\# Test from the EC2 instance with verbose connection details

curl -v http://localhost:8080

```



Expected successful response:



```text

HTTP/1.1 200 OK

Content-Type: text/html

```



\## Port Mapping



```text

\-p 8080:80

&#x20;  |    |

&#x20;  |    +-- Container port (Nginx)

&#x20;  +------- EC2 host port

```



Users access the application through:



```text

http://PUBLIC-IP:8080

```



The EC2 security group and network configuration must allow the required inbound traffic.



\## Troubleshooting



\### Container exits or keeps restarting



```bash

sudo docker ps -a

sudo docker logs cloudops-webapp

sudo docker inspect cloudops-webapp

```



\### Port conflict



```bash

sudo docker ps

sudo ss -lntp

```



Choose a free host port if port `8080` is already occupied. Update the security group and browser URL to match.



\### Image changes are not visible



Rebuild the image and recreate the container when necessary:



```bash

sudo docker build -t cloudops-webapp:1.0 .

sudo docker stop cloudops-webapp

sudo docker rm cloudops-webapp

sudo docker run -d --name cloudops-webapp --restart unless-stopped -p 8080:80 cloudops-webapp:1.0

```



\### Browser times out



Check, in order:



1\. The correct public IPv4 address.

2\. The container status and port mapping.

3\. The local `curl` response.

4\. The EC2 security group's inbound TCP rule for port `8080`.

5\. The instance's subnet routing and public connectivity.



\## Interview Quick Answers



\*\*What is a Docker image?\*\*  

A packaged template containing the application and its required filesystem and runtime configuration.



\*\*What is a Docker container?\*\*  

A running or stopped instance created from a Docker image.



\*\*What does `-p 8080:80` mean?\*\*  

It maps host port `8080` to container port `80`.



\*\*Does `EXPOSE 80` publish the port?\*\*  

No. It documents the intended container port. Port publishing is configured with `-p` or another networking method.



\*\*How do you troubleshoot a container that is not serving HTTP?\*\*  

Check `docker ps -a`, container logs, port mappings, local `curl`, and the EC2 security group and network path.



\*\*Why use Nginx?\*\*  

Nginx is a web server that serves the static HTML page from inside the container.



\*\*What is the difference between local and public testing?\*\*  

Local testing verifies the service from the EC2 host. Public testing also depends on the instance's network reachability, public IP, and inbound security rules.





