```
╭──────────────────────────────────────────────╮
│                                              │
│  M E D I A                                   │
│  video · audio · books · comics · photos     │
│                                              │
╰──────────────────────────────────────────────╯
```

<p align="center">
  <img src="../../assets/mascot/brain_readme.png" width="200">
</p>

<br>


> Libraries and streaming for video, audio, books and photos — a private
> alternative to the cloud services, with the collection on hardware you control.

**Running a stack** — every folder here is a self-contained Docker Compose stack:

```bash
cd <service>
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env          # then edit the values
docker compose up -d
```

> Need Docker installed first? See [Requirements](../README.md#requirements).

New media services are added over time — the folders in this directory are the current list.

<sub><a href="../">← all services</a></sub>
