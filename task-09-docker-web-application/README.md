\# Task 9: Deploy a Dockerized Web Application on AWS EC2



\## Project Overview



This hands-on project demonstrates how to build a custom Docker image, run a containerized web application on an AWS EC2 instance, and expose the application through a public HTTP endpoint.



The application uses Nginx to serve a custom HTML page from inside a Docker container.



\## Architecture



```text

User Browser

&#x20;    |

&#x20;    | HTTP :8080

&#x20;    v

EC2 Public IPv4

&#x20;    |

&#x20;    v

EC2 Security Group

&#x20;    |

&#x20;    v

Host Port 8080

&#x20;    |

&#x20;    v

Docker Container Port 80

&#x20;    |

&#x20;    v

Nginx Web Server

&#x20;    |

&#x20;    v

Custom HTML Application

```



\## Technologies Used



\- AWS EC2

\- Amazon Linux 2023

\- Docker

\- Nginx

\- HTML and CSS

\- AWS Security Groups

\- HTTP and Linux command-line tools



\## Implementation Steps



\### 1. Prepare the application



Created a project directory with the following structure:



```text

task-09-docker-web-application/

├── app/

│   └── index.html

├── Dockerfile

├── README.md

├── CHEATSHEET.md

└── screenshots/

```



\### 2. Create the Dockerfile



```dockerfile

FROM nginx:alpine

COPY app/index.html /usr/share/nginx/html/index.html

EXPOSE 80

```



\*\*Explanation:\*\*



\- `FROM nginx:alpine` uses the lightweight Nginx image as the base.

\- `COPY` places the custom HTML page in Nginx's default web root.

\- `EXPOSE 80` documents the container port used by the web server. It does not publish the port to the host by itself.



\### 3. Build the Docker image



```bash

sudo docker build -t cloudops-webapp:1.0 .

```



This builds an image named `cloudops-webapp` with the tag `1.0` using the Dockerfile in the current directory.



\### 4. Run the container



```bash

sudo docker run -d \\

&#x20; --name cloudops-webapp \\

&#x20; --restart unless-stopped \\

&#x20; -p 8080:80 \\

&#x20; cloudops-webapp:1.0

```



\*\*Explanation:\*\*



\- `-d` runs the container in the background.

\- `--name` assigns a readable container name.

\- `--restart unless-stopped` restarts the container after applicable Docker or host restarts unless it was explicitly stopped.

\- `-p 8080:80` maps EC2 host port `8080` to container port `80`.

\- `cloudops-webapp:1.0` specifies the image to run.



\### 5. Verify the container



```bash

sudo docker ps

sudo docker images

sudo docker logs cloudops-webapp

```



Confirmed that the container was running and that Nginx started successfully.



\### 6. Test the application locally on EC2



```bash

curl -i http://localhost:8080

```



\*\*Observed result:\*\* `HTTP/1.1 200 OK`



The response returned the custom HTML application.



\### 7. Configure public HTTP access



In the AWS EC2 console:



1\. Open the EC2 instance and identify its attached security group.

2\. Add an inbound TCP rule for port `8080`.

3\. Restrict the source to the required client IP/CIDR wherever possible.

4\. Open the public IPv4 address in a browser using `http://PUBLIC-IP:8080`.



The browser successfully displayed the \*\*CloudOps Docker Demo\*\* page, confirming public HTTP access at the time of testing.



\## Verification Results



| Check | Result |

|---|---|

| Docker image build | Successful |

| Container startup | Successful |

| Nginx startup | Successful |

| Local HTTP response | `200 OK` |

| Public browser access | Verified |

| Custom HTML page | Displayed successfully |



\## Evidence



Screenshots are stored in the `screenshots/` directory.



1\. `01-docker-image.png` — Docker image listing.

2\. `02-container-http-200.png` — Running container and HTTP response.

3\. `03-public-web-application.png` — Application displayed in the browser.



\## Troubleshooting



\### Browser cannot reach the application



Check the EC2 public IPv4 address, inbound security group rule, network configuration, and whether the instance has a reachable public network path.



\### Container is not running



```bash

sudo docker ps -a

sudo docker logs cloudops-webapp

```



\### Port 8080 is already in use



Check the existing listener and container mappings before choosing another unused host port.



\### Application returns an unexpected page



Verify the HTML file copied into the image, rebuild the image, and recreate the container if required.



\## Security and Cost Considerations



\- Avoid exposing port `8080` to the entire internet unless the demonstration specifically requires it.

\- Restrict inbound access to trusted IP addresses.

\- Use HTTPS and an appropriate TLS termination setup for production workloads.

\- Stop or terminate lab resources when they are no longer needed, while preserving any evidence you want to retain.

\- Do not store credentials, private keys, or other secrets in the repository.



\## Learning Outcomes



\- Created a custom Docker image.

\- Deployed and managed a Docker container on EC2.

\- Published a container port to the EC2 host.

\- Validated HTTP responses using `curl`.

\- Configured network access through an EC2 security group.

\- Verified a public web application and documented the deployment.





