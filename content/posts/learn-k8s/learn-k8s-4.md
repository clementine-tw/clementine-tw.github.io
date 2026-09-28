---
date: '2026-09-28T16:00:00+08:00'
draft: false
title: '學習 Kubernetes (K8s) 之路 - Part 4 ReplicaSet'
tags: ["Kubernetes", "DevOps"]
description: '學習 K8s 的 ReplicaSet 元件'
---

## ReplicaSet

若一個網路服務只有一個應用程序去處理所有流量，當這個單一程序因各種原因當機而無法處理時，這個網路服務就停止運作。

所以通常一個網路服務會有多個應用程序去分散流量，不只可以降低單一程序的負荷，還可以增加系統的容災能力。

**K8s** 的 **ReplicaSet** 就是用來實現應用程序的併行，當然前提是我們的應用程序需要為了併行而設計，避免資料競爭。

**ReplicaSet** 是一個自動管理多個 **Pod** 併行的元件，幫我們保證對應的 **Pod** 應該要在我們預期的數量。

若有 **Pod** 因為某種原因而無法運作， **ReplicaSet** 會自動重啟一個新的 **Pod** 並刪除無法運作的舊的 **Pod** ，同理，若 **Pod** 的數量超出預期，多出的部分也會被刪除。

### 建立 **ReplicaSet** 設定檔

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: hello-from-rs
  labels:
    app: hello-from-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-from-rs
  template:
    metadata:
      name: hello-from-rs
      labels:
        app: hello-from-rs
    spec:
      containers:
        - name: hello-from-rs
          image: pbitty/hello-from
```

這個設定檔中有幾個值得注意的地方：

1. `replicas`

    用 `replicas: 3` 指示 **Pods** 的數量為 `3`
    ```yaml
    spec:
      replicas: 3
    ```

2. `selector`

    `selector` 用 `matchLabels` 決定 **ReplicaSet** 要管理 `labels` 包含 `app: hello-from-rs` 的 **Pods** 。

    這裡的 `app: hello-from-rs` 必須對應 `template` 部分的 `app: hello-from-rs` ， **RelicaSet** 才能正確的計算現有的 **Pods** 的數量。

    ```yaml
    spec:
      ...
      selector:
        matchLabels:
          app: hello-from-rs  
      ...
    ```

3. `template` 描述 **ReplicaSet** 的 **Pod** 以及其中的 **Container**

4. 依照慣例， **ReplicaSet** 的名稱與它管理的 **Pods** 相同，可以一眼看出這兩個資源的關聯性

### 建立 **ReplicaSet**

將設定檔存成 `hello-from-rs.yaml` ，接著用檔案建立 **ReplicaSet** ：

```bash
kubectl apply -f hello-from-rs.yaml
```

查看建立好的 **ReplicaSet** 與對應的 **Pods** ：

```bash
kubectl get rs,pod -o wide
```

![建立好的ReplicaSet與Pods](/image/learn-k8s-4-1.png)