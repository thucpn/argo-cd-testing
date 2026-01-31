# 🔍 ArgoCD Quick Reference Guide

Tài liệu tra cứu nhanh cho những lệnh và concepts thường dùng.

---

## 📋 Concepts Nhanh

### GitOps
```
Phương pháp: Git = Single Source of Truth
Workflow: Code → Git → ArgoCD phát hiện → Auto-deploy
Lợi ích: Audit trail, Rollback dễ, Reproducible
```

### Pull vs Push Deployment

```
❌ PUSH (Cũ):
Dev → Build → Push image → CI/CD chạy script → kubectl apply
Vấn đề: Khó audit, no rollback easy, khó multi-cluster

✅ PULL (GitOps):
Dev → Git → ArgoCD watch → Auto-deploy
Lợi ích: Audit trail tốt, Rollback dễ, Safe
```

### Application States

```
SYNC:
🟢 Synced      = Git state = Cluster state (OK)
🟡 OutOfSync   = Git state ≠ Cluster state (cần deploy)
🔴 Unknown     = Lỗi khi check (debug needed)

HEALTH:
🟢 Healthy     = Pods running, services ready
🟡 Degraded    = Pods pending/failing
🔴 Unhealthy   = App failed
⚪ Unknown     = Chưa check
```

### Sync Policies

```
AUTOMATED:
syncPolicy:
  automated:
    prune: true      # Xóa resources không có trong Git
    selfHeal: true   # Nếu ai xóa resource, auto-restore

MANUAL:
syncPolicy: {}       # Phải bấm "Sync" trên UI hoặc CLI
```

---

## 🎯 Workflow Nhanh

### Cách deploy app lần đầu:

```bash
# 1. Viết manifests
mkdir -p apps/myapp
echo "..." > apps/myapp/deployment.yaml
echo "..." > apps/myapp/service.yaml

# 2. Commit & Push lên Git
git add apps/myapp/
git commit -m "add: myapp deployment"
git push

# 3. Tạo ArgoCD Application
cat > argocd-apps/myapp.yaml << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/USER/REPO
    path: apps/myapp
    targetRevision: main
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF

# 4. Apply Application
kubectl apply -f argocd-apps/myapp.yaml

# 5. Check UI hoặc CLI
argocd app get myapp
```

### Cách update app:

```bash
# 1. Sửa manifest (ví dụ change image version)
vim apps/myapp/deployment.yaml
# Thay: image: myapp:v1.0 → myapp:v2.0

# 2. Commit & Push
git add apps/myapp/deployment.yaml
git commit -m "update: myapp to v2.0"
git push

# 3. ArgoCD tự động detect (~3 giây)
# 4. Tự động deploy nếu auto-sync bật
```

### Cách rollback app:

```bash
# 1. Revert Git commit
git revert <commit-hash>
git push

# 2. ArgoCD tự động deploy phiên bản cũ
# Hoặc dùng CLI
argocd app rollback myapp 0  # revision 0
```

---

## 🛠️ CLI Commands

### Login & Session

```bash
# Login
argocd login <server> --username admin --password <pwd>

# Logout
argocd logout <server>

# Status
argocd app get <app-name>
```

### List & Describe

```bash
# List all apps
argocd app list

# Get details
argocd app get myapp

# Get with refresh (force check Git)
argocd app get myapp --refresh

# Watch status
argocd app wait myapp --sync
```

### Sync

```bash
# Manual sync
argocd app sync myapp

# Sync tới revision cụ thể
argocd app sync myapp --revision abc123

# Sync with prune (xóa resources không cần)
argocd app sync myapp --prune
```

### Rollback

```bash
# Xem revision history
argocd app history myapp

# Rollback tới revision cũ
argocd app rollback myapp 0

# Rollback tới commit cụ thể
argocd app rollback myapp --revision abc123
```

### Other

```bash
# View logs
argocd app logs myapp

# Diff (so sánh Git vs Cluster)
argocd app diff myapp

# Delete app
argocd app delete myapp

# Create app (CLI)
argocd app create myapp \
  --repo https://github.com/user/repo \
  --path apps/myapp \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

---

## 🐚 Kubectl Commands

### ArgoCD Applications

```bash
# List applications
kubectl get applications -n argocd

# Get application
kubectl get application myapp -n argocd

# Describe (detailed info)
kubectl describe application myapp -n argocd

# Logs
kubectl logs -n argocd deployment/argocd-application-controller
kubectl logs -n argocd deployment/argocd-repo-server
kubectl logs -n argocd svc/argocd-server

# Edit (change directly, not recommended)
kubectl edit application myapp -n argocd

# Delete
kubectl delete application myapp -n argocd
```

### Ports & Access

```bash
# Port-forward ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Port-forward app service
kubectl port-forward svc/myapp 8000:80

# Proxy to access via http://localhost:8080/
kubectl proxy

# Get LoadBalancer IP (if exposed)
kubectl get svc -n argocd argocd-server -o wide
```

### Resources

```bash
# List all apps' resources
kubectl get all --namespace default

# Watch pod creation
kubectl get pods -w

# View pod logs
kubectl logs <pod-name> -f

# Describe pod
kubectl describe pod <pod-name>

# Execute command in pod
kubectl exec -it <pod-name> -- sh
```

---

## 🐛 Debugging

### App OutOfSync

```bash
# Check detailed diff
argocd app diff myapp

# Hoặc dùng kubectl
kubectl diff -f apps/myapp/

# Force sync
argocd app sync myapp --force

# Check Git connection
kubectl logs -n argocd deployment/argocd-repo-server
```

### Sync Failed

```bash
# Check app status
argocd app get myapp

# Check detailed error
kubectl describe application myapp -n argocd

# Check controller logs
kubectl logs -n argocd deployment/argocd-application-controller

# Check manifest syntax
kubectl apply -f apps/myapp/ --dry-run=client

# Validate manifests
kubectl apply -f apps/myapp/ --validate=true --dry-run=client
```

### Git Connection Issues

```bash
# Test SSH connection (if using SSH)
ssh -T git@github.com

# Check repository credentials
kubectl get secrets -n argocd | grep repo

# Check webhook (if using webhook)
kubectl logs -n argocd deployment/argocd-server | grep webhook

# Restart repo-server
kubectl rollout restart deployment/argocd-repo-server -n argocd
```

### ArgoCD Server Issues

```bash
# Check all pods running
kubectl get pods -n argocd

# Check resource usage
kubectl top nodes
kubectl top pods -n argocd

# Describe pod crashes
kubectl describe pod <argocd-pod> -n argocd

# Check logs
kubectl logs -n argocd pod/argocd-server-xxxxx
```

---

## 📁 Manifest Templates

### Minimal Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo
    targetRevision: main
    path: apps/myapp
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### With Kustomize

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo
    targetRevision: main
    path: apps/myapp/overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### With Helm

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.example.com
    chart: myapp
    targetRevision: 1.0.0
    helm:
      values: |
        replicaCount: 3
        image:
          tag: v2.0
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### ApplicationSet (Auto-create apps)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapps
  namespace: argocd
spec:
  generators:
  - list:
      elements:
      - cluster: dev
        namespace: dev
      - cluster: prod
        namespace: prod
  template:
    metadata:
      name: 'myapp-{{cluster}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/user/repo
        targetRevision: main
        path: apps/myapp/overlays/{{cluster}}
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{namespace}}'
```

---

## 📊 Useful Links

### Official
- Docs: https://argo-cd.readthedocs.io/
- GitHub: https://github.com/argoproj/argo-cd
- Examples: https://github.com/argoproj/argocd-example-apps

### Learning
- Getting Started: https://argo-cd.readthedocs.io/en/stable/getting_started/
- Best Practices: https://argo-cd.readthedocs.io/en/stable/best_practices/
- FAQ: https://argo-cd.readthedocs.io/en/stable/faq/

### Community
- Slack: https://argoproj.github.io/community/
- Issues: https://github.com/argoproj/argo-cd/issues

---

## ⏱️ Common Timeouts & Waits

```bash
# ArgoCD detects Git change: ~3 seconds
# Pod creation: ~10-30 seconds
# Full sync: 1-2 minutes
# Health check: 10 seconds default

# Wait for app to be healthy
argocd app wait myapp --health

# Wait for sync
argocd app wait myapp --sync
```

---

## 🎓 Quick Decision Guide

### Bạn muốn:
- **Deploy app lần đầu?** → Viết manifest + tạo Application
- **Update version?** → Sửa manifest, commit, push
- **Rollback?** → `git revert` hoặc `argocd app rollback`
- **Multi-env?** → Dùng Kustomize overlays
- **Reusable app?** → Dùng Helm charts
- **Batch create apps?** → Dùng ApplicationSet
- **Secrets an toàn?** → Dùng Sealed Secrets
- **Instant sync?** → Setup webhook
- **Debug issue?** → Check logs & status

---

**Mẹo:** Bookmark trang này để tra cứu nhanh! 🔖
