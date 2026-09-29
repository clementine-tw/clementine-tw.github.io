---
date: '2026-09-29T20:57:41+08:00'
draft: false
title: '學習 Kubernetes (K8s) 之路 - Part 5 Deployment'
tags: ["Kubernetes", "DevOps"]
description: '學習 K8s 的 Deployment 元件'
---

## Deployment

**Deployment** 是更高一階的幫我們管理 **ReplicaSet** 的元件，它擁有幫我們管理應用的版本迭代等更強大的功能，基本上都是使用 **Deployment** 來部署應用，不會直接使用 **ReplicaSet**。

舉例來說，直接使用 **ReplicaSet** 建立的 **Pods** ，當我們要更新 **Pod** 中 **Container** 的版本時， **ReplicaSet** 不會自動將舊的替換成新的，而是要手動刪除舊的 **Pods** ，再讓 **ReplicaSet** 偵測到數量不足而重新建立新的版本。

而用 **Deployment** 建立並管理的 **ReplicaSets** 與 **Pods** ， **Deployment** 可以在偵測到資源或版本設定變動時，自動用設定好的變更行為去幫我們刪除掉舊的 **Pod** ，讓 **ReplicaSets** 逐步建立新的，管理上相對方便許多。

### 方法一：直接建立 **Deployment**

```bash
kubectl create deploy hello-from-deploy --image=pbitty/hello-from --replicas=3
```

若遇到 **Docker** 拉取 **Image** 失敗，可以試試用完整 `url` ：
```bash
kubectl create deploy hello-from-deploy --image=docker.io/pbitty/hello-from:latest --replicas=3
```

### 方法二：用設定檔建立 **Deployment**

1. 建立設定檔：
    
    用指令輸出設定檔會方便許多，也可以作為未來更複雜設定的基底：

    ```bash
    kubectl create deploy hello-from-deploy --image=pbitty/hello-from --replicas=3 --dry-run=client -o yaml > hello-from-deploy.yaml
    ```

    輸出的設定檔大概會有這些內容：

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: hello-from-deploy
      name: hello-from-deploy
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: hello-from-deploy
      strategy: {}
      template:
        metadata:
          labels:
            app: hello-from-deploy
        spec:
          containers:
          - image: pbitty/hello-from
            name: hello-from
            resources: {}
    status: {}
    ```

2. 用設定檔建立 **Deployment** ：

    ```bash
    kubectl apply -f hello-from-deploy.yaml
    ```

列出 **Deployments** 、 **ReplicaSets** 及 **Pods** ：
```bash
kubectl get deploy,rs,pod
```

![列出Deployments、ReplicaSets及Pods](/image/learn-k8s-5-1.png)

可以看到我們建立了 **Deployment** ，而 **Deployment** 自動建立並管理 **ReplicaSet** 與 **Pods** 。

### 移除 **Deployment**

```bash
kubectl delete deploy/hello-from-deploy
```

因為 **Deployment** 幫我們管理對應的 **ReplicaSets** 與 **Pods** ，所以移除 **Deployment** 時，對應的 **ReplicaSets** 與 **Pods** 也會被移除，不需要個別移除。