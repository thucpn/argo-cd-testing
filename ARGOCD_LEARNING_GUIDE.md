# 📚 Hướng Dẫn Học ArgoCD - Từ Cơ Bản Đến Thực Hành

> **Mục tiêu:** Bạn sẽ hiểu ArgoCD là gì, tại sao cần dùng nó, và cách áp dụng nó trong các dự án thực tế.

---

## 📖 Phần 1: Khái Niệm Cơ Bản
\
### 1.1 ArgoCD là gì?

**ArgoCD** là một công cụ **Continuous Delivery** dựa trên kiến trúc **GitOps** cho Kubernetes. 

Nôm na:
- 🎯 **Mục đích:** Tự động triển khai ứng dụng lên Kubernetes một cách tuyên bố (declarative)
- 📝 **Cách làm:** Bạn khai báo trạng thái mong muốn trong **Git** → ArgoCD tự động áp dụng lên cluster
- 🔄 **Bản chất:** Pull-based deployment (không phải push)

### 1.2 GitOps là gì?

**GitOps** là một phương pháp quản lý hạ tầng và ứng dụng bằng cách:
1. **Lưu tất cả cấu hình** dưới dạng code trong Git repository
2. **Git là nguồn chân lý duy nhất** (Single Source of Truth)
3. **Kiểu khai báo** (Declarative) thay vì chỉ thị (Imperative)
4. **Tự động hóa** việc triển khai khi code thay đổi

### 1.3 Tại sao cần ArgoCD?

#### ❌ Cách cũ (Trước GitOps):
```bash
# Phải chạy lệnh thủ công từng lần
kubectl apply -f deployment.yaml
kubectl set image deployment/myapp myapp=myapp:v2.0
kubectl rollout restart deployment/myapp
```

**Vấn đề:**
- 😰 Dễ quên, dễ làm sai
- 📊 Khó tracking lịch sử thay đổi
- 🔀 Cluster state không khớp với code
- ⛔ Không có audit trail (dấu vết)
- 🤝 Khó collaborate giữa các team

#### ✅ Cách mới (Với ArgoCD):
```yaml
# Tệp deployment.yaml được lưu trong Git
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0  # Thay đổi này trong Git
```

**Lợi ích:**
- ✨ Tất cả thay đổi được track trong Git
- 🤖 Tự động deployed khi push code
- 📈 Dễ rollback (chỉ cần revert Git commit)
- 🔍 Audit trail rõ ràng
- 🎯 Cluster state luôn khớp với Git

---

## 🏗️ Phần 2: Kiến Trúc ArgoCD

### 2.1 Các thành phần chính

```
┌─────────────────────────────────────────────────────────┐
│                      GIT REPOSITORY                     │
│  (Chứa các file Manifest: Deployment, Service, etc)    │
└──────────────────────┬──────────────────────────────────┘
                       │
                       │ (Poll / Watch for changes)
                       ▼
┌─────────────────────────────────────────────────────────┐
│                  ARGOCD APPLICATION                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ API Server   │  │ Repo Server  │  │ Application  │  │
│  │              │  │              │  │ Controller   │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐                    │
│  │ Dex Server   │  │ Redis Cache  │                    │
│  │ (Auth)       │  │              │                    │
│  └──────────────┘  └──────────────┘                    │
└─────────────────────────┬──────────────────────────────┘
                          │
                          │ (Apply manifests)
                          ▼
┌─────────────────────────────────────────────────────────┐
│              KUBERNETES CLUSTER                         │
│  ┌──────────────┐  ┌──────────────┐                    │
│  │  Deployment  │  │   Service    │                    │
│  │  (Pods)      │  │  (Port)      │                    │
│  └──────────────┘  └──────────────┘                    │
└─────────────────────────────────────────────────────────┘
```

### 2.2 Chi tiết các thành phần

| Thành phần | Vai trò |
|-----------|--------|
| **API Server** | Web UI, REST API, login, logout |
| **Repo Server** | Kết nối Git repo, fetch manifests |
| **Application Controller** | So sánh desired state (Git) vs actual state (Cluster), auto-sync |
| **Dex Server** | Xác thực người dùng (SSO integration) |
| **Redis** | Cache |

### 2.3 Workflow cơ bản

```
1. BẠN PUSH CODE LÊN GIT
   └─> Commit: deployment.yaml với image: v1.0

2. ARGOCD PHÁT HIỆN THAY ĐỔI (khoảng 3 giây)
   └─> Repo Server fetch mới từ Git

3. ARGOCD SO SÁNH TRẠNG THÁI
   Desired State (Git):  image: v1.0, replicas: 3
   Actual State (K8s):   image: v0.9, replicas: 2
   
4. PHÁT HIỆN KHÔNG KHỚP → ĐỐI CHIẾU KHÁC BIỆT

5. ARGOCD TỰ ĐỘNG TRIỂN KHI (nếu Auto-Sync bật)
   └─> kubectl apply -f deployment.yaml

6. CLUSTER CẬP NHẬT
   └─> Pods được tạo/cập nhật

7. ARGOCD HIỂN THỊ TRẠNG THÁI XANH ✅
   └─> Synced status trong UI
```

---

## 🚀 Phần 3: Cách Dùng ArgoCD

### 3.1 Các khái niệm quan trọng

#### **Application**
Là một unit triển khai trong ArgoCD. Nó chứa:
- Git repository URL (nơi chứa Manifest)
- Path trong repo (thư mục chứa YAML files)
- Target cluster (cluster nào sẽ deploy)
- Sync policy (cách triển khai)

#### **Sync**
Quá trình ArgoCD áp dụng manifests từ Git lên Kubernetes cluster.

**Các trạng thái:**
- 🟢 **Synced:** Cluster state = Git state
- 🟡 **OutOfSync:** Cluster state ≠ Git state (có thay đổi trong Git chưa apply)
- 🔴 **Error:** Có lỗi trong quá trình sync

#### **Sync Policy**

**Manual Sync:**
```yaml
syncPolicy:
  automated: null  # Phải bấm "Sync" trên UI
```
👉 Bạn phải bấm nút "Sync" trên giao diện UI

**Auto Sync:**
```yaml
syncPolicy:
  automated:
    prune: true      # Xóa resources không còn trong Git
    selfHeal: true   # Nếu ai đó xóa resource, auto-restore
```
👉 Tự động deploy ngay khi phát hiện thay đổi

---

## 💎 Phần 4: Lợi Ích & Use Cases

### 4.1 Lợi ích chính

| Lợi ích | Mô tả |
|---------|--------|
| **Git as Source of Truth** | Tất cả cấu hình được version control |
| **Audit Trail** | Biết ai, khi nào, thay đổi gì |
| **Easy Rollback** | Chỉ cần `git revert` để quay lại phiên bản cũ |
| **Reproducibility** | Dễ dàng tạo lại hệ thống từ Git |
| **Collaboration** | Team có thể review PR trước khi merge |
| **Disaster Recovery** | Nếu cluster bị xóa, chỉ cần reapply từ Git |
| **Environment Consistency** | Dev, Staging, Prod config như nhau |
| **DevOps Automation** | Không cần chạy kubectl lệnh thủ công |
| **Multi-Cluster** | Dễ deploy tới nhiều cluster cùng lúc |
| **Pull-based (Safer)** | Cluster chủ động kéo, không bị push từ bên ngoài |

### 4.2 Use cases thực tế

#### **Use Case 1: Continuous Deployment (CD Pipeline)**
```
Developer push code → Git → ArgoCD phát hiện → Auto deploy → User dùng
```
📱 **Ví dụ:** Web app mới có update → tự động lên production

#### **Use Case 2: Multi-Environment Management**
```
repository/
├── dev/
│   └── deployment.yaml (replicas: 1)
├── staging/
│   └── deployment.yaml (replicas: 2)
└── prod/
    └── deployment.yaml (replicas: 5)
```
🌍 Cùng 1 app, khác cấu hình cho từng môi trường

#### **Use Case 3: ConfigOps (Configuration Management)**
```
Tất cả cấu hình (env variables, secrets, configs) → lưu trong Git
ArgoCD tự động apply → không cần thay đổi thủ công trên cluster
```

#### **Use Case 4: Disaster Recovery**
```
Cluster cũ bị crash → Xóa hết
Cluster mới → ArgoCD reapply từ Git → Hệ thống back online ngay
```

#### **Use Case 5: Team Collaboration**
```
Dev A: Chỉnh sửa deployment.yaml → Push PR
Dev B: Review code → Approve → Merge
ArgoCD: Tự động deploy bản cập nhật
```

---

## 🔄 Phần 5: Quy Trình Thực Hành

### 5.1 Quy trình chuẩn GitOps + ArgoCD

```
STEP 1: BẠN VIẾT/CHỈNH MANIFEST
   └─> Tạo file YAML (deployment.yaml, service.yaml, etc)

STEP 2: PUSH LÊN GIT
   └─> git add . && git commit && git push

STEP 3: ARGOCD PHÁT HIỆN
   └─> Tự động fetch từ Git repo

STEP 4: SO SÁNH & SYNC
   └─> Compare Git vs Cluster → Apply nếu khác

STEP 5: MONITORING
   └─> Xem UI, check status, logs

STEP 6: NẾU CÓ VẤN ĐỀ
   ├─> Rollback: git revert old-commit
   └─> ArgoCD tự động deploy lại bản cũ
```

### 5.2 Nguyên tắc vàng

```
✅ DO:
- Lưu TẤT CẢ manifests trong Git
- Tạo branch cho từng feature
- Review & approve PR trước merge
- Bật auto-sync để tự động deploy
- Monitor ArgoCD UI thường xuyên

❌ DON'T:
- Chạy kubectl apply/delete trực tiếp trên cluster
- Thay đổi resources trực tiếp trong cluster
- Push manifests không review
- Dùng image tags mơ hồ (ví dụ "latest")
- Lấy secrets từ Git (dùng tools như Sealed Secrets, External Secrets)
```

---

## 🎯 Phần 6: Ví Dụ Thực Hành

### 6.1 Tạo ứng dụng đầu tiên

**Bước 1: Viết Manifest**

File: `apps/nginx-app/deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
```

File: `apps/nginx-app/service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-app
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

**Bước 2: Push lên Git**
```bash
git add apps/nginx-app/
git commit -m "feat: add nginx-app deployment"
git push
```

**Bước 3: Tạo ArgoCD Application**

File: `argocd-apps/nginx-app.yaml`
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/YOUR_USERNAME/learn-architect
    targetRevision: main
    path: apps/nginx-app
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

**Bước 4: Apply Application definition**
```bash
kubectl apply -f argocd-apps/nginx-app.yaml
```

**Bước 5: Xem UI ArgoCD**
- Vào `http://localhost:8080`
- Sẽ thấy app "nginx-app" 
- Status: Synced (xanh ✅)

**Bước 6: Thay đổi và xem tác dụng**
```yaml
# Thay đổi trong deployment.yaml
replicas: 3  # từ 2 → 3
image: nginx:latest  # từ 1.19 → latest
```

```bash
git add .
git commit -m "update: nginx app to 3 replicas"
git push
```

→ ArgoCD sẽ tự động phát hiện trong 3 giây
→ Pods sẽ scale từ 2 → 3
→ Xem livewatch trên ArgoCD UI

---

## 📋 Phần 7: Các Lệnh Thường Dùng

### 7.1 Lệnh ArgoCD CLI

```bash
# Đăng nhập
argocd login localhost:8080 --username admin --password YOUR_PASSWORD

# Liệt kê applications
argocd app list

# Xem chi tiết application
argocd app get nginx-app

# Manual sync
argocd app sync nginx-app

# Xem logs của app
argocd app logs nginx-app

# Rollback
argocd app rollback nginx-app 0  # quay về revision 0
```

### 7.2 Lệnh kubectl

```bash
# Xem ArgoCD applications
kubectl get applications -n argocd

# Xem chi tiết
kubectl describe application nginx-app -n argocd

# Xem logs của ArgoCD controller
kubectl logs -n argocd -f argocd-application-controller-0

# Xem tất cả resources tạo bởi ArgoCD
kubectl get all -n default
```

---

## 🧩 Phần 8: Best Practices

### 8.1 Cấu trúc Repository

```
learn-architect/
├── README.md
├── docs/
│   ├── SETUP.md
│   └── WORKFLOWS.md
├── apps/
│   ├── frontend/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── configmap.yaml
│   ├── backend/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── configmap.yaml
│   └── database/
│       └── ...
├── argocd-apps/
│   ├── frontend-app.yaml
│   ├── backend-app.yaml
│   └── kustomization.yaml
└── kustomize/
    ├── base/
    │   └── deployment.yaml
    └── overlays/
        ├── dev/
        └── prod/
```

### 8.2 Kustomize vs Helm

**Kustomize** (đơn giản):
- Template-free, pure YAML
- Tốt cho projects nhỏ-vừa
- Build manifests từ base + overlays

**Helm** (phức tạp):
- Package manager cho K8s
- Tốt cho projects lớn, reusable charts
- Go templating

ArgoCD hỗ trợ cả hai!

### 8.3 Secrets Management

❌ **KHÔNG bao giờ** lưu secrets (passwords, API keys) trực tiếp trong Git!

✅ **Các cách an toàn:**

1. **Sealed Secrets**: Mã hóa secrets trong Git
2. **External Secrets Operator**: Lấy từ external secret manager
3. **AWS Secrets Manager / Azure Key Vault**: Tích hợp với cloud
4. **Bitnami Sealed Secrets**: ArgoCD hỗ trợ native

---

## 🎓 Phần 9: Troubleshooting

### 9.1 App OutOfSync

```
❌ Vấn đề: Application status = OutOfSync
```

**Nguyên nhân:** Có thay đổi trong Git chưa được apply lên cluster

**Cách fix:**
```bash
# Option 1: Bấm "Sync" trên UI
# Option 2: Dùng CLI
argocd app sync nginx-app

# Option 3: Check chi tiết khác biệt
kubectl diff -f apps/nginx-app/
```

### 9.2 Sync Failed

```
❌ Vấn đề: Sync lỗi, status = red
```

**Cách debug:**
```bash
# Xem logs
kubectl logs -n argocd -f argocd-application-controller-0

# Xem detailed status của app
kubectl describe application nginx-app -n argocd

# Check manifest validity
kubectl apply -f apps/nginx-app/ --dry-run=client
```

### 9.3 Git Connection Error

```
❌ Vấn đề: ArgoCD không thể kết nối Git repo
```

**Cách fix:**
```bash
# Check SSH keys nếu dùng SSH
ssh -T git@github.com

# Check credentials
kubectl get secrets -n argocd | grep repo

# Restart repo server
kubectl rollout restart deployment/argocd-repo-server -n argocd
```

---

## 📚 Tài Liệu Tham Khảo

- **Trang chủ:** https://argoproj.github.io/argo-cd/
- **Docs:** https://argo-cd.readthedocs.io/
- **GitHub:** https://github.com/argoproj/argo-cd
- **Example Apps:** https://github.com/argoproj/argocd-example-apps

---

## ✅ Checklist: Bạn đã học được gì?

- [ ] Hiểu ArgoCD là gì và tại sao cần dùng
- [ ] Biết GitOps là gì và lợi ích
- [ ] Nắm kiến trúc của ArgoCD
- [ ] Hiểu workflow: Git → ArgoCD → Cluster
- [ ] Biết các khái niệm: Application, Sync, Sync Policy
- [ ] Có thể viết Manifest đơn giản
- [ ] Có thể tạo ArgoCD Application
- [ ] Biết cách manual sync và auto-sync
- [ ] Hiểu best practices cơ bản
- [ ] Biết cách debug issues

---

**Tiếp theo:** Hãy bắt tay vào [bài tập thực hành](./ARGOCD_HANDS_ON.md) để áp dụng những kiến thức này! 🚀
