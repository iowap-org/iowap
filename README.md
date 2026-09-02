# IOWAP

**Infrastructure · Offloading · Workload Assignment Platform**  
*The Distributed Node Capability Framework*

[![Tests & Linting](https://github.com/iowap-org/iowap/actions/workflows/test-lint.yml/badge.svg)](https://github.com/iowap-org/iowap/actions/workflows/test-lint.yml)
[![Security Scan](https://github.com/iowap-org/iowap/actions/workflows/security-scan.yml/badge.svg)](https://github.com/iowap-org/iowap/actions/workflows/security-scan.yml)
[![Docs Validation](https://github.com/iowap-org/iowap/actions/workflows/docs-validate.yml/badge.svg)](https://github.com/iowap-org/iowap/actions/workflows/docs-validate.yml)

---

I only want a picture. I built a capability mesh.

The origin is embarrassingly simple. I wanted to generate an AI image on my Mac
Mini — it has the GPU, it runs MLX/mflux. But my AI agent doesn't live on the
Mac. It runs in a VM on my Proxmox server. So every time I asked for a picture,
the agent had to reach across the network to a machine that wasn't its own.

That gap — *the thing that thinks* and *the thing that does* living on different
boxes — is the whole reason IOWAP exists. Not a grand architecture. Just: the
Mac can generate images, the VM can't, and I wanted the VM to be able to ask the
Mac to do it.

So I built a dumb coordinator. Nodes claim what they can do, the relay connects
them, and workloads get routed to whatever node actually has the capability.
No orchestration, no micro-managing — just matching capability to task.

For a picture. I built a whole platform and never stopped.

```text
iowap-org/
├── iowap              ← this repo (meta: story + architecture)
├── iowap-server       ← relay server: API, scheduler, auth, DB, dashboard
├── iowap-node         ← node framework: daemon, CLI, capability management, handler runner
├── iowap-storage      ← reference storage node (docker)
├── iowap-ha           ← Home Assistant: HAOS app (node container) + thin integration
├── iowap-federation   ← federation node: bridge capabilities between relays (E2EE)
├── iowap-docker       ← docker images: base, server, storage
└── iowap-docs         ← documentation: setup, concepts, API reference
```

---

## Repos

| Repo | Description |
| ---- | ----------- |
| [iowap](https://github.com/iowap-org/iowap) | Meta-repo — story, architecture, links (this repo) |
| [iowap-server](https://github.com/iowap-org/iowap-server) | Relay server — API, scheduler, auth, database, dashboard for node orchestration |
| [iowap-node](https://github.com/iowap-org/iowap-node) | Node framework — daemon, CLI, capability management, handler runner. Build and register your own node |
| [iowap-storage](https://github.com/iowap-org/iowap-storage) | Storage node — file storage, retrieval, and bridging for the IOWAP ecosystem |
| [iowap-ha](https://github.com/iowap-org/iowap-ha) | Home Assistant — HAOS app (node container) + thin custom integration for submissions |
| [iowap-federation](https://github.com/iowap-org/iowap-federation) | Federation node — bridge capabilities between relays. Inbox/Outbox, transport-agnostic, E2EE |
| [iowap-docker](https://github.com/iowap-org/iowap-docker) | Docker images — base, server & storage |
| [iowap-docs](https://github.com/iowap-org/iowap-docs) | Documentation — setup, concepts, API reference, node guides |

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

Full setup, API reference and node guides live in [iowap-docs](https://github.com/iowap-org/iowap-docs).

## Architecture

```text
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