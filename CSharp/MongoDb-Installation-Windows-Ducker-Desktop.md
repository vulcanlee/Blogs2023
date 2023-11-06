# MongoDB 系列 - 在 Windows 作業系統上安裝 Docker

這是一系列的文章，主要是在說明如何使用 .NET / C# 來進行 NoSQL 之 MongoDB 的程式開發需求，不過，既然是要探討 MongoDB 的程式設計方式，就當然需要有 MongoDB 的服務存在，這可以有三種選擇：使用 MongoDB 提供的 Atlas 雲端服務、安裝 MongoDB 服務到本機電腦或者網路主機上、直接使用現有的 Docker MongoDB Container 容器。

因為我之前並沒有特別去接觸與使用 Docker 這樣的工具，這也許是因為工作的關係與環境問題，不過，這裡，將會介紹如何在 Windows 作業系統上安裝 Docker Desktop，並且，使用 Docker Desktop 來啟動 MongoDB 的 Container 容器。

首先，我們先來看看 Docker 的網站，這裡，可以看到 Docker 這個工具的定義：

> Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly. With Docker, you can manage your infrastructure in the same ways you manage your applications. By taking advantage of Docker’s methodologies for shipping, testing, and deploying code quickly, you can significantly reduce the delay between writing code and running it in production.

Docker 是一個開放平台，用來開發、運送與執行應用程式。Docker 可以讓你將應用程式與基礎設施分開，這樣，你就可以快速的交付軟體。使用 Docker，你可以使用相同的方式來管理基礎設施與應用程式。透過使用 Docker 的方法來運送、測試與部署程式碼，你可以大幅的減少撰寫程式碼與執行程式碼的時間。

對於第一次接觸 Docker 的開發者而言，這裡有一個簡單的說明，可以讓你快速的了解 Docker 的運作方式：

> Docker is a platform for running applications in an isolated environment called a "container" (or Docker container). Applications run differently in containers than they do in virtual machines (VM). A VM virtualizes the hardware, while containers virtualize the operating system. The result is a smaller footprint and better performance than a VM. You can run Docker on Windows and Linux.

Docker 是一個平台，用來在一個稱為「容器」的隔離環境中執行應用程式。應用程式在容器中的執行方式與在虛擬機器中的執行方式不同。虛擬機器會將硬體虛擬化，而容器則是將作業系統虛擬化。結果就是容器的佔用空間比虛擬機器小，而且，效能也比虛擬機器好。你可以在 Windows 與 Linux 上執行 Docker。

在我自身開發環境中，通常幾乎都會僅使用 Windows 作業系統，因此，這裡，將會介紹如何在 Windows 作業系統上安裝 Docker Desktop，確認Docker環境是可以正常運作，在下一篇文章中，將會使用 Docker Desktop 來啟動 MongoDB 的 Container 容器。

若要在 Windows 10 / 11 作業系統中使用 Docker 這個工具，將會有兩種選擇，使用 Hyper-V 或者使用 WSL，這裡將會選擇使用 WSL 的方式。

因此，要能夠執行 Docker 的開發電腦環境，根據官網描述必須具備底下條件

* WSL 版本 1.1.3.0 或更高版本。
* Windows 11 64 位：家庭版或專業版 21H2 或更高版本，或企業版或教育版 21H2 或更高版本。
* Windows 10 64 位：家庭版或專業版 22H2（版本 19045）或更高版本，或企業版或教育版 22H2（版本 19045）或更高版本。
* 在 Windows 上打開 WSL 2 功能。有關詳細說明，請參閱 Microsoft 文檔。
* 要在 Windows 10 或 Windows 11 上成功運行 WSL 2，需要以下硬件先決條件：
  * 具有第二級地址轉換 (SLAT) 的 64 位處理器
  * 4GB 系統 RAM
  * 在 BIOS 中啟用硬件虛擬化。有關更多信息，請參閱虛擬化。

現在，我們的開發電腦已經具備上述條件了，那麼，準備開始進行 Docker Desktop 的安裝作業。

## 安裝 Docker Desktop

* 開啟 [Docker](https://www.docker.com) 網頁 
* 登入到 Docker 網站
* 點選 [Download for Windows] 按鈕
  ![](../Images/X2023-9881.png)
* 下載完成後，執行 [Docker Desktop Installer.exe] 這個安裝檔案
  ![](../Images/X2023-9880.png)
* 此時，出現 [Installing Docker Desktop 4.25.0(126437)] 這個對話出
  ![](../Images/X2023-9879.png)

  這裡提示需要使用 WSL 2 來替換 Hyper-V

  >何謂 WSL 2？
  >
  >WSL 2 是 Windows Subsystem for Linux 的第二代版本，它是一種在 Windows 10 上執行 Linux 二進位應用程式的方法。它包含一個 Linux 核心，可以直接在 Windows 上執行，而不是使用虛擬機器。有關 WSL 2 的詳細資訊，請參閱 [Microsoft 文檔](https://learn.microsoft.com/zh-tw/windows/wsl/about)。
  >
  >
  >何謂 Hyper-V？
  >
  >Hyper-V 是 Microsoft 的一個硬體虛擬化產品，可讓您在單個物理硬體上運行多個虛擬機器 (VM)。每個 VM 都是一個完整的作業系統，可以獨立運行自己的程序和應用程序。
  

* 點選 [Ok] 按鈕
* 此時，安裝程式正在進行解壓縮與安裝
  ![](../Images/X2023-9878.png)
* 安裝完成之後，點選 [Close and restart] 重新啟動作業系統
  ![](../Images/X2023-9877.png)
* 重新開機完成後，將會出現 [Docker Subscription Service Agreement] 對話窗
* 點選右下方的 [Accept] 按鈕
* 現在看到 [Finish setting up Docker Desktop] 對話窗
* 這裡使用預設設定選項，接著，點選 [Finish] 按鈕，完成 Docker 的安裝
* [Docker Desktop] 視窗將會出現在螢幕上，點選 [Sing in] 連結，登入到 Docker 網站
  ![](../Images/X2023-9876.png)
* 登入完成之後，將會重新導入到 [Docker Desktop] 視窗
* 可以回答問卷題目，或者點選 [Skip] 按鈕
* 現在，經可以看到 Docker Desktop 應用程式了
  ![](../Images/X2023-9875.png)

## 確認 Docker 可以正常運作
* 開啟命令提示字元視窗
* 再命令提示字元視窗內輸入 `docker --version`
* 將會看到現在安裝的 Docker 版本 Docker version 24.0.6, build ed223bc
  ![](../Images/X2023-9874.png)
* 現在要來確認 Docker 的容器是否可以正常運作
* 在命令提示字元視窗內，輸入 `docker run hello-world`
* 此時，將會發現到本地端沒有這個 hello-world 影像 Image 檔案存在，所以，將會從 Docker Hub 上拉取下來

```
C:\Users\vulcan>docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
719385e32844: Pull complete
Digest: sha256:88ec0acaa3ec199d3b7eaf73588f4518c25f9d34f58ce9a0df68429c5af48e8d
Status: Downloaded newer image for hello-world:latest
```
* 一旦 Image 拉取下來之後，就會透過 Container 容器來執行
* 執行後，將會看到底下文字內容

```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

![](../Images/X2023-9873.png)

* 緊接著輸入命令 [docker image ls --all]，確認 Image 真的有下載下來

```
C:\Users\vulcan>docker image ls --all
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    9c7a54a9a43c   6 months ago   13.3kB
```
* 輸入 [docker container ls --all] 確認有容器啟動了

```
C:\Users\vulcan>docker container ls --all
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS     NAMES
654e829a338a   hello-world   "/hello"   6 minutes ago   Exited (0) 6 minutes ago             peaceful_williams
```

