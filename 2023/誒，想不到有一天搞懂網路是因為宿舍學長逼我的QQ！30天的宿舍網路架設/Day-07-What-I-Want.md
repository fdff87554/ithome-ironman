# Day-07 - 我想做什麼跟我還缺什麼？網路拓樸圖（Network Topology Graph）跟環境架構解說

我們現在有 PVE 在我們的內網了，但今天這台 PVE Server 一樣遇到了一個問題，他在內網，在連線上除非我直接對其做 Port Forwarding Public 在外網以外，是無法直接在外部連線的。但我這邊當然是不希望測試用跟這麼隱私的環境在外網裸奔，等等被人揍了怎辦對吧？因此讓我們來盤點一下我們的網路環境跟各種狀況，順便整理一下接下來要達成這些目標，會需要多少設備與環境吧！

## PVE 環境設定補充

由於 PVE 仍然有官方的維護資源跟社群維護資源的差異，如果你是有訂閱官方資源的，那可以直接註冊完之後使用，但如果沒有，我們在使用 `apt-get update` 的時候會發現一些 Error Message。

> ![PVE Apt Update Error](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Apt-Update-Error.png)

為此我們要稍微調整一下資料來源。從 [官方文件](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#sysadmin_package_repositories) 的說明裝可以看到 APT Repositories 的相關定義文件是由 `/etc/apt/sources.list` 跟 `/etc/apt/sources.list.d/` 裡面做維護。

那 PVE 官方有直接說明，如果你是有訂閱的使用者，想取得穩定且預設的環境，會是由 `/etc/apt/sources.list.d/pve-enterprise.list` 中的 `deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise` 維護。

如果你沒有訂閱，一樣要到 `/etc/apt/sources.list.d/pve-enterprise.list` 中註解，然後可以直接參考下方調整並把東西貼到 `/etc/apt/sources.list` 中。

```bash
deb http://ftp.debian.org/debian bookworm main contrib
deb http://ftp.debian.org/debian bookworm-updates main contrib

# Proxmox VE pve-no-subscription repository provided by proxmox.com,
# NOT recommended for production use
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription

# security updates
deb http://security.debian.org/debian-security bookworm-security main contrib
```

而 `/etc/apt/sources.list.d/ceph.list` 裡面則是調整成 `deb http://download.proxmox.com/debian/ceph-quincy bookworm no-subscription`

為了安全性簽署，還需要 `wget https://enterprise.proxmox.com/debian/proxmox-release-bookworm.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg`

## 情境盤點與宿舍網路結構拓樸圖

先來回顧一下我目前設備跟網路的關係，不管學校那邊是如何對外溝通的，我從宿舍網路取得了一個對外的實體 IP，也就是 `140.114.234.160`，那我基本上的實體設備目前有 PVE Server / Desktop / Laptop / Phone ... 等等多個設備，但直接連接牆壁裡面的網路孔的只有一台設備，也就是我的 ASUS Router，因此我們回頭看一下現在的網路架構狀況如下。

> ![Network Structure](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Network-Structure.png)

那現在這樣的運作其實沒什麼問題，對於 PVE 裡面的虛擬機來說，你可以全部想像成跟 PVE 同樣層級的機器，你應該全部將其視為一台獨立完整的設備（其虛擬機的網路設定為 Bridge），因此其可以跟 ASUS Router 直接藉由 DHCP 獲得分發的 IP，或者直接指定由 `192.168.50` 子網域的 Static IP 也是沒問題的。那感覺就沒什麼問題了對吧？畢竟依照網路拓樸圖的架構來看，設個設備的狀況應該長這樣，都在 `192.168.50` 的子網域中。

> ![Network Topology Graph](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Network-Topology-Graph.png)

誒，什麼是網路拓樸圖？怎麼突然冒出了一個新名詞？其實網路拓樸圖就是一個說明闡述設備之間網路連接狀態的一種圖示，你也可以說網路拓撲是網路的排列方式。你可以把網路世界想像成城市，將拓撲視為路線圖。當我們繪製或者閱讀網路拓樸圖時，可以直接知道網路設備之間的關係與互動。那從我們上方的網路拓樸圖可以看到我們其實目前所有對外網路都是由我們的 ASUS Router 在處理，且子網域都屬於 `50` 網段。

好，講到這裡其實都沒有什麼問題，唯一的問題是我希望把這台 PVE 的資源外借。原本我需要連線跟回來操作 PVE 是可以直接用 ASUS Router 所提供的 VPN 做連線的，原因是因為其子網域相同，所有虛擬機器都可以在 `192.168.50` 中互相互動，但這邊會有兩個問題，

1. 我的測試用的虛擬機環境竟然可以碰到我一般的其他設備
2. 當我今天想分享我的虛擬機資源時，我竟然要提供權限超高的 Router VPN 讓其他人甚至可以直接更改我的所有 Router 設定

因此到這邊我開始有幾個情境出來了，分別就是

1. 我需要獨立我的實體設備（一般設備）跟我的虛擬機器們（測試用環境 or 其他服務）
2. 我需要另外一個沒辦法直接接觸到 ASUS Router 的 VPN 服務

這也就是說，我需要有另外一個 Router 或者類似能夠切分子網域的工具來幫我將虛擬機跟實體機獨立開，且這些虛擬機器的網域中有屬於自己的 VPN 來做互動，且我非常希望廢棄使用 ASUS Router 的 VPN，因為如果不小心外流，我的所有設備跟環境都會出問題。

也因此盤點到現在，為了能做到子網域的切分，我會需要另外一個 Router 來維護一個屬於虛擬機們的世界。且需要一個新的 VPN 服務可以維護我的連線，甚至分享給別人。那我想當然是絕對沒有另外一台 Router 而且還專門給 PVE Server 做連線只為了切分子網域，因此我們直接使用 Mikrotik Routeros 來幫我們維護一個虛擬路由。且 Routeros 本身有 VPN 服務，就可以做到一樣的功能，讓我連線虛擬機器，但這些 VPN 卻不能跟我的其他實體設備做互動。

## 下集預告

因此明天開始，我們將會開始安裝 RouterOS 且慢慢依照需要的情境，從切分子網域，準備 PVE 網卡等等，將設備全部依照需求獨立。然後建立 VPN Server 到設定 Firewall 來做到不同 VPN 連線的功能狀況。
