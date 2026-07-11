# Day-05 - 誒，我可以遠端開我的電腦了...嗎？

昨天我們總算是把我們的 WoL 開好啦～開開心心的用我們的 Router ... 誒？我的 Router 要怎麼從外面連上 QwQ？

今天我們在這台 Router 的網路底下，可以直接用 Router 的 IP `192.168.50.254` 來進入管理介面，做到設備的調整跟昨天提到的開電腦...，我都已經在房間了我遠端開電腦是在開開心的嗎 QwQ，那今天我在外面要怎麼樣才能連線到自己點 Router 呢？

還記得我們前幾天提到的 Port Forwarding 嗎？那我們應該可以直接把 Router 打出來讓外網調整對吧！當然不可以！你的網路安全性問題呢？如果今天 ASUS Router 可以直接在外網操作，那如果今天 ASUS Router 有安全性問題，是不是你的所有設備都在風險中！

那今天有什麼辦法在外面連線到 Router 但又沒有安全性問題呢？我該怎麼“安全的接觸內部網路”呢？這時候就會需要一個工具叫做 VPN。所以今天就讓我們聊聊 VPN 跟了解如何藉由 ASUS Router 提供的 VPN 服務，來完整做到遠端開機的功能吧～

## VPN 是什麼勒？

“想隨時隨地安全地使用網路嗎？想看區域限制的影片嗎？快用 Surfshar...“ 咳咳，想必 VPN 這個名詞其實大家都不陌生了，但究竟什麼是 VPN？為什麼廣告都說他可以提供安全的網路服務？安全安在哪裡呢？

VPN 的全名叫 Virtual Private Network，中文叫做虛擬私有網路。那 VPN 到底是怎麼運作的呢？

簡單來說，VPN 所保護的是使用者跟網路互動的足跡，而其作法很簡單，今天別人是怎麼知道你是誰，靠 IP 對吧？那今天要怎麼換身份？就是換 IP 換成別人，但是要怎麼換成別人呢？就是把資料傳出去由那台設備對外輸送。

所以概念上就是

1. 資料出去到 VPN Server 的過程中進行資料加密
2. 從 VPN Server 再進行解密，如果需要對外互動，則再由 VPN Server 對外進行網路連線
3. 取得資料 or 內部互動之後，加密，回傳我們的設備
4. 從我們的設備解密然後完成所有流程

因此 VPN 可以做到確保我們跟 VPN Server 互動之前，所有東西都是安全的。那今天要怎麼利用 VPN 跟內部網路互動呢？我們為了確保外部連線不應該隨意跟內部網路與設備做互動，我們本身是不允許這樣的流量進入內網的，但由於今天 VPN 服務為了我們做了身份上的保證 & 資料的保護，因此我們信任連線 VPN 之後的操作且視同其在內部網路做互動。

那我們了解 VPN 可以怎麼幫我們完成連線跟保護之後，我們來看看今天要怎麼設定 VPN 跟操作 Router 吧！

## 怎麼建立我們自己的 VPN？

從零開始建置 VPN 這件事情，我們之後會再介紹跟加深 VPN 安全相關的說明，我們先來走一條舒服的路，也就是 ASUS Router 提供的 VPN 服務吧～

ASUS Router 身為一個家用級產品，當然也幫我們考慮到了如果今天有需要外部連線內網的問題，因此從截圖上來看也會看到 VPN 的設定選項。

那概念上來說，ASUS Router 會將自己兼職成一台 VPN Server，用來驗證外部流量是否為 VPN，而設定的方法跟操作方法也很簡單，就是點選設定頁面的 VPN 選項。
> ![ASUS Router VPN Settings](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-Router-VPN-Setting.png)

那在我們可以看到選項中有 VPN Server 跟 VPN Client 兩個選項，由於我們今天希望跟 Router 做連線，因此設定就是走 VPN Server 的設定。那 ASUS Router 提供了兩種 VPN 服務的選項，一個是 PPTP，另外一個則是 OpenVPN，那我們這邊介紹 OpenVPN 的設定方法，讓我們看看設定畫面。
> ![ASUS Router OpenVPN Settings](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-Router-OpenVPN-Setting.png)

從設定畫面我們可以看到幾個基本設定，一個是 VPN Details，裡面其實包含了連線設定上的各種資訊，例如 Interface Type / Protocol 等等細節設定，裡面主要是用來包含確保 VPN 互動的 protocol 還有加密演算法之類的設定細節，那這些細節不影響我們連線 VPN，因此我們先跳過解釋，我們先設定 Server Port 的部分，我們直接使用 OpenVPN 預設的 Port `1025` 且我們只希望用來調整 Router 相關設定，因此就只允許互動內部網路而不用對外網路連線。
> ![ASUS Router OpenVPN Advanced Settings](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-Router-OpenVPN-Advanced-Setting.png)

都設定好之後，下方有一個非常方便的按鈕 Export 來幫我們匯出 Client 端的設定檔案，這時候我們的官網下載 [OpenVPN](https://openvpn.net/community-downloads/) 的 Client 之後，直接將檔案匯入，我們的設備就準備好要跟 ASUS Router 連線了。
> ![Open VPN Import](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Open-VPN-Import.png)
>
> ![Open VPN Connection Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Open-VPN-Connection-Setting.png)
> - 在這邊輸入帳號密碼，也就是下方 Router 設定下方的使用者資料
> ![Open VPN Connection User Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Open-VPN-Connection-User-Setting.png)

那相關準備都完成後，我們就可以在外面連線回去宿舍網路啦！


## Reference

- [What is VPN? How It Works, Types of VPN - kaspersky](https://www.kaspersky.com/resource-center/definitions/what-is-a-vpn)