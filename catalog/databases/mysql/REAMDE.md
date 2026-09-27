MySQL Service

<p align="center">
  <img src="../../assets/mysql-logo.png" width="180">
</p>

<br>


This directory contains an example configuration for deploying a standalone
MySQL database container using Docker Compose.

What is MySQL

MySQL is an open-source relational database management system (RDBMS)
based on structured query language (SQL).
It is commonly used for web applications, backend services, and development
environments.

Official image:
https://hub.docker.com/_/mysql

Purpose in this lab
	•	Local development database
	•	Persistent containerized storage
	•	Isolated infrastructure layer
	•	Separate Docker network for database services

Network Design

A dedicated Docker network is used:

mysql_network

This isolates the database from unrelated services while still allowing
controlled inter-container communication.

Port Exposure

Port 3306 is exposed to localhost for development access:

localhost:3306

This enables:
	•	Local SQL clients
	•	Application testing
	•	ORM development

Data Persistence

Database data is stored in a named Docker volume:

mysql_data

This ensures data survives container restarts.

Notes
	•	Credentials are stored in a separate .env file
	•	Example configuration provided
	•	Do not commit real secrets

Thanks

Thanks to the MySQL maintainers for providing a stable and widely adopted
database engine suitable for containerized development environments.
