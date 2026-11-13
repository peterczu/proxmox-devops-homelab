# Project 05: Docker and Docker Compose

## 1. Project Overview

### Objective

Build practical containerization and application deployment skills using Docker and Docker Compose.

This project involves deploying a multi-container application environment on an Ubuntu Server virtual machine running under Proxmox VE.

The environment consists of an Nginx web server, a FastAPI backend and a PostgreSQL database.

The objective is to understand container lifecycle management, service communication, persistent storage, application configuration, health monitoring, resource management and container troubleshooting.

## 2. Lab Environment

The Docker environment runs on VM102 in the Proxmox homelab.

| Component | Configuration |
|---|---|
| Hypervisor | Proxmox VE |
| Operating system | Ubuntu Server |
| Hostname | ubuntu-devops-03 |
| VM ID | 102 |
| Role | Docker application host |
| Project directory | `/opt/docker-compose-lab` |

The Docker host is managed remotely from the Ansible control node.

## 3. Application Architecture

The application uses three main services:

1. Nginx: Web server and reverse proxy.
2. FastAPI: Application backend.
3. PostgreSQL: Relational database.

### Architecture Diagram

```text
                 Client
                   |
                   |
             Port 8082
                   |
                   v
          Nginx Reverse Proxy
          compose-nginx-lab
                   |
             Docker Network
                   |
                   v
           FastAPI Backend
        compose-fastapi-backend
                   |
             Docker Network
                   |
                   v
            PostgreSQL DB
          compose-postgres-db
                   |
                   v
             Named Volume
              postgres-data
```

The services communicate over the Docker Compose network using service names.

## 4. Docker Compose Project Structure

The project is maintained under:

```text
/opt/docker-compose-lab
```

The environment includes:

```text
docker-compose-lab/
├── compose.yaml
├── Dockerfile
├── main.py
├── requirements.txt
├── .env
├── index.html
└── nginx/
    └── default.conf
```

The actual project may contain additional files as development progresses.

### Important Files

| File | Purpose |
|---|---|
| `compose.yaml` | Defines services, networks, volumes and configuration |
| `Dockerfile` | Defines how the FastAPI image is built |
| `main.py` | FastAPI application |
| `requirements.txt` | Python dependencies |
| `.env` | Application and database configuration |
| `index.html` | Nginx-served web content |
| `nginx/default.conf` | Nginx reverse proxy configuration |

## 5. Docker Services

### 5.1 Nginx Web Server

Container name:

```text
compose-nginx-lab
```

Image:

```text
nginx:latest
```

Published port:

```text
8082:80
```

Nginx listens on container port 80 and makes the web service available through port 8082 on the Docker host.

A bind mount provides web content:

```yaml
volumes:
  - /opt/docker-bind-lab:/usr/share/nginx/html:ro
  - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
```

The `:ro` option mounts the files read-only inside the container.

### Nginx Reverse Proxy

The Nginx configuration routes API requests to the FastAPI backend.

```nginx
server {
    listen 80;

    location /api/ {
        proxy_pass http://backend:8000/;
    }

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
```

The trailing slash in the `proxy_pass` directive is important.

For example:

```text
/api/health
```

is forwarded to:

```text
/health
```

on the backend.

Nginx also serves the static web content.

### 5.2 FastAPI Backend

Container name:

```text
compose-fastapi-backend
```

Image:

```text
fastapi-backend:1.4
```

The backend provides application functionality and communicates with PostgreSQL.

The application uses environment variables to obtain configuration.

Example:

```python
MESSAGE = os.getenv(
    "APP_MESSAGE",
    "Hello from the FastAPI backend!"
)
```

This allows the application message to be configured outside the Python source code.

### Environment Configuration

Docker Compose uses:

```yaml
env_file:
  - .env
```

The `env_file` directive loads variables into the container environment.

Python's `os.getenv()` reads those environment variables when the application starts.

This separates application configuration from the application source code.

### Backend Health Check

The FastAPI container has a health check that makes an HTTP request to its own health endpoint.

The check uses Python's `urllib` library to request:

```text
http://localhost:8000/health
```

The configured health-check parameters include:

| Parameter | Value |
|---|---|
| Interval | 10 seconds |
| Timeout | 5 seconds |
| Retries | 3 |
| Start period | 15 seconds |

Docker executes the health-check command inside the container.

An important distinction is that changing the health-check configuration does not require rebuilding the application image. The container must, however, be recreated if its Compose configuration changes.

### 5.3 PostgreSQL Database

Container name:

```text
compose-postgres-db
```

Image:

```text
postgres:16
```

The database stores application data and is accessed by FastAPI over the Docker network.

The database configuration is supplied through the `.env` file.

The application uses the PostgreSQL Python driver, `psycopg`, to communicate with the database.

### Database Health Check

The database health check uses:

```bash
pg_isready -U devops_user -d devops_app
```

Configuration:

| Parameter | Value |
|---|---|
| Interval | 5 seconds |
| Timeout | 3 seconds |
| Retries | 5 |
| Start period | 10 seconds |

This checks whether PostgreSQL is ready to accept connections.

## 6. Docker Networking

Docker Compose creates a network for the application services.

Services can communicate using their Compose service names instead of manually configured IP addresses.

For example, the Nginx configuration uses:

```text
http://backend:8000
```

The FastAPI backend uses the database service hostname:

```text
db
```

### Key Learning

Docker Compose service discovery allows containers to communicate through DNS names on their shared network.

Container IP addresses can change when containers are recreated. Using service names avoids hardcoding those addresses into application configuration.

## 7. Persistent Storage

PostgreSQL uses a named Docker volume:

```text
postgres-data
```

This volume preserves database files independently of the database container's lifecycle.

### Important Distinction

- A container is a running instance of an image.
- A named volume stores persistent data.
- Removing or recreating a container does not automatically mean its named volume is deleted.

The PostgreSQL volume must be protected during cleanup and deployment operations.

Avoid using destructive commands such as:

```bash
docker compose down -v
```

unless intentionally deleting the associated persistent volumes.

## 8. Container Health and Dependencies

Docker Compose uses health checks to monitor service readiness.

The application stack uses service dependencies so that dependent services wait for required services to become healthy.

Observed startup output included:

```text
Container compose-postgres-db Running
Container compose-fastapi-backend Running
Container compose-nginx-lab Running
Container compose-postgres-db Healthy
Container compose-fastapi-backend Healthy
```

### Important Learning

A running container is not necessarily a healthy container.

Docker distinguishes between:

- Running: The container process is active.
- Healthy: The configured health check is passing.
- Unhealthy: The configured health check has repeatedly failed.

A health check reports application or service health. It does not automatically restart an unhealthy container.

## 9. Resource Management

Container resource limits were configured to prevent individual services from consuming excessive host resources.

| Service | Memory limit | CPU limit |
|---|---:|---:|
| Nginx | 128 MB | 0.25 |
| FastAPI | 256 MB | 0.50 |
| PostgreSQL | 512 MB | 1.00 |
| **Total** | **896 MB** | **1.75 CPUs** |

### Monitoring Container Resources

Used:

```bash
docker stats
```

This displays live container resource consumption, including CPU, memory and network usage.

Resource limits help maintain predictable resource allocation on a shared infrastructure host.

## 10. Container Logging

Container logs were inspected during application deployment and troubleshooting.

Useful commands:

### View Recent Logs

```bash
docker logs --tail 20 compose-fastapi-backend
```

### View Logs from the Last Five Minutes

```bash
docker logs --since 5m compose-fastapi-backend
```

### Follow Live Logs

```bash
docker logs -f compose-fastapi-backend
```

The `-f` option follows the log stream continuously until interrupted.

### Docker Logging Configuration

The FastAPI backend was configured to use Docker's `json-file` logging driver with log rotation.

Configured limits:

| Setting | Value |
|---|---|
| Maximum log file size | 10 MB |
| Maximum log files | 3 |

This helps prevent unlimited growth of container log files.

## 11. Container Lifecycle Management

Practised managing the application stack using Docker Compose.

### Start Services

```bash
docker compose up -d
```

### Stop and Remove Containers

```bash
docker compose down
```

### List Running Containers

```bash
docker ps
```

### Inspect Container Details

```bash
docker inspect compose-fastapi-backend
```

### Validate Compose Configuration

```bash
docker compose config -q
```

### Restart a Container

```bash
docker restart compose-fastapi-backend
```

The `-d` option runs services in detached mode, allowing the terminal to return without remaining attached to the containers.

## 12. Docker Image and Configuration Management

An important distinction was learned between image changes and container configuration changes.

### Application Code Changes

The Dockerfile copies application source code into the image.

When application source code changes, the image must be rebuilt before a newly created container can use the updated code.

Example:

```bash
docker compose build backend
```

### Compose Configuration Changes

Changes to environment configuration, published ports, resource limits, health checks or logging configuration may require container recreation.

Example:

```bash
docker compose up -d
```

Compose reconciles the running services with the updated configuration.

### Key Learning

A running container does not automatically receive every change made to its Compose file or image source.

Understanding when to rebuild an image and when to recreate a container is essential for reliable deployments.

## 13. Ansible-Based Docker Management

The Docker host is managed remotely using Ansible from VM100.

Example:

```bash
ansible webservers -i inventory.ini -b -m command -a "docker compose -f /opt/docker-compose-lab/compose.yaml up -d"
```

This executes Docker Compose remotely on the managed Docker host.

It combines infrastructure automation with container management.

## 14. Practical Troubleshooting and Learning

During the project, several operational concepts were explored:

- Understanding Docker Compose service definitions.
- Configuring environment variables.
- Understanding how Python reads container environment variables.
- Configuring application health checks.
- Understanding service dependencies.
- Inspecting container logs.
- Managing persistent database storage.
- Monitoring container resource consumption.
- Configuring logging rotation.
- Understanding container restart policies.
- Recreating containers after configuration changes.
- Distinguishing image rebuilds from container recreation.
- Deploying Docker Compose remotely through Ansible.

### Restart Policy

The services use:

```yaml
restart: unless-stopped
```

This configures Docker to restart containers after certain unexpected exits, unless they have been explicitly stopped.

A health check failure alone does not trigger this restart policy.

## 15. Technologies and Tools

- Docker Engine
- Docker Compose
- Nginx
- FastAPI
- Python
- PostgreSQL 16
- psycopg
- Ubuntu Server
- Linux networking
- Ansible
- YAML
- Git and GitHub

## 16. Future Improvements

Planned improvements include:

- Dockerfile optimization and multi-stage builds.
- Docker image versioning and tagging.
- Container security and non-root execution.
- Docker secrets and improved credential management.
- Backup and recovery of PostgreSQL data.
- Centralized container logging.
- Monitoring with Prometheus and Grafana.
- Docker networking and isolation.
- Container orchestration with Kubernetes.
- CI/CD-based image builds and deployments.

## 17. Project Status

**Status:** In progress

A multi-container application environment is operational using Docker Compose.

The project has provided practical experience with reverse proxy configuration, backend and database integration, persistent storage, container health checks, resource limits, logging and remote deployment through Ansible.

The next stage will extend these skills into container orchestration, observability, automation and Kubernetes.
