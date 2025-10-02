# Echo App Helm Chart

A simple echo server application for demonstrating ArgoCD capabilities with Helm charts.

## Features

- Uses `jmalloc/echo-server` image
- Mounts a ConfigMap with application configuration
- Includes validation that config exists before starting
- Health checks (liveness and readiness probes)

## Components

- **Namespace**: echo-app
- **ConfigMap**: echo-app-config (templated from values.yaml)
- **Deployment**: echo-app (validates config on startup)
- **Service**: echo-app-service (ClusterIP on port 8080)

## Deploy with ArgoCD

```bash
argocd app create echo-app \
  --repo https://github.com/YOUR_USERNAME/YOUR_REPO.git \
  --path echo-app \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace echo-app \
  --sync-policy manual

argocd app sync echo-app
```

## Breaking the App for Rollback Demo

To simulate a broken deployment, edit `values.yaml` and change:

```yaml
# Break Option 1: Invalid ConfigMap reference
# In templates/deployment.yaml line 42, change:
name: {{ .Chart.Name }}-config-BROKEN

# Break Option 2: Remove required config key
# In values.yaml, delete the 'message' field
```

Then commit and push. ArgoCD will sync the broken config, pods will crash, and you can demonstrate rollback.

## Testing Locally

```bash
# Install with helm (optional - ArgoCD can do this)
helm install echo-app ./echo-app -n echo-app --create-namespace

# Check status
kubectl get pods -n echo-app
kubectl logs -n echo-app -l app=echo-app

# Access the service
kubectl port-forward -n echo-app svc/echo-app-service 8080:8080
curl http://localhost:8080
```

## Configuration

All configuration is in `values.yaml`:
- `replicaCount`: Number of replicas
- `image`: Container image settings
- `service`: Service configuration
- `config`: ConfigMap data (this is what gets mounted)
