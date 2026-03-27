# Ministry Mapper — Litestream Backup

Replicates SQLite database to Cloudflare R2 using Litestream.
Environment-agnostic — deploy once for staging, once for production.

## Setup

1. Copy `.env.example` to `.env` and fill in values
2. Deploy via Coolify pointing to `docker-compose.yml`
3. Set env vars in Coolify UI (no `.env` file needed in production)

## Environment Variables

| Variable | Description | Staging | Production |
|----------|-------------|---------|------------|
| `R2_ACCESS_KEY_ID` | Cloudflare R2 key | same | same |
| `R2_SECRET_ACCESS_KEY` | Cloudflare R2 secret | same | same |
| `R2_ENDPOINT` | R2 account endpoint | same | same |
| `R2_BUCKET` | R2 bucket name | same | same |
| `R2_PATH` | Path inside bucket | `staging` | `production` |
| `DATA_PATH` | Host path to SQLite data | `/data/staging` | `/data/production` |
| `SYNC_INTERVAL` | Replication frequency | `60s` | `10s` |

## Coolify Deployment

1. Add new service → Docker Compose → point to this repo
2. Set environment variables in Coolify UI
3. Deploy — repeat for staging with different env vars
