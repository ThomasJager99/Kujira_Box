Multi-Service Stack: Nginx + MySQL + Redis

<p align="center">
  <img src="../../../assets/logos/mysql-logo.png" width="180">
</p>

<br>


<p align="center">
  <img src="../../../assets/logos/nginx.svg" width="180">
</p>

<p align="center">
  <img src="../../../assets/logos/redis.png" width="180">
</p>


This directory contains an example Docker Compose stack demonstrating
a basic multi-service architecture using:
	•	Nginx (reverse proxy / web server)
	•	MySQL 8.0 (relational database)
	•	Redis 7 (in-memory data store)

The services are connected through a shared Docker network and use
persistent volumes where required.

⸻

Architecture Overview
	•	All containers are attached to a dedicated external Docker network.
	•	MySQL and Redis use named volumes for data persistence.
	•	Nginx serves static content and routes traffic.
	•	Healthchecks are configured for database and cache services.
	•	Environment variables are externalized via .env.

⸻

Services

Nginx

Official image:
https://hub.docker.com/_/nginx

Purpose:
	•	Acts as the web entry point
	•	Serves static files
	•	Demonstrates port exposure and container dependency control

Exposed port:
	•	${NGINX_HTTP_PORT} → container port 80

⸻

MySQL

Official image:
https://hub.docker.com/_/mysql

Purpose:
	•	Relational database backend
	•	Persistent storage via named volume
	•	Demonstrates environment-based configuration

Data persistence:
	•	mysql_data volume

Healthcheck:
	•	Uses mysqladmin ping to verify readiness

⸻

Redis

Official image:
https://hub.docker.com/_/redis

Purpose:
	•	In-memory key-value store
	•	Password-protected instance
	•	Persistent storage enabled via named volume

Data persistence:
	•	redis_data volume

Healthcheck:
	•	Uses redis-cli ping with authentication

⸻

Network

All services are attached to:

test_media

This network must exist prior to running the stack:

docker network create test_media

The network isolates the stack from unrelated containers while allowing
inter-service communication via Docker DNS.

⸻

Environment Configuration

Sensitive values are stored in .env:
	•	Database credentials
	•	Redis password
	•	Port mappings

Never commit real secrets.

⸻

Deployment

docker compose up -d

Check status:

docker compose ps
docker compose logs


⸻

What This Demonstrates
	•	Multi-container orchestration
	•	Service dependency management
	•	Health monitoring
	•	Persistent storage with named volumes
	•	External network usage
	•	Resource limitation
	•	Production-oriented container design

⸻

Thanks

Thanks to the maintainers of:
	•	Nginx
	•	MySQL
	•	Redis

for providing stable and production-ready container images suitable
for containerized environments and DevOps learning scenarios.
