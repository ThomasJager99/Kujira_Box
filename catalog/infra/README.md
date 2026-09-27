```
╭──────────────────────────────────────────────╮
│                                              │
│  I N F R A                                   │
│  proxy · dashboard · stacks · git            │
│                                              │
╰──────────────────────────────────────────────╯
```

<p align="center">
  <img src="../../assets/mascot/makkokujira.png" width="200">
</p>

<br>


> The plumbing that keeps everything else reachable and manageable — not
> end-user apps, but what the other services run on top of.

**Running a stack** — every folder here is a self-contained Docker Compose stack:

```bash
cd <service>
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env          # then edit the values
docker compose up -d
```

> Need Docker installed first? See [Requirements](../README.md#requirements).

New infrastructure pieces are added over time — the folders in this directory are the current list.

<sub><a href="../">← all services</a></sub>
