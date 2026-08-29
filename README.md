# gitops-pipeline-config

The desired-state repo for [`gitops-pipeline-app`](https://github.com/AnzalKhan16/gitops-pipeline-app). This repo contains **only Kubernetes manifests** — no application code — and is the single source of truth ArgoCD watches to keep the cluster in sync.

## How this repo gets updated

Nothing here is edited by hand in normal operation. On every push to `gitops-pipeline-app`, CI:

1. Builds and scans a new Docker image
2. Pushes it to GitHub Container Registry, tagged with the commit SHA
3. Checks out this repo, rewrites the `image:` line in `k8s/deployment.yaml` to point at that new tag
4. Commits the change as `github-actions[bot]` and pushes

That commit is the entire handoff from CI to CD — nothing in the build pipeline ever runs `kubectl` or holds cluster credentials.

## How this repo gets applied

[ArgoCD](https://argo-cd.readthedocs.io/), running inside the target Kubernetes cluster, continuously watches this repo (path: `k8s/`). It's configured with:

- **Automatic sync** — no manual "Sync" click needed
- **Self-heal** — reverts any manual `kubectl` changes made directly against the cluster, so Git always wins
- **Prune** — removes cluster resources if their manifest is deleted here

## Contents

| File | Purpose |
|---|---|
| `k8s/deployment.yaml` | Deployment spec — replicas, image, readiness/liveness probes |
| `k8s/service.yaml` | ClusterIP Service exposing the app inside the cluster |

## Manual apply (bypassing ArgoCD, for local testing only)

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

In normal operation this is never needed — ArgoCD handles it.

Author - Mohammad Anzal Khan
