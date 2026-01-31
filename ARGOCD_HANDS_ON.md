# 🎯 Bài Tập Thực Hành: ArgoCD Hands-On

> **Mục tiêu:** Bạn sẽ xây dựng một ứng dụng hoàn chỉnh, quản lý nó bằng ArgoCD, và hiểu rõ workflow GitOps

---

## 📝 Bài Tập 1: Tạo Ứng Dụng Đơn Giản (Nginx)

### Mục tiêu:
Triển khai một ứng dụng Nginx đơn giản bằng ArgoCD, từ viết Manifest → Push Git → Deploy

### Các bước:

#### **Step 1: Tạo cấu trúc thư mục**

```bash
mkdir -p apps/nginx-demo
cd apps/nginx-demo
```

#### **Step 2: Viết Deployment Manifest**

File: `apps/nginx-demo/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-demo
  labels:
    app: nginx-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - name: nginx
        image: nginx:1.19-alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
```

#### **Step 3: Viết Service Manifest**

File: `apps/nginx-demo/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-demo
  labels:
    app: nginx-demo
spec:
  type: LoadBalancer
  selector:
    app: nginx-demo
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

#### **Step 4: Viết ConfigMap (tùy chọn)**

File: `apps/nginx-demo/configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-demo-config
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>Welcome to Nginx Demo</title>
    </head>
    <body>
        <h1>🎉 Hello from Nginx Demo!</h1>
        <p>This app is deployed by <strong>ArgoCD</strong></p>
        <p>Version: <strong>1.0</strong></p>
    </body>
    </html>
```

#### **Step 5: Commit & Push lên Git**

```bash
cd /workspaces/learn-architect

git add apps/nginx-demo/
git commit -m "feat: add nginx demo app for ArgoCD"
git push
```

#### **Step 6: Tạo ArgoCD Application**

File: `argocd-apps/nginx-demo.yaml`

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-demo
  namespace: argocd
spec:
  # Project
  project: default
  
  # Source: Từ Git Repository
  source:
    repoURL: https://github.com/thucpn/learn-architect
    targetRevision: main
    path: apps/nginx-demo
  
  # Destination: Cluster & Namespace
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  
  # Sync Policy
  syncPolicy:
    automated:
      prune: true      # Xóa resources không còn trong Git
      selfHeal: true   # Auto-restore nếu ai đó xóa
    syncOptions:
    - CreateNamespace=true
```

#### **Step 7: Apply Application**

```bash
kubectl apply -f argocd-apps/nginx-demo.yaml
```

#### **Step 8: Xem trên ArgoCD UI**

1. Mở `http://localhost:8080`
2. Tìm app `nginx-demo`
3. Chờ status = **Synced** ✅
4. Xem các resources (Deployment, Service, Pods)

#### **Step 9: Kiểm tra Deployment**

```bash
# Xem pods
kubectl get pods

# Xem service
kubectl get svc

# Port-forward để test
kubectl port-forward svc/nginx-demo 8081:80 &

# Test trên browser
curl http://localhost:8081
```

### 📊 Kết quả mong đợi:

```
✅ 2 Pods của nginx-demo running
✅ Service nginx-demo tạo thành công
✅ ArgoCD UI hiển thị: Synced
✅ Có thể curl được tới app
```

---

## 📝 Bài Tập 2: Thay Đổi Manifest & Xem ArgoCD Auto-Sync

### Mục tiêu:
Hiểu workflow GitOps: Thay đổi → Push → Auto-deploy

### Các bước:

#### **Step 1: Thay đổi Replicas**

Sửa file `apps/nginx-demo/deployment.yaml`:

```yaml
# Thay từ:
replicas: 2

# Thành:
replicas: 3
```

#### **Step 2: Commit & Push**

```bash
git add apps/nginx-demo/deployment.yaml
git commit -m "scale: increase nginx replicas to 3"
git push
```

#### **Step 3: Theo dõi trên ArgoCD UI**

1. Vào `http://localhost:8080`
2. Click vào app `nginx-demo`
3. **Trong ~3 giây**, ArgoCD sẽ phát hiện thay đổi
4. Status chuyển từ **Synced** → **OutOfSync** (vài giây)
5. Tự động **Sync** (nếu auto-sync bật)
6. Quay về **Synced** ✅

#### **Step 4: Kiểm tra Pods**

```bash
kubectl get pods
# Sẽ thấy 3 Pods thay vì 2
```

### 💡 Điểm học được:

- ✨ Không cần chạy `kubectl apply` thủ công
- ✨ Git là nguồn chân lý (source of truth)
- ✨ Tất cả thay đổi được track bằng Git commit

---

## 📝 Bài Tập 3: Rollback bằng Git

### Mục tiêu:
Hiểu cách rollback tính năng bằng Git (không cần kubectl rollout)

### Các bước:

#### **Step 1: Xem Git History**

```bash
git log --oneline
# Sẽ thấy:
# abc123 scale: increase nginx replicas to 3
# def456 feat: add nginx demo app for ArgoCD
```

#### **Step 2: Revert Commit**

```bash
# Revert commit scale replicas
git revert abc123 -m "Revert: back to 2 replicas"

# Hoặc reset (cấp độ cao)
git reset --hard def456
```

#### **Step 3: Push lên Git**

```bash
git push
```

#### **Step 4: Theo dõi ArgoCD**

ArgoCD sẽ:
1. Phát hiện Git đã change
2. Deploy version cũ (2 replicas)
3. Xóa 1 Pod

```bash
# Xem pods giảm từ 3 → 2
kubectl get pods --watch
```

### 💡 Điểm học được:

- 🔄 Rollback chỉ cần `git revert` hoặc `git reset`
- 🔄 Không cần chỉnh `kubectl rollout undo` phức tạp
- 📜 Lịch sử rollback được track trong Git

---

## 📝 Bài Tập 4: Sử dụng Kustomize để Quản Lý Multi-Env

### Mục tiêu:
Học cách dùng Kustomize để manage Dev/Staging/Prod bằng same code

### Cấu trúc:

```
apps/nginx-multi-env/
├── kustomization.yaml          # Base config
├── deployment.yaml
├── service.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── replicas-patch.yaml
    ├── staging/
    │   ├── kustomization.yaml
    │   └── replicas-patch.yaml
    └── prod/
        ├── kustomization.yaml
        └── replicas-patch.yaml
```

#### **Step 1: Tạo Base Manifests**

File: `apps/nginx-multi-env/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: default

commonLabels:
  app: nginx-multi-env

resources:
- deployment.yaml
- service.yaml
```

File: `apps/nginx-multi-env/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multi-env
spec:
  replicas: 1  # Default, sẽ được override
  selector:
    matchLabels:
      app: nginx-multi-env
  template:
    metadata:
      labels:
        app: nginx-multi-env
    spec:
      containers:
      - name: nginx
        image: nginx:1.19-alpine
        ports:
        - containerPort: 80
```

File: `apps/nginx-multi-env/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-multi-env
spec:
  selector:
    app: nginx-multi-env
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

#### **Step 2: Tạo Dev Overlay**

File: `apps/nginx-multi-env/overlays/dev/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
- ../../

patchesJson6902:
- target:
    group: apps
    version: v1
    kind: Deployment
    name: nginx-multi-env
  patch: |-
    - op: replace
      path: /spec/replicas
      value: 1

nameSuffix: "-dev"
commonLabels:
  env: dev
```

#### **Step 3: Tạo Prod Overlay**

File: `apps/nginx-multi-env/overlays/prod/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
- ../../

patchesJson6902:
- target:
    group: apps
    version: v1
    kind: Deployment
    name: nginx-multi-env
  patch: |-
    - op: replace
      path: /spec/replicas
      value: 5

nameSuffix: "-prod"
commonLabels:
  env: prod
```

#### **Step 4: Tạo 2 ArgoCD Applications**

File: `argocd-apps/nginx-dev.yaml`

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/thucpn/learn-architect
    targetRevision: main
    path: apps/nginx-multi-env/overlays/dev
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

File: `argocd-apps/nginx-prod.yaml`

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/thucpn/learn-architect
    targetRevision: main
    path: apps/nginx-multi-env/overlays/prod
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

#### **Step 5: Apply Applications**

```bash
kubectl apply -f argocd-apps/nginx-dev.yaml
kubectl apply -f argocd-apps/nginx-prod.yaml
```

#### **Step 6: Kiểm tra**

```bash
# Dev: 1 pod
kubectl get pods -n dev

# Prod: 5 pods
kubectl get pods -n prod
```

### 💡 Điểm học được:

- 🎯 Kustomize cho phép manage config khác nhau
- 📦 Same code, khác environment
- 🏗️ Base + Overlays pattern

---

## 📝 Bài Tập 5: Setup Auto-Sync vs Manual Sync

### Mục tiêu:
Hiểu sự khác biệt giữa Auto-Sync và Manual Sync

### So sánh:

| Khía cạnh | Auto-Sync | Manual-Sync |
|----------|----------|------------|
| Khi deploy? | Tự động khi Git change | Phải bấm "Sync" |
| Thích hợp? | Production (trust system) | Staging (review trước) |
| Control? | Ít | Nhiều |
| Risk? | Cao (bug deploy ngay) | Thấp |

### Các bước:

#### **Step 1: Tạo App với Manual Sync**

File: `argocd-apps/nginx-manual.yaml`

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-manual
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/thucpn/learn-architect
    targetRevision: main
    path: apps/nginx-demo
  destination:
    server: https://kubernetes.default.svc
    namespace: manual-test
  syncPolicy: {}  # Không có automated!
```

#### **Step 2: Apply & Test**

```bash
kubectl apply -f argocd-apps/nginx-manual.yaml
```

#### **Step 3: Thay đổi Manifest**

```bash
# Thay replicas từ 2 → 4
# Push lên Git
git add .
git commit -m "test manual sync: 4 replicas"
git push
```

#### **Step 4: Theo dõi trên UI**

- Status chuyển sang **OutOfSync**
- Resources **KHÔNG** auto-deploy
- Phải bấn nút **Sync** trên UI
- Sau đó mới deploy

#### **Step 5: Cách manual sync bằng CLI**

```bash
argocd app sync nginx-manual
```

### 💡 Điểm học được:

- 📋 Manual Sync cho control tốt hơn
- ⚙️ Auto Sync cho automation tốt hơn
- 🤔 Chọn dựa vào trust level & risk

---

## 📝 Bài Tập 6: Health & Sync Status

### Mục tiêu:
Hiểu các trạng thái và cách đọc logs để debug

### Các trạng thái:

```
HEALTH:
🟢 Healthy  → Pods running, services active
🟡 Degraded → Một số pods pending/crashing
🔴 Unhealthy → Pods failed
⚪ Unknown  → Không biết

SYNC STATUS:
🟢 Synced    → Git = Cluster
🟡 OutOfSync → Git ≠ Cluster
🔴 Unknown   → Lỗi check status
```

### Các bước:

#### **Step 1: Tạo App có lỗi (để test)**

File: `argocd-apps/nginx-broken.yaml`

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-broken
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/thucpn/learn-architect
    targetRevision: main
    path: apps/nginx-demo
  destination:
    server: https://kubernetes.default.svc
    namespace: broken-test
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

#### **Step 2: Apply**

```bash
kubectl apply -f argocd-apps/nginx-broken.yaml
```

#### **Step 3: Xem logs**

```bash
# ArgoCD server logs
kubectl logs -n argocd svc/argocd-server

# Application Controller logs
kubectl logs -n argocd deployment/argocd-application-controller

# Repo Server logs (fetch từ Git)
kubectl logs -n argocd deployment/argocd-repo-server
```

#### **Step 4: Đọc detailed status**

```bash
# Dòng lệnh
argocd app get nginx-broken --refresh

# Hoặc kubectl
kubectl describe application nginx-broken -n argocd
```

### 💡 Điểm học được:

- 🔍 Cách đọc logs để debug
- 📊 Cách kiểm tra health status
- 🛠️ Cách fix sync issues

---

## ✅ Checklist Hoàn Thành

Sau khi làm hết bài tập, bạn đã:

- [ ] **Bài 1:** Tạo app Nginx cơ bản + deploy bằng ArgoCD
- [ ] **Bài 2:** Thay đổi manifest → phát hiện auto-deploy
- [ ] **Bài 3:** Rollback app bằng Git revert
- [ ] **Bài 4:** Dùng Kustomize quản lý Dev/Prod
- [ ] **Bài 5:** Hiểu Auto-Sync vs Manual-Sync
- [ ] **Bài 6:** Debug sử dụng logs & status

---

## 🎓 Kết Quả Cuối Cùng

Bạn sẽ hiểu rõ:

✨ **Workflow GitOps:** Git → Code Review → Merge → Auto Deploy  
✨ **Trách nhiệm ArgoCD:** Quản lý Desired State = Actual State  
✨ **Lợi ích:** Audit trail, Rollback dễ, Collaboration tốt  
✨ **Best practices:** Manifest organization, Multi-env, Secrets  

---

## 🚀 Challenge (Optional)

Nếu bạn muốn thử thách:

1. **Tạo Private Repo** và kết nối SSH với ArgoCD
2. **Setup Sealed Secrets** để quản lý credentials an toàn
3. **Tích hợp Slack notifications** khi app sync
4. **Tạo AppSet** để auto-create apps từ template
5. **Setup webhook** để instant sync (thay vì polling)

---

**Tiếp theo:** Liên hệ nếu bạn có câu hỏi hoặc muốn học thêm chủ đề nào! 🎯
