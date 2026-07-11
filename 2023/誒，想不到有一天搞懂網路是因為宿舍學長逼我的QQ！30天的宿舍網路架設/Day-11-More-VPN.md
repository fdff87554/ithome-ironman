# Day-11 - VPN 小插曲，不同的 VPN 做到不同的事情？Firewall 設定！

我們昨天完成跟我們的 VPN 連線啦！但現在有幾個新的可以玩的事情也隨之發生了。我是不是有機會不用 ASUS Router 的 VPN 連線到 50 網段？但是只有“特定” VPN Peer 可以做到這件事情。

## Firewall 的小故事

我們先回到我們的 VPN Client 端把昨天的 `AllowedIPs` 從 `10.0.0.0/24, 192.168.52.0/24` 中多給一個 `192.168.50.0/24` 後會很驚恐的發現...我們竟然連的回去 50 子網段！為什麼！讓我們先回到 IP > Routes 底下看看。

由於我們的設備目前有兩張網卡，一個接收來自 `192.168.50` 的網路流量，這也就是為什麼我們可以順利接觸到從外面來的網路流量，另一個則是接收管理 `192.168.52` 的網段，也因此其實我們的設備目前算是 `50` 子網域 & `52` 子網域的共同擁有者，那當然當我今天再由這個 RouterOS 做網路連線的時候，我們 `50` 子網域 & `52` 子網域 都是可以連線的。

但這邊會先遇到第一個問題，我們之前提過要隔離 50 的實體設備才對，這樣等於之前提過的目標完全沒有做到啊！那怎麼才能完全不接觸到 `50` 網段呢？

### `52` 到 `50` 網亂大逃殺

這時候就要靠我們的 Firewall 也就是防火牆出馬了。為了避免我們的網路流量從 `52` 網段流向 `50` 網段，我們需要制定流量規則來防止這樣的訪問產生，因此讓我們先訪問 IP > Firewall > Filter Rules 來制定過濾規則。

> ![VPN IP Firewall FilterRules Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/VPN-IP-Firewall-FilterRules-Setting.png)

我們點下 Add New 來添加一個設定，今天我們的目標是所有從 `52` 的網路流量都不應該流向 `50`。

先讓我們看向設定裡面其實有很多可以設定的內容，但今天對我們來說最重要的是 `General` 的網路相關設定跟 `Action` 的防火牆行為設定。

> ![IP Firewall FilterRules Setting Page Overview](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/IP-Firewall-FilterRules-Setting-Page-Overview.png)

那我們先把兩個設定畫面貼上來，然後嘗試用白話文說明這個設定的意思

> ![IP Firewall FilterRules General Settings](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/IP-Firewall-FilterRules-General-Settings.png)
>
> ![IP Firewall FilterRules Action Settings](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/IP-Firewall-FilterRules-Action-Settings.png)

上面的設定們的意思是這樣，今天有從 Ether2（也就是 52 網段的網卡）向 `192.168.50.0/24` 這個網段的網路流量發生時，我都要拋棄掉。這時候你用 VM 向 ASUS Router 的 `192.168.50.254` 發送 ping 的請求時候會發現網路流量沒有了！
那我們其實還可以怎麼設定，我們可以

1. 用 Interface，Ether2 向 Ether1 的網路流量全部阻止。（考題一：這樣網路會有什麼問題？）
   > ![IP Firewall FilterRules Setting with Interface](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/IP-Firewall-FilterRules-Setting-with-Interface.png)
2. 用 IP 們，把 Src Address 設定為 `192.168.52.0/24` 跟 Dst Address 設定為 `192.168.50.0/24`。
   > ![IP Firewall FilterRules Setting with IP](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/IP-Firewall-FilterRules-Setting-with-IP.png)

其他排列組合可以自己玩玩看。

這邊我們成功把 `52` 向 `50` 的網路連線阻擋了！那是不是可以稍微做其他設定來保護我們的環境？

### 堅守 RouterOS 管理畫面

現在有幾個比較敏感的環境需要被照顧，其中最重要的不外乎我們的 RouterOS 的管理介面，因此我們要做一件事情就是不允許我們這個特殊的 VPN 以外的連線連到這個管理畫面，那可以怎麼設定呢？

> ![IP Firewall FilterRules Setting of input disallow 1](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/IP-Firewall-FilterRules-Setting-of-input-disallow-1.png)
>
> ![IP Firewall FilterRules Setting of input disallow 2](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/IP-Firewall-FilterRules-Setting-of-input-disallow-2.png)

畫面中的設定的意思是當今天從 WireGuard VPN 這個 Interface 來的 input 的流量不是（Src Address 前面的 `!` 是 Not 的意思）`10.0.0.2` 這個 IP 時，是不允許流量過來的，會被 Drop 掉。

我們已經慢慢越來越熟悉 Firewall 的設定了，那如果今天希望可以只允許 10.0.0.2 這個 VPN 連線從 52 連線到 50 又可以怎麼設定呢？

### 我的 Peer 可是很特殊的！

下面就直接讓我們上範例，大家看看能不能理解！

> ![IP Firewall FilterRules Setting of input disallow - 3](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/IP-Firewall-FilterRules-Setting-of-input-disallow-3.png)

