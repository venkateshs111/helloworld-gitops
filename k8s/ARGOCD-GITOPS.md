# ArgoCD GitOps Setup Guide

## What is ArgoCD GitOps?

GitOps = Your Git repository is the single source of truth for your infrastructure.

**Flow:**
```
Developer Push Code → Git Repo → ArgoCD Watches → Auto Deploys to K8s
```

## Step 1: Create GitHub Repository

1. Go to github.com and create new repo: `helloworld-gitops`
2. Clone it locally:
```bash
git clone https://github.com/YOUR-USERNAME/helloworld-gitops
cd helloworld-gitops
```

3. Create directory structure:
```bash
mkdir -p k8s
```

4. Copy manifests:
```bash
# Copy deployment.yaml to k8s/
cp k8s/deployment.yaml k8s/

# Copy other manifests if needed
# kustomization.yaml (optional, for Kustomize)
```

5. Push to GitHub:
```bash
git add .
git commit -m "Add Kubernetes manifests"
git push origin main
```

## Step 2: Create ArgoCD Application

The Application tells ArgoCD what to deploy:

```bash
# Replace YOUR-USERNAME with your GitHub username
kubectl apply -f - << EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: helloworld-gitops
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://github.com/YOUR-USERNAME/helloworld-gitops
    targetRevision: main
    path: k8s
  
  destination:
    server: https://kubernetes.default.svc
    namespace: gitops-apps
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
EOF
```

## Step 3: Verify ArgoCD Synced

Check Application status:
```bash
kubectl get applications -n argocd
kubectl get application helloworld-gitops -n argocd -o yaml
```

Check deployed resources:
```bash
kubectl get pods -n gitops-apps
kubectl get svc -n gitops-apps
```

Access the app:
```bash
# Port-forward
kubectl port-forward svc/helloworld 8765:80 -n gitops-apps

# Or via NodePort
# http://localhost:30555
```

## Step 4: Enable ArgoCD UI (Optional)

Port-forward to ArgoCD server:
```bash
kubectl port-forward svc/argocd-server 7777:443 -n argocd
```

Access: https://localhost:7777 (admin/password from secret)

## Step 5: Make Changes (GitOps in Action)

### Change 1: Update replica count

Edit `k8s/deployment.yaml`:
```yaml
replicas: 3  # Changed from 2
```

Push to GitHub:
```bash
git add k8s/deployment.yaml
git commit -m "Scale to 3 replicas"
git push origin main
```

ArgoCD will automatically:
1. Detect the change in Git
2. Sync the new manifest
3. Scale to 3 replicas

Verify:
```bash
kubectl get pods -n gitops-apps
```

### Change 2: Update image

Edit `k8s/deployment.yaml`:
```yaml
image: nginx:1.24  # Specify version instead of latest
```

Push → ArgoCD detects → Auto-updates pods

## Step 6: Manual Sync (if auto-sync disabled)

```bash
argocd app sync helloworld-gitops
```

Or via kubectl:
```bash
kubectl patch app helloworld-gitops -n argocd --type merge -p '{"metadata":{"finalizers":["resources-finalizer.argocd.argoproj.io"],"annotations":{"argocd.argoproj.io/refresh":"hard"}}}'
```

## Monitoring & Status

Check sync status:
```bash
kubectl get application helloworld-gitops -n argocd
kubectl describe application helloworld-gitops -n argocd
```

Watch real-time:
```bash
kubectl get application helloworld-gitops -n argocd -w
```

View ArgoCD logs:
```bash
kubectl logs deployment/argocd-application-controller -n argocd -f
```

## Repository Structure

```
helloworld-gitops/
├── k8s/
│   ├── deployment.yaml       # ConfigMap + Deployment + Service
│   ├── kustomization.yaml    # (Optional) For Kustomize
│   └── README.md
├── .gitignore
└── README.md
```

## ArgoCD Features

✅ **Automated Sync** - Changes in Git auto-deploy
✅ **Self-Healing** - Restores desired state if manual changes made
✅ **Pruning** - Deletes resources removed from Git
✅ **History** - Track all deployments
✅ **Multiple Apps** - Manage many deployments
✅ **GitOps Best Practice** - Git is single source of truth

## Troubleshooting

### Application OutOfSync
- Check Git changes
- Manual sync: `kubectl patch app helloworld-gitops -n argocd -p '{"metadata":{"annotations":{"argocd.argoproj.io/refresh":"hard"}}}'`

### Sync Failed
- Check Application logs: `kubectl describe app helloworld-gitops -n argocd`
- Check ArgoCD controller: `kubectl logs deployment/argocd-application-controller -n argocd`

### Wrong Namespace
- Update Application spec.destination.namespace

### Authentication
- For private repos, add SSH key to ArgoCD: `argocd repo add <url> --ssh-private-key-path ~/.ssh/id_rsa`

## Next Steps

1. Setup CI pipeline (GitHub Actions) to build Docker images
2. Update deployment.yaml image tag in Git
3. ArgoCD auto-syncs and deploys new version
4. Full CI/CD: GitHub Actions (CI) + ArgoCD (CD)
