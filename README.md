# Kubernetes Guestbook 學習專案

這個專案以 Kubernetes 官方的 [PHP Guestbook with Redis](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/) 為主線，透過同一套應用逐步學習 Kubernetes，而不是每個主題換一個不相關的範例。

## 架構

```text
Browser
  │
  ▼
frontend Service ──► frontend Pods × 3
                         │
             ┌───────────┴───────────┐
             │ write                 │ read
             ▼                       ▼
redis-leader Service        redis-follower Service
             │                       │
             ▼                       ▼
redis-leader Pod × 1        redis-follower Pods × 2
```

每組 Pod 都由 `Deployment → ReplicaSet → Pod` 管理；Service 則透過 labels/selectors 找到對應的 Pod。

## 目前環境（2026-08-17 已驗證）

- Ubuntu WSL 2
- kubectl 1.36.3
- Minikube 1.38.1
- Kubernetes 1.35.1，節點 `minikube` 為 `Ready`
- kubectl context：`minikube`
- Container runtime：Docker 29.2.1
- Guestbook：leader × 1、follower × 2、frontend × 3，全部 `Running` / `Ready`
- Service：`frontend`、`redis-leader`、`redis-follower`，皆為 `ClusterIP`

## 學習路線

### Level 1：看懂基本物件

- Deployment、ReplicaSet、Pod、Service
- 追蹤 owner relationship
- 用 Service selector 對照 EndpointSlice 與 Pod labels

### Level 2：副本與自我修復

- 刪除 frontend Pod，觀察 ReplicaSet 補回
- 將 frontend 從 1 擴到 5 replicas，再縮回 3
- 觀察 Service 如何持續指向所有 Ready Pod

### Level 3：版本發布

- 修改 frontend image version
- 觀察 rolling update 與新舊 ReplicaSet
- 使用 `rollout status`、`history`、`undo`

### Level 4：應用設定與健康管理

- 用 ConfigMap 管理非敏感設定
- 用 Secret 管理示範憑證（Secret 預設不是加密保管庫）
- 加入 CPU / memory requests 與 limits
- 加入 livenessProbe、readinessProbe

### Level 5：接近完整部署

- 建立 `guestbook` Namespace
- 啟用 Minikube Ingress Controller 並設定 Ingress
- 啟用 Metrics Server 並設定 HPA
- 以 PVC / PV 為 Redis leader 加入持久化

> 官方 Guestbook 是教學用的簡化架構，不是 production-ready Redis 高可用方案。Level 5 的持久化會用它理解儲存概念，不代表完成 Redis HA。

## 啟動與查看

在 Ubuntu WSL 2 中執行：

```bash
sudo systemctl start docker
minikube start
kubectl get nodes
kubectl get deployments,replicasets,pods,services
kubectl port-forward svc/frontend 8080:80
```

最後一個指令需保持執行。接著開啟 <http://localhost:8080>；按 `Ctrl+C` 停止轉送。

## 重新套用官方基準版本

```bash
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-leader-deployment.yaml
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-leader-service.yaml
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-follower-deployment.yaml
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-follower-service.yaml
kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-deployment.yaml
kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-service.yaml
```

`kubectl apply` 可以重複執行，但如果後續課程已修改相同資源，它會把欄位同步回官方版本。進行 Level 3 之後，執行前應先確認差異。

## 暫停、恢復與清理

```bash
# 保留資源並停止叢集
minikube stop

# 恢復叢集
minikube start

# 刪除整個練習叢集；Guestbook 資料會消失
minikube delete
```

## 下一步

從 Level 1 開始，先執行：

```bash
kubectl get deployment,replicaset,pod,service
kubectl get pod --show-labels
kubectl get endpointslices
```

觀察重點：哪個 ReplicaSet 屬於哪個 Deployment、Pod 名稱如何包含 ReplicaSet hash，以及 Service selector 如何對上 Pod labels。
