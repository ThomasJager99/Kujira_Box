```
╭──────────────────────────────────────────────╮
│                                              │
│  O B S E R V A B I L I T Y                   │
│  metrics · logs · uptime · health            │
│                                              │
╰──────────────────────────────────────────────╯
```

<p align="center">
  <img src="../../assets/mascot/brain_readme.png" width="200">
</p>

<br>


> Metrics, logs, uptime and system health — so problems are seen early instead
> of discovered by accident.

**Running a stack** — every folder here is a self-contained Docker Compose stack:

```bash
cd <service>
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env          # then edit the values
docker compose up -d
```

> Need Docker installed first? See [Requirements](../README.md#requirements).

New tools are added as monitoring coverage grows — the folders in this directory are the current list.

<sub><a href="../">← all services</a></sub>
