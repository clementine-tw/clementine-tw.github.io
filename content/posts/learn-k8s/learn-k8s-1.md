---
date: "2026-09-26T17:02:13+08:00"
draft: false
title: "學習 Kubernetes 之路 - Part 1"
---

## 安裝 Kubernetes (k8s)

既然要學習 **k8s** ，最快的方式還是在本地操作，方便實驗各種功能，而且也不用錢。

我的電腦是 **macOS** ，以下的步驟都是在 **macOS** 系統操作，若是 **Windows** 或 **Linux** 等其他系統，就請自行查找其他文章了。

**k8s** 本身的設計就是給 **叢集系統** 方便管理，因此也支援多種叢集設定的安裝模式，為了方便學習與測試，官方推薦用 [**Minikube**](https://minikube.sigs.k8s.io/docs/)[^1] 來安裝與管理 **k8s**。

1. 安裝 [**Homebrew**](https://brew.sh/)[^2]

   **Minikube** 在 **Homebrew** 有可以直接安裝的版本，所以先安裝 **Homebrew**，已經安裝過的可以跳過這個步驟。

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

2. 安裝 **Minikube**

```bash
brew install minikube
```

3. 安裝 **Docker**

   **k8s** 是叢集管理的工具，它本身的元件也是在叢集中運作，所以需要一個可以讓 **k8s** 執行的叢集環境。

   雖然 **Minikube** 可以設定直接用本地作為叢集環境運行，但這樣會更改到許多本地電腦的環境設定，我不希望本地電腦的設定被改得亂七八糟，所以採用 **Docker** 的虛擬化環境去運行 **k8s** 。

   在有 GUI 的環境安裝並使用 **Docker** 很簡單，只要去官網下載 [**Docker Desktop**](https://www.docker.com/products/docker-desktop/) 的安裝檔並安裝就完成了。

4. 啟動 **k8s** 叢集

   在本地執行 **k8s** 所需的工具都準備好了，直接使用 **Minikube** 的指令就可以建立 **k8s** 的叢集。

   ```bash
   minikube start
   ```

   **Minikube** 會自動去找可以使用的虛擬化環境，若有多個虛擬化環境可用，則會預設使用 **Docker** 。

[^1]: **Minikube** 是眾多用來安裝與管理 **k8s** 的工具之一，因其為 **All-In-One Single-Node** 的叢集安裝設定，大多用來在本地學習或測試。

[^2]: **Homebrew** 是支援 **macOS** 、 **Linux** 或 **WSL** 的套件管理工具。
