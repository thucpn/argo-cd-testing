# 📚 ArgoCD Learning Path - Complete Guide

Xin chào! 👋 Đây là một bộ tài liệu toàn diện để bạn học ArgoCD từ cơ bản đến nâng cao.

---

## 📖 Tài Liệu Học Tập

### 1. **[ARGOCD_LEARNING_GUIDE.md](./ARGOCD_LEARNING_GUIDE.md)** - Khái Niệm & Lý Thuyết
   - ✅ ArgoCD là gì?
   - ✅ GitOps là gì?
   - ✅ Kiến trúc ArgoCD (components)
   - ✅ Workflow cơ bản
   - ✅ Lợi ích chính
   - ✅ Use cases thực tế
   - ✅ Best practices
   - ✅ Troubleshooting cơ bản

   **Bạn sẽ học được:**
   - Tại sao cần ArgoCD
   - Cách nó hoạt động
   - Những lợi ích chính

---

### 2. **[ARGOCD_HANDS_ON.md](./ARGOCD_HANDS_ON.md)** - Bài Tập Thực Hành
   
   #### 📝 Bài 1: Deploy Nginx App
   - Viết deployment manifest
   - Viết service manifest
   - Tạo ArgoCD Application
   - Deploy tới cluster
   - Kiểm tra kết quả

   #### 📝 Bài 2: Auto-Sync & Update
   - Thay đổi replicas
   - Thay đổi image version
   - Xem ArgoCD tự động deploy
   - Theo dõi trên UI

   #### 📝 Bài 3: Rollback bằng Git
   - Revert commit
   - Xem cluster tự động restore
   - Hiểu power của Git

   #### 📝 Bài 4: Multi-Environment (Kustomize)
   - Setup base manifests
   - Tạo dev/staging/prod overlays
   - Deploy cùng app tới nhiều env
   - Quản lý config khác nhau

   #### 📝 Bài 5: Manual vs Auto-Sync
   - So sánh 2 approaches
   - Khi nào dùng cái nào
   - Test cả 2 cách

   #### 📝 Bài 6: Health & Sync Status
   - Hiểu các trạng thái
   - Debug issues
   - Đọc logs

   **Thời gian:** ~3-4 giờ để hoàn thành tất cả bài tập

---

### 3. **[ARGOCD_DIAGRAMS.md](./ARGOCD_DIAGRAMS.md)** - Hình Ảnh & Sơ Đồ
   - 🏗️ Kiến trúc tổng quan
   - 🔄 Workflow chi tiết (từng bước)
   - 💥 Các scenarios khác nhau
   - 📊 Pull vs Push deployment
   - 🗂️ Repository structure recommendations
   - 🎓 Learning path visualization

   **Dùng khi:** Cần visual để hiểu rõ hơn

---

### 4. **[ARGOCD_QUICK_REFERENCE.md](./ARGOCD_QUICK_REFERENCE.md)** - Tra Cứu Nhanh
   - ⚡ Concepts nhanh
   - 🎯 Workflow nhanh
   - 🛠️ Lệnh ArgoCD CLI
   - 🐚 Lệnh kubectl
   - 🐛 Debugging tips
   - 📁 Template manifests
   - 📊 Decision guide

   **Dùng khi:** Cần tra cứu lệnh hoặc syntax nhanh

---

## 🎯 Learning Path Recommended

### Tuần 1: Hiểu Khái Niệm
```
Day 1: Đọc ARGOCD_LEARNING_GUIDE.md (Phần 1-2)
       └─ Hiểu GitOps & kiến trúc

Day 2: Đọc ARGOCD_LEARNING_GUIDE.md (Phần 3-4)
       └─ Hiểu workflow & lợi ích

Day 3: Xem ARGOCD_DIAGRAMS.md
       └─ Visual understanding

Day 4: Đọc ARGOCD_LEARNING_GUIDE.md (Phần 5-8)
       └─ Best practices & troubleshooting
```

### Tuần 2: Thực Hành
```
Day 1: Làm Bài 1-2 (ARGOCD_HANDS_ON.md)
       └─ Deploy & auto-sync

Day 2: Làm Bài 3-4 (ARGOCD_HANDS_ON.md)
       └─ Rollback & multi-env

Day 3: Làm Bài 5-6 (ARGOCD_HANDS_ON.md)
       └─ Sync policies & debugging

Day 4: Challenge questions & review
       └─ Consolidate learning
```

---

## 🔍 Quick Access by Topic

### Muốn biết...

#### **"ArgoCD là gì?"**
→ ARGOCD_LEARNING_GUIDE.md → Phần 1

#### **"Kiến trúc như thế nào?"**
→ ARGOCD_LEARNING_GUIDE.md → Phần 2
→ ARGOCD_DIAGRAMS.md → Kiến Trúc Tổng Quan

#### **"Có lợi ích gì?"**
→ ARGOCD_LEARNING_GUIDE.md → Phần 4

#### **"Làm cách nào để deploy?"**
→ ARGOCD_HANDS_ON.md → Bài 1

#### **"Tôi cần lệnh gì?"**
→ ARGOCD_QUICK_REFERENCE.md → CLI Commands

#### **"Làm sao debug khi lỗi?"**
→ ARGOCD_QUICK_REFERENCE.md → Debugging
→ ARGOCD_LEARNING_GUIDE.md → Phần 9

#### **"Cấu trúc repo như thế nào?"**
→ ARGOCD_LEARNING_GUIDE.md → Phần 8.1
→ ARGOCD_DIAGRAMS.md → Repository Structure

#### **"Tôi muốn thấy workflow trực quan"**
→ ARGOCD_DIAGRAMS.md → Workflow Chi Tiết

---

## 📊 Current Status

```
✅ Setup:
   ├── Docker ............................ Done
   ├── Minikube .......................... Done
   ├── kubectl ........................... Done
   ├── ArgoCD CLI ........................ Done
   ├── ArgoCD Server ..................... Running on cluster
   └── Port Forwarding ................... Port 8080:443

📚 Learning Materials:
   ├── ARGOCD_LEARNING_GUIDE.md ......... ✅ Ready
   ├── ARGOCD_HANDS_ON.md ............... ✅ Ready
   ├── ARGOCD_DIAGRAMS.md ............... ✅ Ready
   ├── ARGOCD_QUICK_REFERENCE.md ........ ✅ Ready
   └── README.md (this file) ............ ✅ Ready

🎯 Next Steps:
   1. Read learning materials (suggest: Start with Learning Guide)
   2. Work through hands-on exercises
   3. Use quick reference as needed
   4. Practice on your own projects
```

---

## 🔌 Access ArgoCD UI

```bash
# UI is currently running at:
http://localhost:8080

# If port forwarding closed, restart:
kubectl port-forward svc/argocd-server -n argocd 8080:443 &

# Credentials:
Username: admin
Password: O8oglSFqjFruhH2c
```

---

## 💡 Tips for Success

### 1. **Hiểu Concept Trước Khi Làm**
   - Đọc learning guide trước
   - Xem diagrams để visual
   - Rồi mới đến hands-on

### 2. **Làm Bài Tập Từ Từ**
   - Không cần vội
   - Hiểu từng step
   - Thử modify & experiment

### 3. **Dùng Quick Reference**
   - Bookmark nó
   - Tra cứu khi cần
   - Tiết kiệm thời gian

### 4. **Monitor ArgoCD UI**
   - Watch real-time changes
   - Xem resource tree
   - Hiểu lifecycle

### 5. **Read Logs When Error**
   - kubectl logs rất useful
   - ArgoCD logs rất detailed
   - Học từ error messages

### 6. **Experiment Safely**
   - Tạo namespace tách biệt
   - Delete app & retry
   - Không sợ "break" anything

---

## 📝 Glossary

| Term | Meaning |
|------|---------|
| **Application** | ArgoCD resource định nghĩa cái gì deploy từ đâu |
| **Sync** | Quá trình apply Git manifests lên cluster |
| **OutOfSync** | Git state ≠ Cluster state |
| **Synced** | Git state = Cluster state ✅ |
| **Healthy** | App resources running properly |
| **Degraded** | Some resources not ready |
| **Manifest** | YAML file describing K8s resources |
| **Repo Server** | ArgoCD component fetch từ Git |
| **Application Controller** | ArgoCD component do sync logic |
| **Pull-based** | Cluster chủ động kéo (safer) |
| **Push-based** | External system push (risky) |
| **GitOps** | Using Git as source of truth |
| **Kustomize** | Tool để customize K8s manifests |
| **Helm** | K8s package manager |
| **Declarative** | Khai báo desired state |
| **Imperative** | Chỉ thị từng step (old way) |

---

## 🎓 Assessment: Bạn Đã Hiểu Chưa?

### Sau khi học xong, bạn có thể trả lời:

- [ ] ArgoCD là gì? Tại sao cần dùng?
- [ ] GitOps workflow là gì?
- [ ] ArgoCD có những component nào?
- [ ] Pull-based vs Push-based deployment khác gì?
- [ ] Khi nào dùng Auto-Sync vs Manual-Sync?
- [ ] Làm sao deploy app lần đầu?
- [ ] Làm sao update app?
- [ ] Làm sao rollback?
- [ ] OutOfSync status là gì?
- [ ] Cách debug khi app lỗi?

Nếu trả lời được hết → **Bạn đã sẵn sàng!** 🚀

---

## 🤝 Need Help?

### Resource
- **Official Docs:** https://argo-cd.readthedocs.io/
- **GitHub:** https://github.com/argoproj/argo-cd
- **Examples:** https://github.com/argoproj/argocd-example-apps
- **Community:** Slack (https://argoproj.github.io/community/)

### In This Repo
- Quick reference → Use `ARGOCD_QUICK_REFERENCE.md`
- Troubleshooting → See each guide's troubleshooting section
- Hands-on issues → Follow ARGOCD_HANDS_ON.md step by step

---

## 🚀 Next Level (Optional)

Sau khi master cơ bản, bạn có thể học:

1. **Sealed Secrets** - Quản lý secrets an toàn
2. **Helm Integration** - Dùng Helm charts với ArgoCD
3. **ApplicationSet** - Auto-generate applications
4. **Webhooks** - Instant sync thay vì polling
5. **Multi-cluster** - Manage nhiều Kubernetes clusters
6. **Notifications** - Slack, email alerts
7. **RBAC** - Role-based access control
8. **Custom CRDs** - Extend ArgoCD

---

## 📅 Suggested Study Schedule

```
Week 1 (Concepts):
├── Mon: Read ARGOCD_LEARNING_GUIDE.md Part 1-2 (2 hours)
├── Tue: Read ARGOCD_LEARNING_GUIDE.md Part 3-4 (2 hours)
├── Wed: Study ARGOCD_DIAGRAMS.md (1 hour)
├── Thu: Read ARGOCD_LEARNING_GUIDE.md Part 5-8 (2 hours)
└── Fri: Review & Q&A (1 hour)

Week 2 (Practice):
├── Mon: ARGOCD_HANDS_ON.md Lesson 1-2 (3 hours)
├── Tue: ARGOCD_HANDS_ON.md Lesson 3-4 (3 hours)
├── Wed: ARGOCD_HANDS_ON.md Lesson 5-6 (3 hours)
├── Thu: Experiment & Challenge (2 hours)
└── Fri: Review & Present learnings (1 hour)

Total: 22 hours → Solid understanding
```

---

## ✅ Checklist: Bạn Đã Sẵn Sàng?

- [ ] ArgoCD server running (`kubectl get pods -n argocd`)
- [ ] Port forwarding active (`http://localhost:8080` accessible)
- [ ] Can login to UI (admin / O8oglSFqjFruhH2c)
- [ ] Tất cả tài liệu learned guides ready
- [ ] Terminal mở, kubectl configured
- [ ] Minikube cluster running
- [ ] Sẵn sàng làm bài tập

**Nếu hết check → Bạn sẵn sàng bắt đầu!** 🎉

---

## 🎯 Final Goals

By the end of this learning path:

```
✨ Conceptual Mastery
   ├── Hiểu GitOps philosophy
   ├── Nắm ArgoCD architecture
   ├── Biết pull-based benefits
   └── Understand sync lifecycle

⚙️ Practical Skills
   ├── Deploy app with ArgoCD
   ├── Update manifests & auto-sync
   ├── Rollback using Git
   ├── Multi-env management
   └── Debug issues effectively

🚀 Ready for Production
   ├── Write proper manifests
   ├── Structure repo correctly
   ├── Apply best practices
   ├── Monitor & maintain apps
   └── Handle disasters
```

---

**Happy Learning!** 🎓 Nếu bạn có câu hỏi, hãy hỏi bất cứ lúc nào!

---

**Last Updated:** January 31, 2026  
**ArgoCD Version:** v3.2.6  
**Kubernetes:** Minikube  
**Status:** Ready for learning ✅
