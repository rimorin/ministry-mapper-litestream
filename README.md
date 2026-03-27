# 🗺️ Ministry Mapper — Litestream Backup

> Continuously replicates a SQLite database to Cloudflare R2 using [Litestream](https://litestream.io).
> Environment-agnostic — deploy once for staging, once for production.

---

## 🤔 Why Litestream?

Ministry Mapper uses SQLite as its database — simple, fast, and zero-dependency. The tradeoff is that SQLite lives on disk, making it vulnerable to data loss if the server goes down or is replaced.

Litestream solves this by continuously streaming SQLite WAL changes to remote object storage in real time.

| Benefit | Details |
|---------|---------|
| ⚡ **Continuous replication** | Changes streamed as they happen, not on a cron schedule |
| 🕐 **Point-in-time restore** | Recover the database to any point in its history |
| 🔌 **Zero app changes** | Runs as a sidecar; your app reads/writes SQLite normally |
| 💸 **Low cost** | R2 has no egress fees, ideal for frequent small writes |
| 🚀 **Fast recovery** | Restore and resume in seconds on a new server |

---

## ⚙️ Setup

```bash
# 1. Configure environment
cp .env.example .env  # fill in your values

# 2. Run locally (optional)
docker compose up
```

> **Production:** Set env vars directly in Coolify UI — no `.env` file needed.

---

## 🔐 Environment Variables

| Variable | Description | Staging | Production |
|----------|-------------|:-------:|:----------:|
| `R2_ACCESS_KEY_ID` | Cloudflare R2 access key | ← same → | ← same → |
| `R2_SECRET_ACCESS_KEY` | Cloudflare R2 secret | ← same → | ← same → |
| `R2_ENDPOINT` | R2 account endpoint | ← same → | ← same → |
| `R2_BUCKET` | R2 bucket name | ← same → | ← same → |
| `R2_PATH` | Path inside bucket | `staging` | `production` |
| `DATA_PATH` | Host path to SQLite data | `/data/staging` | `/data/production` |
| `SYNC_INTERVAL` | Replication frequency | `60s` | `10s` |

---

## 🚢 Coolify Deployment

1. **Add new resource** → connect git repo → select **Docker Compose** build pack → point to `docker-compose.yml`
2. **Set environment variables** in the Coolify UI
3. **Deploy** — repeat with different env vars for staging vs production
