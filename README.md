# Actions Deployment Repository

This repository contains Kubernetes deployment manifests for the `actions` application.

## Repository Structure

```
.
├── base/                    # Base Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── namespace.yaml
│   └── kustomization.yaml
├── environments/            # Environment-specific overlays
│   ├── dev/
│   │   └── kustomization.yaml
│   ├── staging/
│   │   └── kustomization.yaml
│   └── prod/
│       ├── kustomization.yaml
│       └── namespace.yaml
└── argocd/                  # ArgoCD Application definitions
    ├── dev-app.yaml
    ├── staging-app.yaml
    └── prod-app.yaml
```

## Environments

### Dev
- **Namespace**: `actions-dev`
- **Replicas**: 2
- **Image**: Automatically updated by CI/CD
- **ArgoCD**: Auto-sync enabled

### Staging
- **Namespace**: `actions-staging`
- **Replicas**: 2
- **Image**: Manually promoted from dev
- **ArgoCD**: Auto-sync enabled

### Prod
- **Namespace**: `actions-prod`
- **Replicas**: 3
- **Image**: Manually promoted from staging
- **ArgoCD**: Auto-sync enabled

## Version Management

Image versions are managed in `environments/<env>/kustomization.yaml`:

```yaml
images:
  - name: shinobislayer/actions
    newName: shinobislayer/actions
    newTag: 2025.12.015-MAIN  # Version managed here
```

## Deployment Flow

1. **Code pushed** to [actions](https://github.com/ShinobiSlayer/actions) repo
2. **GitHub Actions** builds Docker image with version tag
3. **GitHub Actions** updates `environments/dev/kustomization.yaml` with new version
4. **ArgoCD** detects change and deploys to dev automatically
5. **Manual promotion** to staging/prod by updating their kustomization.yaml

## Manual Promotion Example

To promote version `2025.12.015-MAIN` from dev to staging:

```bash
# Update staging kustomization
sed -i 's/newTag: .*/newTag: 2025.12.015-MAIN/' environments/staging/kustomization.yaml

git add environments/staging/kustomization.yaml
git commit -m "Promote 2025.12.015-MAIN to staging"
git push
```

ArgoCD will automatically sync the change.
