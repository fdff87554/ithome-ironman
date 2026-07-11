# Day-12 - 我還要更快！用 AdGuard 自建 DNS Server 吧！

到現在我們還缺了一個小拼圖，那就是我們之前一直提到的屬於我們自己的 DNS Server！讓我們來了解一下什麼是 DNS Server 跟怎麼建立吧～

## 召喚 DNS Server 開始～

在開始之前，我們先聊聊什麼是 DNS。大家還記得我們前面都一直打的那一連串 IP 嗎？當我們今天的互動比較單純 & 管理的環境不大時，稍微背誦幾個 IP 確實不是太難的工作，但是！當今天我取得每一個生活中的服務都需要靠記得他們的 IP 時，這就是一個大工程了。舉例來說，我們今天要找到 Google Search 是靠 `142.251.42.238` 的話，誰記得住啊喂！

誒，你說等等，你平常跟 Google 連線都是靠 `google.com`？這就對了，我們今天跟很多網路服務互動都是靠他們的 Magic Name 而不是 IP 對吧？但你有沒有想過為什麼我們可以靠這些可愛的名稱就直接找到網路服務呢？

### About DNS & DNS Server

我們前幾天有說過，網路的世界要建立連線一定是靠 IP，但現在我們卻又能靠不是 IP 的方式造訪服務，這是為什麼？其實就是靠我們的 DNS 來幫忙啦！

DNS（Domain Name System）是一個將人類可讀的網域名稱（例如 www.example.com）轉換為機器可讀的 IP 地址（例如 192.0.2.1）的系統。你可以想像成一個電話簿，我們今天靠 DNS 系統來幫我們將 `google.com` 這種 **Domain Name** 轉換成他們實際的 IP。從而讓我們再拿這個 IP 去跟網路世界互動。

那我們知道 DNS 可以幫我們做到館換名稱 to IP 了，但...是誰幫我們做這件事情？就要靠 DNS Server 們來做到啦！網路上有許多 DNS Server 的節點，當我們今天向網路世界詢問一個 Domain Name 的 IP 時，這些節點們的互動能最終提供我一個 IP 做網路連線。

那大家聽到這邊應該也知道為什麼我們會希望可以建立自己的 DNS Server 了吧？當然就是為了更快的取得一個 Domain Name 的實際 IP 啦！當我們今天有一個非常接近我們的 DNS Server 可以直接做 Domain 的詢問時，我們就不用再花過多的時間去

1. 找到能跟我們回答問題的 DNS Server
2. 我們的 DNS Server 能先幫我們儲存常用的 IP 從而減少從零開始查詢一個 Domain Name 所需要的時間

而且而且有自己的 DNS Server 還有很多好處，包含

1. 隱私：使用自己的 DNS 伺服器可以避免第三方 DNS 提供商追踪您的網絡活動。
2. 性能：本地的 DNS 伺服器可能會比公共 DNS 更快。
3. 自訂：您可以自訂哪些網域可以訪問，哪些不可以。
4. 安全：可以阻擋已知的惡意網域。

也就是說，我們甚至可以把我們自己的 DNS Server 當作另外一個防護網，做到 Domain 的黑白名單管理等等，非常的實用。

### AdGuard Home 是什麼？

「AdGuard Home 是一款用於封鎖廣告 & 追蹤之全網路範圍的軟體。」by 官方網站，可以發現它其實是一個 Agent 在幫忙過路網路流量。那 AdGuard Home 身為一個什麼封鎖廣告的東東，為什麼會跟 DNS 有關呢？原因是因為 DNS Server 的概念其實很像一個網路流量中繼器，我們今天只要不是直接透過指定 IP 來做網路連線的時候，我們的連線是不是**一定會**需要向 DNS Server 做 IP 的詢問，那是不是他其實就等於另外一種網路流量過濾器，我可以直接限制跟管理我不想互動的資訊，而 AdGuard 身為一個封鎖廣告等功能的服務提供者，其實也剛好可以身兼這個功能，且很開心他有提供。

### 如何架設自己的 AdGuard Home

那現在要安裝 AdGuard Home 其實非常懶人包齁，我們只需要事先準備好一個 Linux Server，然後依照官方 [github](https://github.com/AdguardTeam/AdGuardHome#automated-install-linux-and-mac) 教學用一個 curl 做安裝即可 `curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v`，又或者有其他安裝需求都可以直接參考官方文件，寫得非常清楚。

那這邊有一個要注意的細節

- 安裝這台 Server 的時候，請想好要使用的 Static IP，這會是我們的 DNS Server 的 IP，因為之後我們重新設定 DNS IP 時當然得會是一個 Static IP 做連線了。

其他就都沒什麼需要注意的啦～

> ![AdGuard Home Login Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/AdGuard-Home-Login-Page.png)
