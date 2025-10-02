# Echo App Helm Chart

A simple echo server application for demonstrating ArgoCD capabilities with Helm charts.

## Features

- Uses `jmalloc/echo-server` image
- Mounts a ConfigMap with application configuration
- Mounts a Secret with sensitive credentials
- Includes validation that both config and secret exist before starting
- Health checks (liveness and readiness probes)
- Environment variables from both ConfigMap and Secret

## Components

- **Namespace**: echo-app
- **ConfigMap**: echo-app-config (templated from values.yaml)
- **Secret**: echo-app-secret (templated from values.yaml)
- **Deployment**: echo-app (validates config and secret on startup)
- **Service**: echo-app-service (ClusterIP on port 8080)

## Mounted Volumes

- **ConfigMap** mounted at `/config`:
  - `config.json` - Application configuration
  - `message` - Welcome message
  - `environment` - Environment name

- **Secret** mounted at `/secrets`:
  - `secret-config.yaml` - Credentials configuration
  - `api-key` - API authentication key
  - `database-password` - Database password

## Environment Variables

From ConfigMap:
- `CONFIG_MESSAGE`
- `ENVIRONMENT`

From Secret:
- `API_KEY`
- `DATABASE_PASSWORD`

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

### Option 1: Break ConfigMap reference
Edit `templates/deployment.yaml` line 59:
```yaml
# Change:
name: {{ .Chart.Name }}-config
# To:
name: {{ .Chart.Name }}-config-BROKEN
```

### Option 2: Break Secret reference
Edit `templates/deployment.yaml` line 88:
```yaml
# Change:
secretName: {{ .Chart.Name }}-secret
# To:
secretName: {{ .Chart.Name }}-secret-BROKEN
```

### Option 3: Remove required config key
In `values.yaml`, delete the `message` field under `config`

### Option 4: Remove required secret key
In `values.yaml`, delete the `apiKey` field under `secret`

Then commit and push. ArgoCD will sync the broken config, pods will crash during postStart validation, and you can demonstrate rollback.

## Testing Locally

```bash
# Install with helm (optional - ArgoCD can do this)
helm install echo-app ./echo-app -n echo-app --create-namespace

# Check status
kubectl get pods -n echo-app
kubectl logs -n echo-app -l app=echo-app

# View secret (base64 encoded)
kubectl get secret echo-app-secret -n echo-app -o yaml

# Access the service
kubectl port-forward -n echo-app svc/echo-app-service 8080:8080
curl http://localhost:8080
```

## Configuration

All configuration is in `values.yaml`:
- `replicaCount`: Number of replicas
- `image`: Container image settings
- `service`: Service configuration
- `config`: ConfigMap data (mounted at /config)
- `secret`: Secret data (mounted at /secrets)

**Note:** In production, never commit real secrets to Git. Use tools like:
- Sealed Secrets
- External Secrets Operator
- Vault
- SOPS
