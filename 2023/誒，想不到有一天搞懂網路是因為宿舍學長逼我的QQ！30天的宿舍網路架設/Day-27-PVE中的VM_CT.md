# Day-27 - 服務開設大哉問，我該選 VM 還是 CT 呢？

歷經前面好幾天的各種實驗與操作之後，我們可以發現我們都是使用 VM 的方式在創建服務，但其實在 Proxmox VE 中，除了 VM 之外，還有一種叫做 CT 的服務，那麼今天我們就來看看 VM 與 CT 的差異吧！

## VM 與 CT 大對決

VM（Virtual Machine）跟 CT（Container）是兩種不同的虛擬化技術，由於這部分的介紹會牽涉到很多的虛擬化技術，因此我們就盡可能白話的說明一下差異：

Virtual Machine 虛擬機，顧名思義是模擬一套硬體設備後，獨立再運作一台服務 / 電腦的技術，那 PVE 就是一種專門為了建置多個虛擬機設備的平台，而這些虛擬機設備可以是 Windows、Linux、MacOS 等等，甚至是其他的虛擬機平台，例如：VMware、VirtualBox 等等。那虛擬機的好處與優勢有

1. 可以獨立運作，不會受到其他虛擬機的影響
2. 可以獨立設定硬體資源，例如：CPU、RAM、硬碟空間等等
3. 可以獨立設定網路，例如：IP、MAC、網路卡等等

但相對的，虛擬機也有一些缺點，例如：

1. 資源消耗上因為需要模擬完整的硬體資源，因此會消耗較多的資源
2. 因為是例外的完整作業系統，再啟動的時候會需要較多的時間

而 Container 容器，則是另外一種較新的虛擬化技術，他不再從頭模擬一套完整的硬體設備，而是直接使用主機的核心，並且在上面建立一個獨立的環境，讓使用者可以在上面安裝服務，而這些服務可以是 Windows、Linux、MacOS 等等。那這樣的虛擬化情境最大的好處就是

1. 對於硬體資源的損耗有明顯的降低
2. 且運作效率高須多
3. 資源利用率高

但壞處也很明顯，其

1. 靈活性與完整性沒有虛擬機好
2. 隔離性不如虛擬機

那兩者都已經是很成熟的虛擬化技術了，如何挑選有時候界線也不是很明顯，但我自己會用以下幾點判斷：

1. 對環境的完整性跟真實性的要求，如果需要完整的環境，那就用虛擬機，如果不需要，那就用容器
2. 對於資源的需求，如果需要大量的資源，那就用虛擬機，如果不需要，那就用容器

因此當今天我是以服務的角度思考時，今天只需要特定環境做特定的事情時，Container 能提供良好的效能、低廉的成本，但今天如果是要測試實驗一些項目時，虛擬機就是一個很好的選擇。

## LXC 與 Docker

LXC (Linux Containers) 和 Docker 都是容器技術，而在 PVE 上使用的是 LXC，但之前在實驗的時候，我們有使用過 Docker，那兩者有什麼差異呢？

LXC 相比 Docker 技術更早出現，在

1. 系統完整性
2. 靈活性
3. 安全性
4. 成熟度上

在跟 Docker 相比時各有優缺，LXC 相比 Docker 可以說是更加接近 VM 的存在，但由於 Docker 技術上跟規劃上的調整，又使得 Docker 在安全性上跟靈活度上都有更好的表現，兩者各有優缺，但目前 Docker 的生態更加完整一些。

那我們今天就是試著利用 PVE 有所提供的 LXC 跟 Gitlab 服務做範例，看看跟我們之前在 Docker 上的操作有什麼不同。

### 建立 LXC 容器

首先我們要先準備 LXC 容器所需要的環境範本，那其實 PVE 已經提供很多可以供使用的範本，但如果我們直接使用官方範例下載在台灣有點慢，因此這邊推薦使用 Debian 所提供的相關載點：https://cdimage.debian.org/mirror/turnkeylinux/images/proxmox/，我們今天會示範使用的就是其中的 Gitlab 範本，`	debian-11-turnkey-gitlab_17.1-1_amd64.tar.gz`，先下載回我們的 PVE 主機上。

在完成下載後，我們點選 Create CT 後，有些關於 Container 的基本設定如下參考，

> ![Create LXC Container General](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Create-LXC-Container-General.png)

在 Template 的部分就跟選擇我們 VM 的 ISO 一樣，這邊選擇我們要使用的 Container Template 就可以了。

> ![Create LXC Container Template](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Create-LXC-Container-Template.png)

接下來的設定基本上跟 VM 的設定相似，不外乎要給予磁碟大小 / CPU 資源跟記憶體資源，這邊一樣視自己的設備給予，但請注意 Gitlab 所需要的硬體資源不算太少，從官方標準需求中可以看到至少需要 4 core CPU 跟 4 GB RAM 來提供 500 個用戶的服務支援，而最低最低的系統資源也應該要給到 2GB 的 RAM。在提供完基本的資源後，我們就可以點選 Create 按鈕來建立我們的 LXC 容器了。

- https://docs.gitlab.com/ee/install/requirements.html

> ![Created Gitlab LXC](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Created-Gitlab-LXC.png)

### 啟動 LXC 容器 - 完成 Gitlab 安裝

接下來我們會需要在 Console 中完成 Gitlab 的初次初始化設定，再看到第一次 gitlab login 時，利用 `root`:`<剛剛設定的密碼>` 進行第一次的登入，接下來就可以開始進行 Gitlab 的初始化設定了。

> ![Gitlab LXC First Login](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Gitlab-LXC-First-Login.png)

首先會詢問我們新的 gitlab 的 root 密碼，這邊會要求我們設定超過八個字元的大小寫英文數字的密碼，請牢記這個密碼，他會是整個系統最重要的密碼，然後就是包含 Email 等等設定。

> ![Gitlab LXC Root Password Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Gitlab-LXC-Root-Password-Setting.png)
>
> ![Gitlab LXC Root Email Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Gitlab-LXC-Root-Email-Setting.png)
>
> ![Gitlab LXC Root Domain Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Gitlab-LXC-Root-Domain-Setting.png)

接下來會有關於一些服務的設定，

> ![Gitlab LXC Initialize Hub server](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Gitlab-LXC-Initialize-Hub-server.png)
>
> ![Gitlab LXC System Notification](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Gitlab-LXC-System-Notification.png)
>
> ![Gitlab LXC Security Update](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Gitlab-LXC-Security-Update.png)

最後就可以看到我們的連線資訊啦！

> ![Gitlab LXC appliance services](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Gitlab-LXC-appliance-services.png)
>
> ![Gitlab Web Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Gitlab-Web-Page.png)
>
> ![Gitlab Main Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Gitlab-Main-Page.png)
