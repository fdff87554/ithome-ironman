# Day-02 - 從 Router LAN 設定學網路知識

昨天有提到 ASUS Router 的設定頁面裡面有很多可以調整跟設定的部分，每個設定都有其存在的意義跟希望解決的問題與功能，因此今天就一樣讓我們藉由 LAN 的設定選項中的可調整的項目學習功能與服務吧。

> ![ASUS Router LAN Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-Router-LAN-Setting.png)

## LAN Setting

由於 WAN 剛剛說過了，以一個節點角色來說，他屬於“對外溝通”，所以設定上來說會比較繁複，會需要跟外面互動，我們先從 LAN 區域的設定，看看怎麼管理好自己的環境吧。

首先看到 LAN 的部分，他們主要切分了 `LAN IP` / `DHCP Server` / `Route` / `IPTV` / `Switch Control` 五個功能，讓我們一個一個拆解這些功能是幹嘛用的。

### LAN IP Settings

> ![LAN IP Settings](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-Router-LAN-IP-Setting.png)
先來看看 LAN IP 這個格子裡面可以設定什麼，可以看到很乾淨的就只有兩個部分，

1. `LAN IP`
2. `Subnet Mask`

那為什麼會需要有這兩個設定值？讓我們聊聊一下現在 Router 所扮演的角色，還有 IP Address 的一些資訊。

### About IP Address & Subnet Mask

昨天有提過 IP Address 的概念非常重要，身為網路世界的地址，對於設備與設備之間的溝通有著不可或缺的地位，可以想像今天沒有 IP 這個網路地址，我們會無法正確瀏覽任何網路上的資源。

那究竟什麼是 IP 什麼是 Subnet Mask 呢？讓我們重新好好藉由這個章節在聊一次。

#### 不可或缺的 IP Address

網路世界互動的協議有很多，網路世界希望完成的任務與工作也非常繁瑣，但不管要做到什麼事情，“到達正確的位置”永遠是最首要的任務，昨天有提過，在真實世界中，我們希望從一個地方到達另外一個地方，最簡單的定位方式就是利用地址，那當然網路世界也必須要有類似的機制，因此**將 IP Address 視為網路世界的電腦設備/網路設備/伺服器等等的位置**當然再好不過了。

而昨天我們也提過，當今天會需要靠一個**唯一**的標示來做 1 to 1 的對應，那想必這個 IP 是一定要有唯一性的。那 IP 目前有分兩個版本，分別是 IPv4 跟 IPv6，簡單來說 IPv6 是因為 IPv4 不夠用了所以出現的，但這邊我們就不過多贅述兩者了，但由於 IPv4 仍然是目前最常用的標準，我們在解釋後續的資訊時會用 IPv4 的結構做說明。

##### 什麼是 IPv4

IPv4 主要是一個 32 位的位址系統，並且提供了共 2^32 個（也就是 4,294,967,296 個）不重複的 IP Address，由於 2 進位不好快速理解與閱讀，一般來說會轉成十進位，且每 1 byte(8 bits) 做表示並用 `.` 做分隔，因此其數字表示範圍會由 `0.0.0.0` ~ `255.255.255.255`。所以舉例來說，你可能拿到 `74.14.59.87` 這樣的 IP Address。

那 IP 還有分類的狀況，其主要存在的意義在於界定其可以操作跟管理的後續 IP 狀況，我們也可以藉由這個分類與等級來判定取得這個 IP 的人員是屬於一般民眾，或者機構管理人員之類的，但這部分我們先暫時不過多贅述。

那有了基本的 IP 概念，我們來聊聊 Subnet Mask。

##### 什麼是 Subnet Mask 跟為什麼需要有它？

簡單來說，Subnet Mask 是一種用來劃分 IP 網路的方法。借用一種說法就是，你可以把它想像成電話號碼的區碼。

那 Subnet Mask 主要的功用就是用來切分區域，我們可以利用 Subnet Mask 將一個大網路切分成好幾個小網路區域，這樣可以讓我們更好的進行管理。

那 Subnet Mask 會將 IP Address 切分成兩個部分，一個是 Network Address，另一個是 Host Address。其中 Network Address 的部分會由 1 做標示，Host Address 的部分則是由 0 做標示。並且由於他需要對應到 IP Address 的 32 bits，因此也會是一個格式跟 IP Address 相同的表示法。

那 Subnet Mask 有兩種表示法，一種如前所述，就是 IP 的那種表示法，如 `255.255.255.0`，另外一種則是 CIDR 表示法，利用 `/` 來標示**前綴長度**（1 的位數），舉例來說，`255.255.255.0` 的 CIDR 表示法為 `/24`，因為前面的 1 bit 的長度有 32 - 8 = 24 位數。

那到底 Subnet Mask 會把整個東西切分成怎樣的狀態呢？讓我們直接用一個範例來看看。

還記得上面我所截圖的 Router 設定嗎？可以看到我的 LAN IP 設定為 `192.168.50.254`，然後我的 Subnet Mask 設定為 `255.255.255.0`，那這樣的設定代表著什麼意義？

1. 我目前的這台 Router 的 IP 是 `192.168.50.254`
2. 並且他屬於 `192.168.50` 這個 Subnet 底下，也就是他跟 `192.168.50.0` ~ `192.168.50.254` 這些 IP 都是同一個子網域底下的。
3. 當他今天要找 `192.168.50.xxx` 的設備時，他不用再往外查詢，只需要直接跟目前互動的 Router 做詢問就好。
4. 當他今天希望找到 `192.168.51.xxx` 的設備時，他就會知道不是由目前的 Router 做維護，會再往上查詢其他資訊。

Subnet Mask 管理範圍跟控制範圍做一個界定還有調整，讓網路設備的互動更有規則跟目標方向。

### 回到 LAN IP Setting

所以由於今天我將 ASUS Router 的 IP 設定在 `192.168.50.254`，並且子網路遮罩設定為 `255.255.255.0`，因此我的子網路網域為 `192.168.50` 網段，總共會有 256 個 hosts 可以使用，但由於 255 為特殊 IP，跟扣除自己，總共會有 254 個 hosts 是 usable 的。

### DHCP Server 的設定

當今天有設備與 Router 連接時，要跟 Router 要一個 IP 來做使用，否則想當然你的設備就沒有網路可以使用了，這個要 IP 的概念就跟你在房子裡面借房間很像，你可以想像成租用一個位址。那前面也有提到了，我們的 IP 有分靜態 IP 跟動態 IP，也就是今天 Router 是固定分一個寫死的 IP，或者動態調整給你 IP。

那先來說說為什麼會有靜態跟動態 IP 的差異跟應用，我們想像一下你今天是一個咖啡廳的 Wifi，今天來咖啡廳的客戶想跟你使用網路的時候，是不是都需要一個由 Router 核發的 IP 才能使用，但是，以剛剛上述的 IP Setting 來說，這台 Router 最多可以發出去的 IP 只有 254 個，而你的客戶又是固定獲取某一個 IP 的狀況下，你只能有前 254 個客戶可以使用網路，而且這個客戶可能只來一次。這樣對於 IP 的使用率/管理分配上，都非常不友善，而且對於使用者也非常不方便，我要先確保有哪些 IP 可以使用，然後在自己的電腦上面設定好固定下來，用完之後到另外一個環境又要重新來過，對於現在行動跟移動設備越來越多的狀況下，這樣的做法顯然已經不是一個很好的使用情境跟方式。

那為什麼還是要有靜態 IP 呢？最重要的就是管理跟分配的問題，舉例來說，以我的宿舍來講，我不可能讓宿舍的 Router 自己去分配網路孔的 IP 對吧？因為這樣會有

1. 無法固定穩定的跟宿舍設備做通訊，因為你不知道他的 IP 現在重新開機之後是什麼
2. 可能會有搶 IP 跟連線中斷的問題，因為你無法確認 IP 會不會被分配出去

因此一般來說，如果環境/設備是固定的，為了確保使用上跟一些使用情境上的穩定，會給予靜態的 IP，那如果是會時常更換、時常移動切換的設備，則動態獲取和分配 IP 就是一個比較合適的做法，今天這個設備不用了還可以給其他人用的概念。

那 DHCP（Dynamic Host Configuration Protocol）可以說是為此而生，其中文叫做**動態主機配置通訊協議**，主要的兩個用途是

1. 用於**內部網路**或**網路服務供應商**做**“自動分配 IP 位址”**給使用者的工作
2. 用於**內部網路**管理員對所有電腦的中央管理

而我手上的 ASUS Router 身為一個家用級網通設備，主要當然是標榜什麼都不用知道就可以使用的，那為了讓所有設備只要連上 Router 就能有網路通訊，DHCP 當然就必不可少了，且現在一般來說，設備都是預設獲取動態 IP 來做使用的，因此不管是網路設備或者使用的設備都是全自動的狀況下，DHCP 會幫我們處理好所有的事情。

那剛剛有提到了，跟 DHCP 要 IP 其實就跟在大樓裡面要一間房間很像，這個借用（租用）想必就會有一些限制，我們已學校教室為例，我們都可以借學校教室，但會有幾個限制

1. 辦公室或者固定教室不能借用（有可以借用的教室清單）
2. 借用要說明借用時間（不可以無限期佔用，超過使用時間要給其他人用）

那想當然，DHCP 的相關設定也要注意這些部分了。那就讓我們藉由看看 ASUS Router 的 DHCP Server Setting 來了解一下。
> ![DHCP Server Settings](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-Router-DHCP-Server.png)

從上圖可以看到，我們開啟 DHCP 服務時，可以

1. 給予分配的 IP 範圍
   * IP Pool Starting Address
   * IP Pool Ending Address
2. 給予這個 IP 的存取時間 Lease Time
3. 預設走的路由通道（就是他要藉由哪裡跟其他人互動，一般來說這個 IP 會是 Router 的 IP） Default Gateway

那後續的其他設定就是一般的網通設定如 DNS Server 等等。

那如我的截圖，我們可以了解到

1. 我的 DHCP 會發出去 `192.168.50.101` ~ `192.168.50.199` 這個區段的 IP
2. 另外他的 IP 被釋出（租約到期）的時間是 `86400` **秒**，也就是 24 hr 後會被釋出。
3. 由於我的 Router IP 是 `192.168.50.254`，所以當然預設 Gateway 就是 `192.168.50.254`
4. 最後由於我本身有開自己的 DNS Server，因此我是指向 `192.168.51.53`，如果要用如 google 或者 cloudflare 的 DNS Server 的話，一般會設定成 `1.1.1.1` 或 `8.8.8.8`。

那 ASUS Router 也有一個很貼心的功能，就是直接把一個設備分配固定的 IP。由於預設 ASUS Router 會將子網段全部給 DHCP 做分配，因此如果我們剛好只是需要一到兩個 IP 固定給某些設備的話，可以不用更改上面的預設設定，而是類似添加白名單的概念，來固定設備 IP。

### Route 的設定

Route 中文翻譯叫路由，可以想像成如何制定今天從 A 到 B 的路線的概念，我們從下面關於 ASUS 針對 LAN-Route 的設定敘述，可以發現他直接說明當今天這台 Router 後面還有其他 Router 時，可以使用這邊的設定，讓我們來看看為什麼。
> ![Route Settings](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-Router-Route-Setting.png)

這邊其實需要回歸到另外一個概念，也就是網路結構跟設備所在的位置的問題，這個我們應該會在之後再深聊，但先幫我了解就是，我現在外部網路實際上直接接通的位置就是這台 ASUS Router，並且我並沒有直接把外部網路做轉發，而是讓這台 ASUS 做處理。又由於我剛好有另外的設備被管理在其他的路由器底下，因此為了讓這些路由能直接被通訊與處理，我需要將 `192.168.51` 這個子網段的資料轉送到另外一台 Router，這時 Routes 的設定就很重要了，上方圖片的意思就是當今天有人正在查詢關於 `192.168.51` 網段的服務時，請直接幫我轉送到 `192.168.50.253` 這個 IP 去，而這個 IP 的設備正是我的另外一台 Router。

這邊還沒聽懂沒關係，我預計於後續章節討論跟盤點我的設備狀況，到時候會回來重新說明相關設定的意義。

### IPTV 與 Switch Control

最後的最後兩個功能，IPTV 跟 Switch Control，由於他對於網路相關互動比較沒有直接關係，我們就放在一起討論了。

- IPTV，直接借用維基百科的解釋，是一種網路電視相關協議。
  > 網路協定電視（英語：Internet Protocol Television，縮寫：IPTV）是寬頻電視的一種。IPTV是用寬頻網路作為介質傳送電視資訊的一種系統，將廣播節目透過寬頻上的網際協定（Internet Protocol, IP）向訂戶傳遞數位電視服務。
- Switch Control 則是關於 Router 在處理轉址等工作時的一些設定，簡單來說，當今天外部連線要跟內部設備還有服務做互動時，會有一個外網 IP 轉內網 IP 的過程，那相關設定跟加速可以在這個頁面做到。

## 總結

我們總算是講完 LAN 相關設定啦！很多東西需要理解，沒關係，我們後面會直接用實際情境說明為什麼會使用到這個部分的設定，那當然也有一些細節沒有提到，但這部分就留給有需要的夥伴們自己去稍微查詢了～

基本上 LAN 的相關設定都是為了界定設備可以怎麼更好的取得網路，還有如何互相互動，當然，這些設定的背後都代表著各種網路概念與管理概念，如何精熟這些設定我也不敢說自己非常清楚了，但實際走過一遍確實有加速我對於網路概念知識的理解，因此大家也可以玩自己家的路由器看看歐～

讓我們明天聊聊 WAN 相關設定吧！

## References

- [Introduction of Classful IP Addressing - geeksforgeeks](https://www.geeksforgeeks.org/introduction-of-classful-ip-addressing/)
- [IP Subnet Calculator - Calculator.net](https://www.calculator.net/ip-subnet-calculator.html)
- [What is an IP address and a subnet mask, in simple terms? - Digital Citizen](https://www.digitalcitizen.life/what-is-ip-address-subnet/)
- [What Is a Subnet Mask? A Beginner's Guide to Subnetting - IPXO](https://www.ipxo.com/blog/what-is-subnet-mask/)
- [DHCP (Dynamic Host Configuration Protocol) - TechTarget](https://www.techtarget.com/searchnetworking/definition/DHCP)
- [What is DHCP and why is it important? - efficient iP](https://efficientip.com/glossary/what-is-dhcp-and-why-is-it-important/)
- [IPTV - Wifipedia](https://zh.wikipedia.org/zh-tw/IPTV)
