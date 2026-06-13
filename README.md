# milvus-coolify — self-hosted Milvus 3.0 vector store + Attu (Coolify-deployable)

> _Byline: Claude Code · Opus 4.8 · 2026-06-13_

Coolify-friendly **Milvus 3.0 standalone (embedded)** stack + the **Attu v3** admin GUI.
The shared semantic-search backend for:

1. **Agno platform** code + knowledge indexing (via `claude-context`), and
2. the **Case Bible** corpus.

Replaces the managed **Zilliz `aws-eu-central-1` serverless** cluster — off the EU region,
onto our own OVH box, on mapped volumes we can back up. Decision of record: **ADR-0026** in
the `Agno-MCP-Platform` repo.

## What's in the box (light embedded design)

| Service | Image | Exposed via | Notes |
|---|---|---|---|
| `milvus` | `milvusdb/milvus:v3.0-beta` | Traefik h2c → gRPC :19530 | **Embedded etcd + local storage + WoodPecker WAL** — no separate etcd/MinIO/Pulsar |
| `attu` | `zilliztech/attu:v3.0.0-beta.6` | Traefik http → :3000 | Admin GUI (requires Milvus 3.x) |

Everything persists to **mapped (bind-mount)** volumes under `${DOCKER_VOLUME_DIRECTORY}/volumes/*`
(`milvus/` holds embedded etcd + local segments + woodpecker WAL; `attu/` holds Attu's SQLite).
Set `DOCKER_VOLUME_DIRECTORY` to a stable, backed-up host path. (Owner preference: mapped volumes.)

Config files mounted into Milvus: `embedEtcd.yaml` (embedded etcd) and `user.yaml`
(auth ON + WoodPecker pinned to local storage).

## Deploy via Coolify

1. **New Resource → Docker Compose → Repository**, point it at this repo.
2. Set env vars (Coolify UI or `.env`) — see `.env.example`:
   - `MILVUS_DOMAIN` — domain for Milvus gRPC (e.g. `milvus.yourdomain.com`); DNS A-record → OVH box. Traefik provisions the Let's Encrypt cert.
   - `ATTU_DOMAIN` — domain for the Attu UI.
   - `DOCKER_VOLUME_DIRECTORY` — host path for mapped volumes (e.g. `/data/milvus`).
3. **Deploy.** First Milvus boot takes ~60–90s (healthcheck has a 90s start period).

### Traefik / gRPC note

Milvus's SDK port (19530) is **gRPC over h2c** (cleartext HTTP/2), not plain HTTP. The
compose labels use `loadbalancer.server.scheme=h2c` so Traefik terminates TLS at the edge
and forwards h2c — clients connect over `https://<MILVUS_DOMAIN>` (gRPC-TLS on 443), the
same URL shape as the old Zilliz endpoint. Attu is plain HTTP on 3000.

To reach Milvus over **Tailscale/LAN** instead, uncomment the `ports:` block and lock 19530
to the tailnet/firewall.

## Point claude-context at it

After deploy, edit `~/.context/.env` (the only change needed to leave Zilliz EU):

```ini
MILVUS_ADDRESS=https://milvus.yourdomain.com
MILVUS_TOKEN=root:<your-new-password>
```

Then `/mcp` reconnect claude-context and re-index. Embedder config (OpenRouter +
`mistralai/codestral-embed-2505`, 1536-d) is unchanged.

## Security

- **Auth is ON** (`user.yaml`). Default credential `root` / `Milvus`. **Change on first boot:**

  ```python
  from pymilvus import MilvusClient
  c = MilvusClient(uri="https://milvus.yourdomain.com", token="root:Milvus")
  c.update_password(user_name="root", old_password="Milvus", new_password="<strong-pass>")
  ```

- Attu is an admin UI — it still requires the Milvus token to log in, but consider adding a
  Traefik basic-auth middleware in front of `ATTU_DOMAIN` for defense in depth.
- Isolate corpora with separate Milvus **databases/collections** (code-index vs Case Bible).

## Backups

Data lives on mapped host paths: back up `${DOCKER_VOLUME_DIRECTORY}/volumes/` (milvus + attu).
Stop `milvus` for a consistent cold snapshot, or use Attu's backup/restore for hot backups.

## Versions

`v3.0-beta` Milvus + `v3.0.0-beta.6` Attu are a **matched pair** (Attu v3 supports Milvus 3.x
only). Bump deliberately and re-test `claude-context` connectivity after any change.
