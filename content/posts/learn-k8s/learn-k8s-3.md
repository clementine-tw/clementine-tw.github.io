---
date: '2026-09-28T14:45:00+08:00'
draft: false
title: '學習 Kubernetes (K8s) 之路 - Part 3 Pods'
tags: ["Kubernetes", "DevOps"]
description: '學習 K8s 的 Pod 元件'
---

## Pods

**Pod** 是 **K8s** 中，我們可以管理與部署的最小運算單元，一個 **Pod** 可以包含多個 **Container** 。

先嘗試建立一個 **Pod** ，在這個 **Pod** 中執行用 **Image** `pbitty/hello-from`[^1] 建立的 **Container** ：

### 方法一：直接建立

1. 用指令建立

    ```bash
    kubectl run hello-from-pod --image=pbitty/hello-from
    ```

2. 查看 **K8s** 中的 **Pods**
    
    ```bash
    kubectl get pod -o wide
    ```
    ![`kubectl get pod`的輸出](/image/learn-k8s-3-1.png)

### 方法二：先建立 **Pod** 的設定檔，再用設定檔建立

這個方法的好處是，可以保留建立 **Pod** 時的參數，需要重建或複製系統時非常方便，也是 **K8s** 的主要使用方式。

1. 用 `yaml` 格式建立 **Pod** 的設定檔
  
    ```bash
    kubectl run hello-from-pod --image=pbitty/hello-from --dry-run=client -o yaml > hello-from-pod.yaml
    ```
    參數說明：
    - `run hello-from-pod` - 在 **K8s** 建立一個名為 `hello-from-pod` 的 **Pod**
    - `--image=pbitty/hello-from` - 在 **Pod** 中用 **Image** `pbitty/hello-from` 建立 **Container**
    - `--dry-run=client` - 不直接在 **K8s** 建立 **Pod** ，而是輸出設定檔
    - `-o yaml` - 用 `yaml` 為格式輸出
    - `> hello-from-pod.yaml` - 將輸出存到檔案 `hello-from-pod.yaml`

2. 用剛建立的設定檔 `hello-from-pod.yaml` 建立 **Pod**

    ```bash
    kubectl apply -f hello-from-pod.yaml
    ```

3. 查看 **K8s** 中的 **Pods**
    
    ```bash
    kubectl get pod -o wide
    ```
    ![`kubectl get pod`的輸出](/image/learn-k8s-3-1.png)

### 嘗試連到 **Container**

  因為安全性的考量， **K8s** 是一個封閉的網路，內部有自己的 **DNS** 系統，若要公開 **Container** 所在的 **Pod** 給外部使用，必須要另外設定，我們在後面的章節中會談到。

  學習或測試階段，可以先用 `port-forward` 的方式暫時公開 **Pod** 的 `port` ，去測試我們是否可以連到 **Pod** 中的 **Container** 。

  這裡我們可以把 **Pod** 想成一台虛擬機，在這台虛擬機中運行的 **Container** 監聽的 `port` ， **Pod** 也要公開對應的 `port` ， **Container** 才能被外部存取。

  因為 `pbitty/hello-from` 監聽的是 `port 80` ，暫時將 **Pod** 的 `port 80` 公開到本地的 `port 8080`：
  ```bash
  kubectl port-forward pod/hello-from-pod 8080:80
  ```

  測試存取 `hello-from-pod` ：
  ```bash
  curl localhost:8080
  ```
  ![`curl localhost:8080`的輸出](/image/learn-k8s-3-2.png)

到這裡，我們已經成功在 **K8s** 中建立並執行 **Container** 了！

[^1]: `pbitty/hello-from` 是一個在 **Docker** 上的 **Image** ，方便用來測試在 **K8s** 中， **Container** 所在的 **Pod** 的 **IP** 。