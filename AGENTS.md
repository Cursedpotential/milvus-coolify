# milvus-coolify — Agent Entry Point

> _Byline: Codex · GPT-5 · 2026-08-29._

This is an independent infrastructure repository. Its Git root must resolve exactly to
`E:/AI_Workspace/Projects/the-platform-workspace/milvus-coolify`. Stage only explicit files in this
repository; never stage its parent raw Gitlink incidentally.

## Current role

The repository retains the Coolify-facing Milvus/Attu configuration and historical operational
knowledge. The Evidence Platform has completed its Weaviate cutover and Milvus is deliberately
parked/down. Do not deploy, start, restart, migrate, or repoint clients to this stack unless the
owner explicitly authorizes reactivation and the current Platform decisions are reverified.

`docker-compose.yml` is production-facing infrastructure source. Do not run a duplicate local
Docker/Podman/Compose stack. A static `docker compose config` check does not prove a Coolify
deployment or live service.

## Source map

- `docker-compose.yml` — Coolify service topology and host bind mounts.
- `embedEtcd.yaml` — embedded-etcd timing configuration.
- `user.yaml` — Milvus authentication/storage configuration.
- `.env.example` — variable names only; never populate or commit secrets.
- `README.md` — historical deployment/operator guide; verify it against current Platform decisions
  before treating activation guidance as current.

Never hard-delete. Move approved removals to this repository's `to_be_deleted/` directory; only the
owner permanently deletes from quarantine. Preserve unrelated work and verify live infrastructure
state before making operational claims.
