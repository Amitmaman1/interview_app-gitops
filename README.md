# GitOps Configuration

ArgoCD configs for deploying the interview app across dev, staging, and prod environments. This is the single source of truth for what's running in the cluster.

## Structure

```
gitops/
├── applicationset.yaml      # ArgoCD ApplicationSet
└── environments/
    ├── dev/
    │   └── values.yaml     # Dev-specific Helm values
    ├── staging/
    │   └── values.yaml     # Staging values
    └── prod/
        └── values.yaml     # Production values
```

## How it works

The `applicationset.yaml` tells ArgoCD to:
1. Pull the Helm chart from the main app repo
2. Use environment-specific values from this repo
3. Deploy to the appropriate namespace
4. Auto-sync on any Git changes

## Environments

- **dev**: `killer-app-dev` namespace → devopsinterviewer.online
- **staging**: `killer-app-staging` namespace (currently disabled in ApplicationSet)
- **prod**: `killer-app-prod` namespace (currently disabled in ApplicationSet)

## Making changes

Just edit the `values.yaml` for your target environment and push. ArgoCD picks it up automatically and syncs to the cluster.

### Example: Update image tag
```yaml
backend:
  image:
    tag: backend-abc123
```

Push it, and ArgoCD will roll out the new version.

## Secrets

We don't store secrets here. Everything sensitive comes from AWS Secrets Manager via External Secrets Operator:
- Supabase credentials
- Groq API key

The secrets config is in each environment's `values.yaml` but just references the AWS secret names.

## Enabling staging/prod

Uncomment the staging/prod sections in `applicationset.yaml`:
```yaml
- env: staging
  namespace: killer-app-staging
```

Make sure the EKS cluster and secrets are set up first (see infra repo).

## Related Repos

- **Infrastructure**: [interview_app-infra](https://github.com/Amitmaman1/interview_app-infra) - Terraform for EKS/VPC/monitoring
- **Application**: [interview_app](https://github.com/Amitmaman1/interview_app) - The app code + Helm chart
