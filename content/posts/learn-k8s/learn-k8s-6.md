---
date: '2026-09-30T19:58:10+08:00'
draft: false
title: '學習 Kubernetes (K8s) 之路 - Part 6 Service'
tags: ["Kubernetes", "DevOps"]
description: '學習 K8s 的 Service 元件'
---

## Service

到目前為止，我們可以用 **Deployment** 幫我們管理多個併行應用的自動增減，但是每個 **Pod** 都有自己的 **IP** ，而且每次移除並啟動新的 **Pod** 時， **IP** 都會不同。

這時就需要 **Service** 來幫我們做自動的導流，我們可以建立 **Service** 連接 **Deployment** ，並透過它來存取應用，**Service** 就會像反向代理一樣，將請求發送至其中一個 **Pod** ，而不需要知道每個 **Pod** 的 **IP** 。

常見的 **Service** 有幾種：

1. ClusterIP - **K8s** 內部使用
2. NodePort - 公開給 **K8s** 外部使用
3. LoadBalancer - 同 **NodePort** ，但要雲端服務供應商有支援

我們直接用 **NodePort** 來實際操作。

### 方法一：直接建立 **Service**

```bash
kubectl expose deploy hello-from-deploy --name=hello-from-svc --type=NodePort --port=80
```

參數說明：
- `kubectl expose deploy hello-from-deploy` - 為 **Deployment** `hello-from-deploy` 建立 **Service**
- `--name=hello-from-svc` - **Service** 命名為 `hello-from-svc`
- `--type=NodePort` - 建立 **NodePort** 類別的 **Service** ，若不指定，預設為 **ClusterIP**
- `--port=80` - **Service** 要連到 **Pod** 的 `port`

### 方法二：用設定檔建立 **Service**

建立資源的指定大多可以加上 `--dry-run=client` 來輸出設定而不執行，建立 **Service** 也一樣：

```bash
kubectl expose deploy hello-from-deploy --name=hello-from-svc --type=NodePort --port=80 --dry-run=client -o yaml > hello-from-svc.yaml
```

`hello-from-svc.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello-from-deploy
  name: hello-from-svc
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: hello-from-deploy
  type: NodePort
status:
  loadBalancer: {}
```

用設定檔建立：

```bash
kubectl apply -f hello-from-svc.yaml
```

列出 **Services**

```bash
kubectl get svc
```

![列出Services](/image/learn-k8s-6-1.png)

可以看到我們已經建立了 **Service** `hello-from-svc` 。

接著可以嘗試透過 **Service** 來存取 **Pod** ，有個要注意的點是，因為是用 **Docker** 執行 **Minikube** ，因為 **Docker** 網路的限制，必須要透過 **Tunnel** 才能從本地連到 **Docker** 中的 **K8s** 的 **Service** 。

用 **Minikube** 啟動 **Tunnel** ：

```bash
minikube service hello-from-svc
```

![hello-from-svc tunnel](/image/learn-k8s-6-2.png)

在測試之前，讓我們觀察一下 **Minikube** **Tunnel** 的資訊，可以看到， **Tunnel** 是將 `127.0.0.1:52290` 轉送到 **Docker** **Bridge** 中的 `192.168.49.2:31938` ，所以如果不是用 **Docker** ，而是用 **VirtualBox** 執行 **Minikube** ，是可以不用透過 **Tunnel** ，直接連到 **Service** 的。

在 **Tunnel** 啟動的狀態下，另外開啟 **Terminal** 執行 `curl` 測試：

```bash
curl localhost:52290
```

![hello-from-svc tunnel](/image/learn-k8s-6-3.png)

多執行幾次，可以看到 **Service** 自動將請求分配到不同的 **Pod** ！