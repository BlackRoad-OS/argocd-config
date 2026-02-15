# BlackRoad ArgoCD Configuration

**GitOps for sovereign infrastructure**

## Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

## Applications

### Root App of Apps
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: blackroad-apps
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/BlackRoad-OS/argocd-config
    targetRevision: main
    path: apps
  destination:
    server: https://kubernetes.default.svc
    namespace: blackroad-system
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Structure

```
apps/
├── infrastructure/    # Core infra apps
│   ├── prometheus.yaml
│   ├── grafana.yaml
│   └── cert-manager.yaml
├── ai/               # AI workloads
│   ├── vllm.yaml
│   ├── ollama.yaml
│   └── hailo.yaml
└── services/         # Business services
    ├── api.yaml
    └── dashboard.yaml
```

## Sync Waves

| Wave | Components |
|------|------------|
| -10 | CRDs |
| -5 | Namespaces |
| 0 | Infrastructure |
| 5 | Services |
| 10 | AI Workloads |

## Notifications

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
data:
  trigger.on-sync-succeeded: |
    - send: [slack]
  template.slack: |
    message: "{{.app.metadata.name}} synced to {{.app.status.sync.revision}}"
```

---

*BlackRoad OS - GitOps Sovereignty*
