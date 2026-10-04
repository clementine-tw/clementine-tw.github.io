---
date: '2026-10-04T13:28:27+08:00'
draft: false
title: '學習 Kubernetes (K8s) 之路 - Part 8 ConfigMap (2)'
tags: ["Kubernetes", "DevOps"]
description: '學習 K8s 的 ConfigMap (2)'
---

## ConfigMap

上一篇說明了建立 **ConfigMap** 的方法，這篇用 **ConfigMap** 來設定 **Deployment** 的環境變數。

因為要實際測試 **Deployment** 的 **Pod** 有沒有將 **ConfigMap** 設定到環境變數，需要特別製作一個會讀取環境變數的 **Docker Image** 來用，比較麻煩，所以我們直接看如何在 **Deployment** 的設定檔中引用 **ConfigMap** 作為環境變數。

特別要注意的是，當要在 **K8s** 中建立引用 **ConfigMap** 的 **Deployment** 時，被引用的 **ConfigMap** 必須要先建立好， **Deployment** 才能正確的引用到 **ConfigMap**。

### 1. 引用整份 **ConfigMap**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: app-deploy
  name: app-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app-deploy
  strategy: {}
  template:
    metadata:
      labels:
        app: app-deploy
    spec:
      containers:
      - image: my-app
        name: my-app
        resources: {}
        envFrom:
        - configMapRef:
          name: app-configmap
status: {}
```

這個設定檔直接引用了整份 **ConfigMap** `app-configmap`，若 **ConfigMap** 所有的值都是我們需要的，這個方式可以很方便的把所有的鍵值對應都載入到 **Container** 中。

若用這個方式載入，**Container** 中的環境變數名稱跟值會跟 **ConfigMap** 的鍵值一樣，無法像單一引用的方式做變數名稱與鍵的對應。

### 2. 引用 **ConfigMap** 中特定的值

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: app-deploy
  name: app-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app-deploy
  strategy: {}
  template:
    metadata:
      labels:
        app: app-deploy
    spec:
      containers:
      - image: my-app
        name: my-app
        resources: {}
        env:
        - name: ENV_VAR_1
          valueFrom:
            configMapKeyRef:
              name: app-configmap
              key: KEY_1
        - name: ENV_VAR_2
          valueFrom:
            configMapKeyRef:
              name: app-configmap
              key: KEY_2
status: {}
```

這個設定檔中，我們將 **ConfigMap** `app-configmap` 的 **Key** `KEY_1` 的值作為 **Container** `my-app` 的 **環境變數** `ENV_VAR_1` 的值，`ENV_VAR_2` 也是同理。

單獨引用的方式，讓我們可以控制 **ConfigMap** 中哪些值是不要的，也可以從不同的 **ConfigMap** 各自取出我們需要的值。

也可以看到，我們可以將 **ConfigMap** 中的鍵，對應到 **Container** 中與鍵不同的名稱的變數。
