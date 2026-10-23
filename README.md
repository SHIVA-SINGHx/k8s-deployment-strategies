# Kubernetes Deployment Strategies

![Kubernetes](https://img.shields.io/badge/Kubernetes-Kind-326CE5?logo=kubernetes&logoColor=white)
![kubectl](https://img.shields.io/badge/kubectl-Rollouts-326CE5?logo=kubernetes&logoColor=white)
![YAML](https://img.shields.io/badge/Manifests-YAML-CB171E?logo=yaml&logoColor=white)

Hands-on implementations of the four most common Kubernetes deployment strategies (**Recreate**, **Rolling Update**, **Blue-Green**, and **Canary**), each with ready-to-run manifests on a local Kind cluster.

📖 Full walkthrough: [Kubernetes Deployment Strategies Explained with Hands-On Examples](https://medium.com/@shivasingh.1dev/kubernetes-deployment-strategies-explained-with-hands-on-examples-2c120c191426)

---

## Why This Project

Releasing a new version of a running application raises real questions: Can users stay connected during the update? How fast can I roll back if something breaks? How much extra infrastructure will it cost? Each strategy answers these differently. This repository shows how each one works in practice so the trade-offs can be compared side by side.

---

## Strategies at a Glance

| Strategy | How it works | Downtime | Rollback | Extra resources | Best for |
|----------|--------------|----------|----------|-----------------|----------|
| [Recreate](Recreate-deployment) | Stops all old pods, then starts new ones | Yes | Redeploy old version | None | Dev/test, apps that can't run two versions at once |
| [Rolling Update](Rolling-Update-Deployment) | Replaces pods gradually, a few at a time | No | `kubectl rollout undo` | Low | Most stateless production apps (Kubernetes default) |
| [Blue-Green](Blue-green-deployment) | Runs old (blue) and new (green) side by side, then switches traffic | No | Instant, switch traffic back | High (2x) | Releases that need instant rollback |
| [Canary](Canary-deployment) | Sends a small share of traffic to the new version first | No | Remove the canary | Low to medium | Validating risky changes on real traffic |

A detailed comparison is available in [deployment-strategies-comparison.md](deployment-strategies-comparison.md).

---

## How Each Strategy Works

**Recreate:** All existing pods are terminated before the new version starts. Simple and predictable, but the application is unavailable between the two steps.

**Rolling Update:** Kubernetes replaces pods incrementally, controlled by `maxSurge` and `maxUnavailable`. The app stays available throughout, and a bad release can be reverted with a single rollout command.

**Blue-Green:** Two complete environments run in parallel. Traffic is routed to the live (blue) version through a Service. Once the new (green) version is verified, the Service selector is switched, and rollback is just switching it back.

**Canary:** The new version runs alongside the stable one and receives only a small portion of traffic. If it behaves well, it is scaled up and the old version is retired; if not, it is removed with minimal user impact.

---

## Repository Structure

```
.
├── Recreate-deployment/         # Recreate strategy manifests
├── Rolling-Update-Deployment/   # Rolling update manifests
├── Blue-green-deployment/       # Blue and green deployments + Service
├── Canary-deployment/           # Stable and canary deployments
├── Simple-Canary-Example/       # Minimal canary example
├── kind-cluster/                # Cluster setup scripts
├── kind-config.yml              # Kind cluster configuration
└── deployment-strategies-comparison.md
```

---

## Getting Started

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

### 1. Create the cluster

```bash
kind create cluster --config kind-config.yml
kubectl get nodes
```

### 2. Try a strategy

```bash
# Replace the folder name with the strategy you want to try
kubectl apply -f Rolling-Update-Deployment/
kubectl get pods -w
```

### 3. Trigger an update and observe

```bash
kubectl set image deployment/<deployment-name> <container-name>=<image>:<new-tag>
kubectl rollout status deployment/<deployment-name>
```

### 4. Roll back

```bash
kubectl rollout undo deployment/<deployment-name>
```

### Blue-Green: switch traffic

```bash
kubectl patch service <service-name> \
  -p '{"spec":{"selector":{"app":"<app-name>","version":"green"}}}'
```

### Cleanup

```bash
kind delete cluster
```

---

## Choosing a Strategy

- **Need the simplest setup and can accept downtime?** Recreate
- **Want zero downtime with minimal cost?** Rolling Update
- **Need instant rollback and can afford double the resources?** Blue-Green
- **Want to test a risky change on a small slice of users first?** Canary

---

## What I Learned

- How Deployments, ReplicaSets, and Services work together during a release
- How `maxSurge` and `maxUnavailable` control the pace of a rolling update
- Using label selectors on Services to route and switch traffic
- Trade-offs between availability, rollback speed, and infrastructure cost

---

## Author

**Shiva Singh** · [GitHub](https://github.com/SHIVA-SINGHx) · [Medium](https://medium.com/@shivasingh.1dev)
