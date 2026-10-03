---
date: '2026-10-02T20:45:38+08:00'
draft: false
title: '學習 Kubernetes (K8s) 之路 - Part 7 ConfigMap (1)'
tags: ["Kubernetes", "DevOps"]
description: '學習 K8s 的 ConfigMap 元件 (1)'
---

## ConfigMap

**ConfigMap** 是 **K8s** 用來儲存鍵值對應的元件，可以用 **ConfigMap** 將預先準備好的環境變數值或各種設定檔，提供給其他元件使用。

**ConfigMap** 可以用各種來源建立，方便我們將預先準備好的檔案做成 **ConfigMap** ，也有利於專案的版本控管。

### 1. 從指令的文字建立

最快速直接的方法就是用 `--from-literal` 參數將內容直接填入 **ConfigMap**：

```bash
kubectl create configmap hello-cm --from-literal=name=clement --from-literal=color=blue
```

參數說明：

- `kubectl create configmap hello-cm` - 建立名叫 `hello-cm` 的 **ConfigMap**
- `--from-literal=<key>=<value>` - 填入 **ConfigMap** 的鍵值對應

查看 **ConfigMap** `hello-cm`：

```bash
kubectl describe cm hello-cm
```

可以看到 **ConfigMap** 的鍵值對應是放在 `Data` 中：

![hello-cm 的內容](/image/learn-k8s-7-1.png)

### 2. 從檔案建立

從檔案建立時， **ConfigMap** 會以 **檔名** 為 **鍵**，以 **檔案內容** 為 **值** 建立內容。

先建立兩個要作為 **ConfigMap** 的鍵值對應的檔案：

```bash
echo 'clement' > name
echo 'blue' > color
```

再以這兩個檔案作為來源建立 **ConfigMap**：

```bash
kubectl create configmap hello-cm-from-file --from-file=name --from-file=color
```

查看 **ConfigMap** 的結果會與[從指令的文字建立](#1-從指令的文字建立)相同，借用同一張圖

![hello-cm 的內容](/image/learn-k8s-7-1.png)

要同時指定多個檔案有點麻煩，所以也可以指定整個資料夾。

建立資料夾與設定檔：

```bash
mkdir settings
echo 'clement' > settings/name
echo 'blue' > settings/color
```

以資料夾作為來源：

```bash
kubectl create configmap hello-cm-from-path --from-file=settings
```

結果也會跟用檔案作為來源一樣，就不特別展示了。
