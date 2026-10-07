---
date: '2026-10-06T17:06:02+08:00'
draft: false
title: '學習 Kubernetes (K8s) 之路 - Part 9 Volume (1)'
tags: ["Kubernetes", "DevOps"]
description: '學習 K8s 的 Volume (1)'
---

## Volume

**Volume** 是 **K8s** 的儲存裝置，依照生命週期可以分為兩種：

1. Ephemeral - 與 **Pod** 同時建立與移除
2. Persistent - 可以持久保存

### 1. Ephemeral Volume

這邊介紹幾種我認為 **RD** 比較需要知道的類型。

#### 1. ConfigMap

我們前面有提到 **ConfigMap**，它就可以作為 **Volume** 直接使用，還記得 **ConfigMap** 可以將檔案名稱與內容作為它的鍵值嗎？當我們將 **ConfigMap** 作為 **Volume** 時，它的鍵值就會轉換回掛載路徑下的檔名與內容了！

我用 [Caddy](https://caddyserver.com/) 的 [Docker Image](https://hub.docker.com/_/caddy) 來實際操作掛載 **ConfigMap** 作為 **Volume**，沒有選擇 [Nginx]() 是因為 **Caddy** 的設定檔相對沒那麼複雜，適合不需要 **Nginx** 的更多更強大的功能的情況。

先在本地準備好 **Caddy** 的設定檔 `Caddyfile`，這個設定檔簡單的讓 **Caddy** 監聽 `port 2015` 且回應 `hello world`。

```plaintext
:2015

respond 'hello world'
```

用 `Caddyfile` 建立 **ConfigMap**。

```bash
kubectl create configmap caddy-configmap --from-file=Caddyfile
```

顯示 **ConfigMap** `caddy-configmap` 設定。

```bash
kubectl describe cm caddy-configmap
```

![caddy-configmap內容](/image/learn-k8s-9-1.png)

建立執行 **Caddy** 並掛載 **ConfigMap** `caddy-configmap` 作為 **Volume** 的 **Pod**。

`caddy-deploy.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: caddy-deploy
  name: caddy-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: caddy-deploy
  strategy: {}
  template:
    metadata:
      labels:
        app: caddy-deploy
    spec:
      volumes:
        - name: config-vol
          configMap:
            name: caddy-configmap
      containers:
      - image: caddy:2.11-alpine
        name: caddy
        ports:
        - containerPort: 2015
        volumeMounts:
          - name: config-vol
            mountPath: /etc/caddy
        resources: {}
status: {}
```

```bash
kubectl apply -f caddy-deploy.yaml
```

檢查 `caddy-configmap` 有沒有掛載成功，如圖有看到 `Caddyfile` 的話就是成功了！

```bash
kubectl exec -it deploy/caddy-deploy -- ls /etc/caddy
```

![/etc/caddy下的檔案](/image/learn-k8s-9-1.png)

接著來檢查 **Caddy** 有沒有正常運作，這次直接用 `port-forward` 將本地的連接埠轉送到 **Pod** 的連接埠，會說是 **Pod** 是因為 `port-forward` 會選定 **Deployment** 中的固定一個 **Pod** 進行轉送。

因為 `Caddyfile` 監聽 `port 2015`，所以我們將本機的 `port 8080` 轉送到 `port 2015`。

```bash
kubectl port-forward deploy/caddy-deploy 8080:2015
```

另外開啟一個終端機嘗試請求，收到回應 `hello world` 就完成了！

```bash
curl localhost:8080
```

下一篇繼續介紹其它類型。