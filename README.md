# IOWAP

**Infrastructure · Offloading · Workload Assignment Platform**  
*The Distributed Node Capability Framework*

---

We only wanted a picture. We got a distributed orchestrator.
I only want a picture. I built a capability mesh.

IOWAP is a trust-based capability matcher for distributed nodes — a dumb coordinator that matches capabilities and routes workloads, but never orchestrates the details. Nodes claim what they can do, the relay connects them.

```
iowap-org/
├── iowap              ← this repo (meta: story + architecture)
├── iowap-server       ← API, scheduler, auth, DB, dashboard, docs
├── iowap-node         ← node framework: daemon, CLI, relay_client, handler_runner
└── iowap-storage      ← reference storage node implementation (docker)
```

---

## Repos

| Repo | Description | Commits |
|------|-------------|---------|
| [iowap-server](https://github.com/iowap-org/iowap-server) | The relay server — API, scheduler, auth, dashboard | 163 |
| [iowap-node](https://github.com/iowap-org/iowap-node) | Node framework — build your own node (daemon + CLI) | 73 |
| [iowap-storage](https://github.com/iowap-org/iowap-storage) | Reference storage-node implementation (Docker) | 15 |

## Quick Start

```bash
# 1. Start the relay server
docker run -d --name iowap-server -p 8080:8080 \
  ghcr.io/iowap-org/iowap-server:latest

# 2. Run a node (example: storage)
docker run -d --name iowap-storage \
  -e RELAY_URL=http://host.docker.internal:8080 \
  ghcr.io/iowap-org/iowap-storage:latest

# 3. Submit a task
curl -X POST http://localhost:8080/relay/v2/scheduler/task-simple \
  -H "Authorization: Bearer <node-token>" \
  -H "Content-Type: application/json" \
  -d '{"capability": "storage.archive", "payload": {"path": "/data"}}'
```

## Architecture

```
┌──────────┐    heartbeat/capabilities    ┌──────────┐
│  NODE A  │ ◄───────────────────────────► │  RELAY   │
│ (GPU)    │                               │  SERVER  │
└──────────┘    claim/submit tasks         └────┬─────┘
┌──────────┐                                   │
│  NODE B  │ ◄─────────────────────────────────┤
│ (STORAGE)│                                   │
└──────────┘                                   │
┌──────────┐                                   │
│  NODE C  │ ◄─────────────────────────────────┤
│ (CUSTOM) │                                   │
└──────────┘                                   │
                                    ┌──────────┴──────────┐
                                    │  DASHBOARD (iFrame)  │
                                    └─────────────────────┘
```

## License

AGPL-3.0