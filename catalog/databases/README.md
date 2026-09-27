```
╭──────────────────────────────────────────────╮
│                                              │
│  D A T A B A S E S                           │
│  sql · cache · shared backends               │
│                                              │
╰──────────────────────────────────────────────╯
```

<p align="center">
  <img src="../../assets/mascot/brain_readme.png" width="200">
</p>

<br>


> The databases and caches other services depend on, kept separate from the
> applications that use them.

**Running a stack** — every folder here is a self-contained Docker Compose stack:

```bash
cd <service>
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env          # then edit the values
docker compose up -d
```

> Need Docker installed first? See [Requirements](../README.md#requirements).

New backends are added as needed — the folders in this directory are the current list.

<sub><a href="../">← all services</a></sub>
