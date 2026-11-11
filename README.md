# Ministry Mapper — Litestream Backup

Replicates the Ministry Mapper PocketBase SQLite database (`data.db`) to Cloudflare R2 with [Litestream](https://litestream.io). It runs as a sidecar next to PocketBase, deployed as one Coolify resource per environment.

## Environments

|  | Dev | Staging | Production |
|---|---|---|---|
| Compose file | `docker-compose.dev.yml` | `docker-compose.staging.yml` | `docker-compose.production.yml` |
| Host data path | `/data/dev` | `/data/staging` | `/data/production` |
| R2 path | `dev` | `staging` | `production` |
| `SYNC_INTERVAL` | `300s` | `60s` | `10s` |
| `SNAPSHOT_RETENTION` | `24h` | `72h` | `168h` |

All environments use the `mm-pb-backups` bucket. `SYNC_INTERVAL` is how often changes are uploaded, so it's the most data you can lose. `SNAPSHOT_RETENTION` is how far back you can restore (units: `s`, `m`, `h` only).

Shared config is in `docker-compose.base.yml`. Per-environment values are hardcoded in each env file.

## Environment variables

Set these in the Coolify UI:

- `R2_ACCESS_KEY_ID`
- `R2_SECRET_ACCESS_KEY`
- `R2_ENDPOINT`: `https://<account_id>.r2.cloudflarestorage.com`

## Deploying with Coolify

1. Add a new resource, connect this repo, and choose the **Docker Compose** build pack.
2. Set the compose file path to the environment's file.
3. Set the environment variables.
4. Deploy.

### Coolify workarounds

- **Hardcoded bind mount paths.** Since v4.0.0-beta.441, Coolify treats `${VAR}` in a bind mount source as a named volume ([#7127](https://github.com/coollabsio/coolify/issues/7127)), so each environment has its own compose file with the path written out.
- **Repeated `environment:` blocks.** Coolify only scans the selected compose file for `${VAR}` references, not the `extends` base, so each env file lists the credentials itself.

## Restoring

### On a new host

At startup, Litestream restores `data.db` from R2 if the file doesn't exist. If PocketBase starts first, it creates an empty `data.db`, the restore is skipped, and the empty database is backed up to the same R2 path. So:

1. Make sure the data directory (e.g. `/data/production`) exists and has no `data.db`.
2. Deploy Litestream with PocketBase stopped.
3. Wait for `attempting restore before replication` in the logs, with no `restore failed` after it.
4. Start PocketBase.

### To a point in time

```bash
# Restore a copy from inside the running Litestream container (UTC timestamp)
docker exec <litestream-container> litestream restore -config /tmp/litestream.yml \
  -timestamp 2026-11-10T12:00:00Z -integrity-check quick \
  -o /data/restored.db /data/data.db

# Stop PocketBase and Litestream, then on the host:
cd /data/production
mkdir pre-restore && mv data.db* pre-restore/
rm -rf .data.db-litestream
mv restored.db data.db

# Start Litestream, then PocketBase.
```
