# Hướng dẫn viết Kubernetes manifest cho ArgoCD

Mục tiêu: Tài liệu này giúp bạn hiểu rõ các loại file YAML phổ biến (Deployment, Service, ConfigMap, Secret, Ingress, PVC, HPA, RBAC, v.v.), khi nào dùng, và các ví dụ thực tế.

---

## Tổng quan ngắn gọn
- Mỗi file YAML mô tả một resource Kubernetes (ví dụ: `Deployment`, `Service`).
- ArgoCD đọc các file này từ Git và áp dụng lên cluster để đảm bảo trạng thái thực tế khớp với trạng thái trong Git.
- Tách nhỏ file giúp maintainability, audit, reuse và apply patches (Kustomize/Helm).

---

## 1) `Deployment` — chạy ứng dụng (Pods)
- Mục đích: mô tả số replica, container image, strategy update, probes, resources.
- Khi thay đổi `image` hoặc `replicas` trong file và push lên Git → ArgoCD sẽ deploy thay đổi.

Ví dụ cơ bản:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "250m"
            memory: "256Mi"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
```

Advanced: update strategy (rolling update) và maxSurge/maxUnavailable:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

Gợi ý: luôn khai báo `selector` + `template.metadata.labels` khớp nhau.

---

## 2) `Service` — expose ứng dụng
- Mục đích: tạo endpoint nội bộ hoặc công khai cho Pod.
- `ClusterIP`: mặc định, chỉ trong cluster.
- `NodePort`: expose trên node port (tạm dùng cho dev).
- `LoadBalancer`: (minikube hỗ trợ bằng addon) tạo IP tạm thời; trong cloud sẽ tạo LB thật.

Ví dụ:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```

---

## 3) `ConfigMap` & `Secret`
- `ConfigMap`: lưu cấu hình text không nhạy cảm (HTML, config file, env var values).
- `Secret`: lưu dữ liệu nhạy cảm (passwords, tokens). Không commit plain secrets vào Git.

Secret an toàn hơn: dùng Sealed Secrets hoặc External Secrets để lưu encrypted/managed secrets.

ConfigMap ví dụ:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-config
data:
  WELCOME_MSG: "Hello from GitOps"
```

Secret ví dụ (base64 encoded):

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
stringData:
  username: admin
  password: s3cr3t
```

Gợi ý: tránh lưu secret plain-text trong repo; dùng sealed-secrets hoặc HashiCorp/Vault.

---

## 4) `Ingress` — expose HTTP(S) ở layer 7
- Dùng khi cần domain-based routing, TLS, path routing.
- Cần controller (nginx-ingress, ingress-nginx, traefik) đã cài trên cluster.

Ví dụ đơn giản:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: example.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
```

---

## 5) PersistentVolumeClaim (PVC) & Volume
- Dùng khi ứng dụng cần lưu trữ bền.
- PVC request dung lượng và storage class.

Ví dụ:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Mount vào Pod:

```yaml
volumes:
- name: data
  persistentVolumeClaim:
    claimName: data-pvc

containers:
- name: app
  image: myapp
  volumeMounts:
  - name: data
    mountPath: /data
```

---

## 6) `StatefulSet` — cho app stateful (DB, Kafka)
- Khi cần stable network id, ordered deploy, hoặc stable storage per pod.

Ví dụ tóm tắt: StatefulSet + PVC templates.

---

## 7) `HorizontalPodAutoscaler` (HPA)
- Tự scale Pod theo CPU/memory hoặc custom metrics.

Ví dụ:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
```

---

## 8) Liveness/Readiness/Startup Probes
- `livenessProbe`: kiểm tra pod còn "sống"; nếu fail -> restart.
- `readinessProbe`: kiểm tra app sẵn sàng nhận traffic; nếu fail -> service ngưng gửi traffic.
- `startupProbe`: dùng cho app khởi động chậm.

Ví dụ readiness:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

---

## 9) RBAC: `Role`, `RoleBinding`, `ClusterRole`, `ClusterRoleBinding`
- Quyền truy cập resources cho service accounts, người dùng.
- ArgoCD agent/ServiceAccount cần quyền để apply resources; thường ArgoCD chạy trong namespace `argocd` và cần access tới target namespaces (tùy config).

Ví dụ Role + Binding:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: app-manager
rules:
- apiGroups: [""]
  resources: ["pods","services","configmaps"]
  verbs: ["get","list","create","update","delete"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-manager-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: argocd-manager
  namespace: argocd
roleRef:
  kind: Role
  name: app-manager
  apiGroup: rbac.authorization.k8s.io
```

---

## 10) Kustomize & Overlays (multi-env)
- Kustomize cho phép patch base manifests cho `dev/staging/prod` bằng overlays.
- Tốt cho cấu trúc repo: `base/` + `overlays/{dev,prod}`.

Ví dụ `kustomization.yaml` overlay patch replicas:

```yaml
patchesJson6902:
- target:
    group: apps
    version: v1
    kind: Deployment
    name: web
  patch: |-
    - op: replace
      path: /spec/replicas
      value: 1
```

---

## 11) Helm charts
- Nếu project dùng Helm, lưu chart trong repo hoặc dùng chart repo; ArgoCD hỗ trợ Helm as source.
- `values.yaml` là nơi override cấu hình cho environment.

---

## 12) Patterns: Canary / Blue-Green (thực tế)
- ArgoCD bản thân sync manifests; để thực hiện canary/blue-green chuyên nghiệp thường dùng `Argo Rollouts` hoặc CI step tạo resources tạm thời.
- Ví dụ đơn giản: tăng dần `replicas` ở một Deployment mới, hoặc dùng `Service` chuyển traffic.
- Đề xuất: dùng `Argo Rollouts` để có builtin canary strategies.

---

## 13) Common pitfalls & best practices
- Không lưu secrets plain-text trong Git. Dùng SealedSecrets / Vault / External Secrets.
- Tránh dùng `image: latest`. Dùng immutable tags (v1.2.3) để reproducibility.
- Luôn khai báo `resources.requests` và `resources.limits`.
- Dùng probes để health checks; không rely only on Pod status.
- Keep manifests small & modular (per resource). Một file = một resource hoặc logical group.
- Use `CreateNamespace=true` option in ArgoCD Application if app creates its own namespace.
- Validate manifests locally: `kubectl apply --dry-run=client -f .` and `kubeval` tools.

---

## 14) Real-world examples (short)

1) Web app + DB (separate manifests):
- `frontend/deployment.yaml` (image, env from ConfigMap/Secret)
- `frontend/service.yaml`
- `backend/deployment.yaml` (env from Secret)
- `database/statefulset.yaml` + `pvc.yaml`
- `argocd-app.yaml` pointing to path `apps/webapp`

2) CronJob for batch tasks:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-job
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: job
            image: myjob:1.0
          restartPolicy: OnFailure
```

3) Using ConfigMap for feature flags:
- Update ConfigMap in Git → ArgoCD updates app (app reads config periodically or via mounted file reload).

---

## 15) Quick checklist before commit
- [ ] Validate YAML syntax
- [ ] Avoid secrets in plain text
- [ ] Pin image tags
- [ ] Add resource requests/limits
- [ ] Add probes
- [ ] Ensure labels/selectors match
- [ ] Add namespace or enable CreateNamespace

---

## Kết luận
File YAML là cách bạn mô tả mong muốn cho cluster. Việc tách file theo loại resource giúp quản lý, review, deploy và rollback an toàn thông qua GitOps + ArgoCD. Nếu bạn muốn, tôi có thể:

- Commit các file `apps/nginx-demo/*` và `argocd-apps/nginx-demo.yaml` lên repo và apply `argocd-apps/nginx-demo.yaml` để ArgoCD tạo Application và deploy.
- Hoặc mở rộng hướng dẫn với mẫu `StatefulSet`, `Ingress` TLS, hoặc SealedSecrets.

Bạn muốn tôi làm bước nào tiếp theo?