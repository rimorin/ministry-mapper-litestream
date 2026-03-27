# 🗺️ Ministry Mapper — Litestream Backup

> Continuously replicates a SQLite database to Cloudflare R2 using [Litestream](https://litestream.io).
> Environment-agnostic — deploy once for dev, staging, and production.

> **Platform note:** This setup is configured for [Coolify](https://coolify.io). The deployment steps and workarounds are Coolify-specific, but the core principle — one compose file per environment, inline litestream config, bind mount to the data directory — applies to any Docker-based platform.

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

| Variable | Description | All Envs |
|----------|-------------|:--------:|
| `R2_ACCESS_KEY_ID` | Cloudflare R2 access key | ← same → |
| `R2_SECRET_ACCESS_KEY` | Cloudflare R2 secret | ← same → |
| `R2_ENDPOINT` | R2 account endpoint | ← same → |

The following are **hardcoded** in each compose file (not env vars):

| Setting | Dev | Staging | Production |
|---------|:---:|:-------:|:----------:|
| `R2_BUCKET` | `mm-pb-backups` | ← same → | ← same → |
| `R2_PATH` | `dev` | `staging` | `production` |
| `SYNC_INTERVAL` | `300s` | `60s` | `10s` |

---

## 🚢 Coolify Deployment

Each environment gets its own Coolify resource pointing to its respective compose file. All shared litestream config lives in `docker-compose.base.yml` — edit only that file when updating replication settings.

| Environment | Compose File | Data Path |
|-------------|-------------|-----------|
| Dev | `docker-compose.dev.yml` | `/data/dev` |
| Staging | `docker-compose.staging.yml` | `/data/staging` |
| Production | `docker-compose.production.yml` | `/data/production` |

1. **Add new resource** → connect git repo → select **Docker Compose** build pack
2. **Set the compose file path** to the environment's file (see table above)
3. **Set environment variables** in the Coolify UI
4. **Deploy**

### Known Coolify Limitations

> **🔒 Env vars blocked in volume paths** ([issue #7127](https://github.com/coollabsio/coolify/issues/7127))
> Since v4.0.0-beta.441, Coolify blocks `${VAR}` in bind mount source paths as a security measure against command injection. `${DATA_PATH}:/data` is silently treated as a named Docker volume instead of a bind mount, so the container never sees your data directory. **Workaround:** hardcode the path per environment in separate compose files (current approach).

> **🔍 Env vars not auto-detected from `extends` base file**
> Coolify only scans the compose file you point it to for `${VAR}` references — it does not follow `extends` into `docker-compose.base.yml`. This means env vars defined only in the base file won't appear in Coolify's UI. **Workaround:** the `environment:` block is intentionally repeated in each env file so Coolify surfaces them for editing.
