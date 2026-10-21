# Kubernetes Cluster Build — First Production Cluster

> Building the first production Kubernetes cluster from scratch.

## Problem

All workloads ran on individual DigitalOcean droplets with no orchestration, no shared ingress, and no centralized container registry. Each service was its own island — separate TLS, separate deployments, separate monitoring.

## What I Built

The organization's first production Kubernetes cluster, from provisioning through first workload.

### Cluster

- **DOKS** (DigitalOcean Kubernetes) on v5 infrastructure in ATL1
- **Node pool**: s5-4vcpu-8gb Shared AMD (sized after s-2vcpu-4gb variants had insufficient capacity)
- **Kubernetes**: v1.37.0

### Ingress & TLS

- **Traefik** v3.7 as ingress controller (chosen over ingress-nginx, which was retired upstream)
- **cert-manager** v1.21 with Let's Encrypt production ClusterIssuer
- **IngressRoutes** for per-service routing and TLS termination

### Container Registry

- **Harbor** at `registry.weown.tools` — self-hosted registry replacing DOCR
- Migrated all production images from DOCR to Harbor
- Images organized under the `weown` project

### Workloads Deployed

- **weown.events** (WordPress) — first production workload on the cluster
- **Buzz multi-agent platform** (Postgres, Redis, MinIO) — in progress
- Namespace isolation between workloads

## Architecture

```
Internet
    │
    ▼
┌───────────────────────────────────────────────────┐
│  DOKS Cluster (ATL1, v5)                          │
│                                                   │
│  ┌────────────────────────────────────────────┐   │
│  │ Traefik Ingress Controller                 │   │
│  │ (TLS via cert-manager + Let's Encrypt)     │   │
│  └──────┬──────────────┬──────────────────────┘   │
│         │              │                          │
│  ┌──────▼──────┐ ┌─────▼───────────────────┐     │
│  │ ns: weown-  │ │ ns: buzz                │     │
│  │ events      │ │                         │     │
│  │             │ │ Postgres  Redis  MinIO   │     │
│  │ WordPress   │ │ Buzz relay               │     │
│  │ MariaDB     │ │                         │     │
│  └─────────────┘ └─────────────────────────┘     │
│                                                   │
│  registry.weown.tools (Harbor)                    │
│  ├── weown/anythingllm:v1.14.1                   │
│  ├── weown/caddy:2                                │
│  ├── weown/keycloak:26.7.0                       │
│  ├── weown/openclaw:2026.6.1                     │
│  ├── weown/buzz:main                              │
│  └── weown/minio:*                                │
└───────────────────────────────────────────────────┘
```

## Decisions

| Decision | Why |
|---|---|
| Traefik over ingress-nginx | ingress-nginx retired upstream; Traefik is the org's standard |
| Harbor over DOCR | Self-hosted, more control, organized by project; DOCR was Basic tier (5GB limit) |
| cert-manager + LE | Automatic TLS for every IngressRoute; no manual cert management |
| Single large node over multiple small | s-2vcpu-4gb variants had no capacity in ATL1; one s5-4vcpu-8gb is simpler to start |
| Namespace isolation | Workloads can't reach each other's services without explicit policy |

## Outcome

First workload (`weown.events`) is live on the cluster. Multi-agent platform deployment in progress.

## Stack

`Kubernetes` · `DOKS` · `Traefik` · `cert-manager` · `Harbor` · `DigitalOcean` · `Helm` · `Let's Encrypt`

## Context

Built as Platform Engineer at [WeOwn](https://weown.net) (2026). Source code is proprietary.
