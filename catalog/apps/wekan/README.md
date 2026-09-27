<p align="center">
  <img src="../../../assets/logos/wekan.svg" width="180">
</p>

<br>

This directory contains an example deployment of WeKan, an open-source kanban board application used for task and project management. WeKan provides a Trello-like web interface where boards, lists and cards can be used to organize tasks, track progress and manage projects collaboratively.

Image

Official WeKan container image: https://github.com/wekan/wekan

Deployment Notes

Runs as two containers: the WeKan web application and a MongoDB database configured as a single-node replica set (required by WeKan for realtime updates)
Stores board data and uploaded files in persistent Docker volumes
Exposes a single HTTP interface
Includes healthchecks and memory limits for both containers
Environment-specific values (ports, memory limits, mail server, etc.) are externalized to a .env file

Example Use WeKan can be used to organize and track:

personal task boards
team kanban workflows
project and sprint planning
general to-do and progress tracking

This allows individuals or teams to manage work visually through a self-hosted Trello alternative.

Thanks Thanks to the WeKan project maintainers for providing a full-featured, open-source kanban board solution suitable for self-hosted environments.
