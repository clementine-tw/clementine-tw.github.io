---
date: "2026-09-27T12:57:14+08:00"
draft: false
title: "學習 Kubernetes (K8s) 之路 - Part 2"
tags: ["Kubernetes", "DevOps"]
description: '初步認識 K8s'
---

## 概覽

在開始實際操作 **K8s** 之前，我們需要對 **K8s** 有初步的認識。

容器是現在一個熱門的建置與部署應用的方式，容器可以將應用所需的依賴與應用包在一起，將應用與其依賴從作業系統獨立出來，同時不同的應用可以各自更新所需依賴的版本，而不互相影響。

**K8s** 提供了一個同時包含高可用及可擴展，且支援分散式系統的容器管理介面。

## K8s Cluster

**K8s** 的系統架構可以視為一個叢集 **K8s Cluster** ，包含了兩個部分：

1. Control Plane Node - 負責運行協調 **K8s** 各個元件的控制中樞的機器，也是 `kubectl` 控制指令的橋樑
2. Worker Node - 運行容器化應用的機器

這裡的 **Node** 可以是一台實體的機器，也可以是虛擬機器。

**Minikbe** 的 **All-In-One Single-Node** 安裝模式就是將這兩個部分放在同一個 **Node** 上，也就是 **Control Plane** 與 **Worker** 在同一個 **Node** ，也因此降低容錯能力，只適合用來學習與測試。

正式環境的 **K8s** 會將 **Control Plane** 與 **Worker** 各自部署在一個或多個 **Node** ，增加系統的容災能力。

## Namespaces

**Namespace** 是 **K8s** 用來區隔環境的手段， **K8s** 本身運作的元件都在 `kube-*` **Namespace** 下，而使用者建立的資源等等，若沒有特別指定，都會在 `default` **Namespace** 下。

使用者可以自己指定建立的資源要放在哪個 **Namespace** 下，藉此來區隔不同部門、不同生產環境或是不同用途等等，但一般來說不建議放在 `kube-*` **Namespace** 下，因為那是 **K8s** 系統使用的。

查看現有的 **Namespace** ：

```bash
kubectl get namespace
```

## Labels

在 **K8s** 各種資源的設定檔中，常常可以見到 `labels` ， `labels` 是一個鍵值對應的集合，可以說是資源的標籤。

`labels` 本身沒有特別的意義，但是可以在各種管理資源的設定去作為選擇的依據。