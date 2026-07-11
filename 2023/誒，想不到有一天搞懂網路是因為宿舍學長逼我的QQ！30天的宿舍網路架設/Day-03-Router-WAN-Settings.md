# Day-03 - 從 Router WAN 設定學網路知識

昨天我們聊完了內部管理的 LAN Settings 之後，是時候面對怎麼跟外面的網路世界互動了。一樣，讓我們上圖片慢慢看。

> ![ASUS Router WAN Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-Router-WAN-Setting.png)

## WAN Setting

開始要跟外面的夥伴互動了，身為一個可愛的 Router 要有自己的 IP 讓網路世界出現自己的一席地位了吧！那這邊剛好藉由宿舍網路給的資訊設定吧。

> ![Easy Device Example of WAN & LAN](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/WAN_LAN_Graph.png)

### Internet Connection 設定

先來看看整體設定，可以發現他有分成幾個不同的設定區域，讓我們一個一個慢慢看。

#### Basic Config - 基本設置

在 Basic Config 的部分，可以看到第一個選項就是 WAN Connection Type，也就是 WAN 的連接方式，一般來說，我們獲取網路的來源往往是來自於中華電信之類的 ISP（Internet Service Provider, 網際網路連線服務公司），或者其他網通設備後面。因此當然首先就是要幫我們的 Router 設定一個獲取 IP 的方法了。（這裡的 IP 是這台設備在外面別人跟他互動的對外 IP）

下方的示意圖就是一般來說剛今天網路連線是透過中華電信小烏龜，跟在學校配合學校的設定去使用自己的 Router 時的結構狀況。

> ![Router from ISP](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Graph-of-Router-from-ISP.png)
> 
> ![Router from School Dormitory](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Graph-of-Router-from-School-Dormitory.png)

那今天讓我們看看在 Internet Connection 在選擇 Connection Type 時的選項有哪些。

> ![ASUS Router WAN Setting Internet Connection Connection Type Option](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-Router-WAN-Internet-Connection-ConType.png)

##### Automatic IP & Static IP 選項

先來講 `Automatic IP` 跟 `Static IP` 這兩個選項，一般來說會使用這兩個選項做網路操作時，就是前面還有其他正在分配 IP 的 Router，那 `Automatic IP` 就是從前面的設備由 DHCP 取得 IP；而 `Static IP` 就是依照分配指定的 IP 進行相關設定，讓我們分別看看兩者的截圖。

> ![ASUS Router WAN Setting Internet Connection Automatic IP](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-Router-WAN-Internet-Connection-AutoIP.png)
> 
> ![ASUS Router WAN Setting Internet Connection Automatic IP](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-Router-WAN-Internet-Connection-StaticIP.png)

可以看到 Automatic IP 相比設定 Static IP 直接少了 WAN IP Setting 一大個欄位。那 WAN IP Setting 跟昨天提到的 LAN IP Setting 很像，不外乎就是填上我們的 IP / Subnet Mask / Default Gateway。
另外由於 DHCP 也會比如 DNS Server 相關資料都設定好，因此當設定 Static IP 時，也會需要填上相關資訊，跟昨天一樣的說明，由於我自己有另外開 DNS Server，因此是指向自己的服務，要不然可以直接使用 Google 跟 Cloudflare 所提供的 `1.1.1.1` 跟 `8.8.8.8` 即可，又或者如我們是學校的網路設定時，他們也會直接提供可以連線的位置歐。

> ![School Dormitory IP Setting Documents](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/School-Dormitory-IP-Setting.png)

##### PPPoE / PPTP / L2TP 選項

這三個選項的到底是什麼？差異又是什麼呢？讓我們來一個一個看。

首先我們先聊一個詞彙叫 PPP（Point-to-Point Protocol，點對點協議），顧名思義我們可以知道他是一個在兩個節點上建立直接連線的方法，並提供在這個連接上的認證、傳輸加密跟壓縮的方法。白話文翻譯就是，他是一種直接把資料/資訊從 A 送到 B 的方式，而且這個中間的連線不限制在我們所說的 HTTP / FTP 等等。一般來說 PPP 協議被用在例如說電話上面，而在網路被發明之後，ISP 也直接使用了 PPP 為使用者提供 Internet 的撥接連線，原因是因為 IP 的報文本身是無法在沒有 Data Link Protocol（資料鏈路協定）的狀況下從 Router 送出去。

那 PPPoE（Point-to-Point Protocol over Ethernet）就是乙太網上的點對點協議，是一種把 PPP 封裝在 Ethernet 框架中的一種網路協議。那因為有 PPP 的部分，因此我們可以針對這個連線做到如身份驗證等功能。那一般來說，目前 ISP 在提供網路連線跟 IP 給一般使用者時，都是使用 PPPoE 的方式提供。

那 ISP 在驗證你是不是他底下的使用者時，使用的方法就是依靠 username / password 等方式做連線，因此從設定頁面可以看到多了一個 Account Settings 的部分，讓 ISP 可以驗證你的連線並且提供 IP 給你。

那由於 ISP 提供的 WAN 相關設定是可以透過 PPPoE 互動後回傳來協助 Router 完成設定，因此可以選擇 `Get WAN IP Automatically`。但如果希望可以不要受到 ISP 的一個區段 IP 的動態影響，可以直接寫死，因為你在跟中華電信購買網路時，是有直接購買一個固定 IP 的。

而 PPTP 跟 L2TP 則都比較像是 VPN 的存在，主要是為了實施私人網路的相關設定，這邊我們就暫時跳過，但基本上一樣會多了身份驗證相關設定。

### Dual WAN 設定

Dual WAN 為 ASUS Router 提供的一個特殊功能，就是允許設備擁有兩個 WAN 連線，那這樣有什麼好處，就是允許我們在某一個 WAN 線路釣線時，可以做到另外一個連線維持，或者依照流量狀況分散流量附載。相關功能可以再閱讀官網說明書操作，我們就不過多贅述了。

- [[無線路由器] 如何設定雙 WAN 的故障轉移(Failover)和負載平衡(Load Balance)功能? - ASUS](https://www.asus.com/tw/support/FAQ/1011719/)

### Port Trigger & Virtual Server / Port Forwarding 設定

接下來要介紹的兩個設定跟我們進行設備互動有直接關係，因此我們介紹上直接從情境開始介紹。

我們說過了，Router 身為網通設備，除了確保內部設備能正確的連線到外面的網路世界以外，身為類似大樓管理員的身份的他，也要協助外部的送貨員正確的把包裹送到你我的手上，畢竟網路連線往往是雙向的。

那今天我只有一個 Router，拿著一個對外的 IP，今天我要怎麼讓外面的人直接 & 正確地找到躲在 Router 後面的設備或者服務呢？那就要靠 Port 這個東西了。

Port 中文叫做埠，是一個配合 IP 運作的小可愛，由於我們說過 IP 有唯一性，但我們有那麼多服務跟功能正在運作，一個 IP 當然是不夠幫我分類歸類的，因此 Port 的存在就類似幫忙說明房間的房號，舉例來說，ssh 服務就是預設使用 22 port，因此我們要利用 ssh 連線另外一個設備時，就可以使用 `<設備 IP>:22` 的方式做連線。

那 Port 的數字範圍是 0 ~ 65535，那一般來說如剛剛提到的 ssh 等常見服務，是已經有預設的 Port 號了，例如 HTTP 就是 80 Port、HTTPS 則是 443 Port 等等，但除了這些通則以外，剩下的 Port 都是可以隨意使用的。

回歸到情境，我今天為了更方便的寫程式跟測試環境，我有在我的宿舍跑了一台小電腦並且安裝 PVE（關於什麼是 PVE 跟怎麼安裝設定之後會再說明），那 PVE 的相關設定是會使用他們提供的 GUI 直接在網路上操作的，而我的 PVE IP 在 router 的下是設定為 `192.168.50.252` 的 Static IP，而我要從外面連回去宿舍網路操作 PVE 來幫忙開虛擬機時，會需要 VPN 之類的服務才能連線到環境內網做操作，原因是因為這台 PVE 並沒有在外部網路世界中有一個身份跟存在，那怎麼辦呢？我也只有一個給 router 的 IP 而已啊 QQ。

這時就會需要 Port Forwarding 或者 Port Trigger 的功能了，簡單來說兩者都是提供一個簡單的轉送，將一個 port 指向一個服務，來做到我直接訪問唯一的對外 IP + Port 就可以訪問到特定機器。

回到剛剛的例子，由於我有一台 PVE Service，其 IP 為 `192.168.50.252` 且 Port 為 8006，那今天我希望訪問我對外 IP `59.4.8.7` 的 20000 Port。我等於要讓 `59.4.8.7:20000` 等於內網 `192.168.50.252:8006`，那我們先用 Port Forwarding 的功能來完成這個目標。

- Port Forwarding Settings:
  - 打開 `Add Profile` 之後，會看到以下畫面，
    > ![ASUS Router WAN Port Forwarding Settings](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-Router-WAN-PortForwarding.png)
  - 可以看到裡面有 `Service Name` / `Protocol` / `External Port` 等設定期分別代表:
    - `Service Name`：服務名稱，可以選填，我這邊是填寫說明這是 PVE Service。
    - `Protocol`：協議，裡面的選項有 `TCP` / `UDP` / `BOTH` / `OTHER`，那這邊是選擇跟這個服務的互動協議，由於 PVE Service 可以想像成一般網站，因此我們就選擇 `TCP` 即可。
    - `External Port`：對外 Port 號，我們要指定一個 Port Number 來代表這個服務，否則沒辦法做服務對應，那如剛剛的情境，我希望 20000 Port 會 Mapping 到 PVE 服務，因此這邊填上 `20000`。
    - `Internal Port`：內部 Port 號，由於這台目標機器可能不只一個服務，為了正確的顯示資料，可以選填內部 Port 號，那這邊有提到 PVE 的 Port 是 8006，因此我們就選填 `8006` 到這邊。（一般建議就算整台機器只有一個服務，還是直接指定 Internal Port 以避免任何錯誤發生）
    - `Internal IP Address`：內部機器的 IP Address，選擇我們內部的 IP 才能提供服務，這邊我的 PVE 是開在 `192.168.50.252`，所以填在此欄位。
    - `Source IP`：對外 IP，由於我的 Router 可以有 Dual WAN 的功能，因此我們可以有超過一個 Source IP 來做使用，為此我們可以指定要走哪一個 WAN 出去而指定 Source IP，但在我的情境中沒有這個狀況，因此就不用設定。
到這邊我們就成功讓我們的服務在 `59.4.8.7:20000` 做提供啦～可喜可賀！

那剛剛有提到還有另外一個做法，叫做 Port Trigger，Port Trigger 你可以想像成一個動態的 Port Forwarding 功能，在一個區段內的 Port 都可以觸發一個類似及時建立的 Port Forwarding 行為，他有他的好處跟壞處，這部分的設定研究可以大家再自己研究觀看。

### DMZ / DDNS / NAT Passthrough 的設定

這邊先讓我滑水跳過，等之後用到再來解釋～

## Reference

- [對等協定 - Wifipedia](https://zh.wikipedia.org/zh-tw/%E7%82%B9%E5%AF%B9%E7%82%B9%E5%8D%8F%E8%AE%AE)
- [關於 ppp、PPPoE、PPTP、L2TP、IPSec 協議的簡單認識](https://www.796t.com/content/1550256301.html)
- [網際網路連線](https://sc1.checkpoint.com/documents/SMB_R80.20/GSG/V1/zh-TW/Content/Topics-V1/FTW-Internet-Connection.htm?TocPath=%E4%BD%BF%E7%94%A8%E3%80%8C%E9%A6%96%E6%AC%A1%E8%A8%AD%E5%AE%9A%E7%B2%BE%E9%9D%88%E3%80%8D%7C_____7)
- [[無線路由器] 如何在華碩路由器設定網際網路連線? - ASUS](https://www.asus.com/tw/support/FAQ/1011715/)
