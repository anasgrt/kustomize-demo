# Kustomize Multi-Directory Demo

This repo demonstrates **managing multiple directories** with Kustomize.

## Directory Structure

```
k8s/
├── kustomization.yaml          # ROOT: references all subdirs
├── api/
│   ├── kustomization.yaml      # Lists api resources
│   ├── api-deployment.yaml
│   └── api-service.yaml
├── db/
│   ├── kustomization.yaml      # Lists db-depl.yaml, db-service.yaml
│   ├── db-depl.yaml            # Named to match diagram exactly
│   └── db-service.yaml
├── cache/
│   ├── kustomization.yaml
│   ├── cache-deployment.yaml
│   └── cache-service.yaml
└── kafka/
    ├── kustomization.yaml
    ├── kafka-deployment.yaml
    └── kafka-service.yaml
```

## Key Concepts

### 1. Hierarchical Kustomization
The root `kustomization.yaml` references subdirectories:
```yaml
resources:
  - api/
  - db/
  - cache/
  - kafka/
```

Each subdirectory **must** have its own `kustomization.yaml`.

### 2. Essential Commands

```bash
# Preview the merged output (DRY RUN)
kubectl kustomize k8s/

# Apply directly
kubectl apply -k k8s/

# Apply a single component
kubectl apply -k k8s/db/

# Delete all resources
kubectl delete -k k8s/

# View what would be applied with diff
kubectl diff -k k8s/
```

### 3. Common Labels Propagation
Labels defined in root are applied to ALL resources:
```yaml
# In k8s/kustomization.yaml
commonLabels:
  environment: dev
```
This adds `environment: dev` to every Deployment, Service, and Pod.

### 4. Namespace Override
```yaml
# In k8s/kustomization.yaml
namespace: myapp
```
All resources are created in `myapp` namespace regardless of what's in individual files.

## CKA Exam Tips

1. **Speed**: `kubectl apply -k .` is faster than `kubectl apply -f .` with multiple files
2. **Exam Environment**: Kustomize is built into kubectl (v1.14+), no installation needed
3. **Common Task**: "Create resources from the kustomize directory at /path" → `kubectl apply -k /path`
4. **Verify First**: Always run `kubectl kustomize /path` before applying to see merged output

## Testing Locally

```bash
# Create namespace first
kubectl create namespace myapp

# Apply all resources
kubectl apply -k k8s/

# Verify
kubectl get all -n myapp

# Check labels were applied
kubectl get pods -n myapp --show-labels
```

## Extending This Example

### Add Overlays (dev/staging/prod)
```
k8s/
├── base/                    # Move current structure here
│   ├── kustomization.yaml
│   └── ...
└── overlays/
    ├── dev/
    │   └── kustomization.yaml   # resources: [../../base]
    ├── staging/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

### Add ConfigMap Generator
```yaml
# In any kustomization.yaml
configMapGenerator:
  - name: app-config
    literals:
      - LOG_LEVEL=debug
      - DB_HOST=postgres
```

### Add Secret Generator
```yaml
secretGenerator:
  - name: db-secret
    literals:
      - password=mypassword
```
