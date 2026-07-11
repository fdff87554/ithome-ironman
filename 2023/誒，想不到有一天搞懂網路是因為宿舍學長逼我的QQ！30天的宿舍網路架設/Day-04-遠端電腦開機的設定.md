# Day-04 - 誒，我可以遠端開我的電腦了！

明明只是要設定個 Router 來連上網路，怎麼感覺快去世了 QQ，不是說好學長會帶我飛嗎？沒事，今天我們就回歸一個小目標，當電腦關機時，能遠端開啟宿舍的桌電吧！

身為高級菸酒生，時常需要跟著學長出差想必也是很正常的事情對吧 QwQ，那這個時候如果需要用到宿舍的桌電，拿一些檔案或者資料也都是很常會發生的情境，但是，總不可能電腦就開著都不關機了吧？住宿舍跟住家裡最大的差異就是有室友。我的學長，身為睡眠品質追求的第一人，連我的“呼吸聲”都無法忍受，是不可能接受一台桌電在宿舍的怒吼的，那我有沒有辦法在不打擾學長的狀況下，實現遠端開關電腦自由呢！

## Wake up from LAN

好家在，有類似遠端開電腦需求的人並不是只有我一個，先不管其他人的理由為何，能有這樣的功能真的好棒棒。

要遠端開啟電腦這個目標會使用到的功能叫 `Wake-on-LAN`（中文叫網路喚醒），要實現這個功能會需要以下設備的配合

1. 主機板 / 網卡要支援這個功能（因此我們會需要調整到 Bios 設定）
2. 作業系統要支援有相關設定
3. Router 要支援此功能

為什麼會說需要這幾個東西互相配合呢，先讓我們介紹什麼事 Wake-on-LAN 機制。

### 邏輯上來說如何喚醒設備？

Wake-on-LAN (或者縮寫 WoL) 是一種特殊的技術，他的主要功能就如同其名字所述，讓正在 “休眠” 或者 “關機” 狀態電腦能透過區域網路的其他設備所發送的訊號做喚醒，讓這個設備做啟動跟喚醒。

而 Wake-on-LAN 這個技術的概念是，就算電腦關機之後，仍然持續為主機板還有網卡進行部分供電，來監聽網啊卡是否有收到來自電腦外部網路的封包，並進行封包的解讀，而當這個時候電腦解讀到一個 Magic Packet 的時候，就會針對這個 Packet 研判並且如果資訊正確就會啟動電腦。

由於我們之前提過，在網路的世界是依靠 IP 這個網路的地址做互動的，但由於電腦關閉的時候，router 也不會有辦法提供 IP 且你的設備對於網路世界來說也不存在，所以當然不會有 IP，那這個時候是沒辦法跟設備做互動的，感覺是不是陷入死回圈了 wwwww，究竟其他設備要怎麼跟這個關閉的設備做互動呀？

這就是為什麼 Wake-on-LAN 要叫做 Wake-on "LAN" 了，今天從其他設備送出去的封包並不是一般的封包，而是叫做“廣播封包”，這種封包並沒有指定的接收者，所有在這個 LAN 中的設備都會收到這個封包，那今天當目標設備的網卡收到這個 Magic Packet 之後，發現裡面的資料跟我的設備資料是正確匹配的，就會執行這個魔術封包中所需要做的事情，例如開機。

那一樣，我要怎麼確認這個設備說明的是“我”？其實除了 IP Address 這個唯一識別位置的地址以外，每一台設備還有一個獨立的 “裝置編號”，我們叫 `Mac Address`，我們可以想像這個 Mac Address 是網路世界用來識別一個裝置的唯一身分證，那 WoL 就是改用這個 Mac Address 用來指定開啟的設備的。

所以我們已經知道遠端喚醒的邏輯了，讓我們看看剛剛提到的設備的各種角色。

### 設備與 WoL 的關係

剛剛提到了，WoL 的邏輯就是目標設備一直監聽有沒有收到 Magic Packet，那總要有人發送這個 Magic Packet 對吧？那這個設備會是誰呢？其實並沒有固定，我們說過了，基本上只要能對外發送 Magic Packet 的不管是電腦 / 網通設備都是可以的，甚至現在例如 AnyDisk 之類的軟體也都有辦法做到類似的事情，但其實他們的概念都是向目標網域內發送 Magic Packet 來觸發正在監聽的設備解析。

那為什麼這邊我們直接說是 Router 呢？原因正是因為依照我的情境或者一般家庭的情境，你的電腦設備會直接在 Router 後面所管理的子網域中，那由 Router 來幫忙發送訊號當然再好不過了，也不用另外長駐開著其他設備了。

剩下就是目標設備要乖乖地調整完相關設定，接下來讓我們看看怎麼設定吧？

## 設定 WoL 功能

要成功做到 WoL 來遠端開啟我們可愛的電腦，會需要以下幾個步驟。

1. 開啟主機板的相關功能（也就是讓電腦持續為網卡通電並解析 Magic Packet）
2. 開啟 Windows 相關功能（如果是其他作業系統也可以上網查相關設定，這邊用 Windows 舉例）
3. 開啟 Router 相關功能

讓我們一個一個來。

### Bios 設定

硬體的設定往往還是要用到 Bios，那這部分也正是之前提到的，要先確認設備有沒有支援這個功能。我們直接拿 ASUS 主機板設定做舉例。（相關照片來源為 ASUS 官網教學 https://www.asus.com/support/FAQ/1045950/）

1. 電腦開機之後狂按 Del / F2 進入你的 BIOS 並選擇 “Advanced Mode（進階模式）”
   > ![BIOS Setting Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-ROG-BIOS-Settings.png)
2. 由於我們要讓電腦持續使用部分電力，因此選擇 “Advanced（進階）” 裡面的 “APM Configuration（電源管理設定）”
   > ![BIOS Advanced ARM Configuration](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-ROG-BIOS-Settings-ARM-Config.png)
3. 那由於主機板對於 WoL 這個功能的訊號觸發是叫做 “Power On By PCIE”，所以我們就開啟這個功能之後，電腦就可以由這個訊號做觸發了
   > ![BIOS Power On By PCIE Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-ROG-BIOS-Power_on_by_PCIE.png)

所以現在主機板已經準備好為我們服務了，來調整 Windows 吧～

### Windows 設定

為什麼要調整 Windows 相關設定有一個很重要的理由是，作業系統現在為了能夠提供更好的服務，其實會有所謂的電源策略這個東西，是為了加速我們開啟 Windows 的速度。現在的電腦關機我們會叫做“偽關機”，電腦會在關機之後，將電腦狀態維持在關機前的狀態，且某種意義上算是劫持電腦的電源，將狀態維持在 RAM 中，並持續通電，來做到開機時直接讀取 RAM 的資料完成開機。那也因為這個所謂的 Fast Boot 或者快速啟動的邏輯，電腦會沒辦法由 Bios 控制啟動邏輯，或者你的電源會被搶走，來維護開機速度，因此我們需要改變這個現況。當然還有一個原因是要允許網卡做到 WoL 這個功能。

所以這邊我們有兩個目標，調整網卡設定 & 電源策略。

1. 打開電腦之後，右鍵點擊可愛的 Windows 圖示，並且開啟 Device Manager（裝置管理員）
   > ![Windows Open Device Manager Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Windows-Device-Manager-Open.png)
2. 找到我們的 Network Adapters（網路介面卡），並且在裡面找到我們的實體網路卡，就是接著你的 Router 的那個洞洞的網路介面，一般來說由於網卡廠商都是 Intel 的，一般來說會是 `Intel(R) xxx` 開頭的，那不要選擇無線網路相關的，要選實體網路孔。
   > ![Windows Network Adapters Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Windows-Network-Adapters-Page.png)
3. 點擊進去之後，可以看到一個 Power Management（電源管理），裏面就會有一個選項叫 “Allow this device to wake the computer（允許這個裝置喚醒電腦）”，只要開啟這個選項就等於允許 WoL 這個功能了。那一般來說由於我們明確是要靠 “Magic Packet（魔術封包）”來做開機，因此我會建議也要開啟 “Only allow a magic packet to wake the computer（僅允許 Magic Packet 喚醒電腦）”。另外上面那個允許電腦關閉此裝置來節省電源也建議關閉，因為這樣才不會有電腦錯誤關閉服務的狀況發生。
   > ![Windows Settings to Allow device to wake the computer](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Windows-Wake_the_computer-Settings.png)

那我們已經允許電腦設備從網卡喚醒了，快來設定電源策略吧！

1. 打開電腦之後，右鍵點擊可愛的 Windows 圖示，並且開啟 Power Options（電源選項）
   > ![Windows Open Power Options Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Windows-Power-Options-Open.png)
2. 打開設定頁面之後，選擇 “Additional Power Settings（其他電源設定）”
   > ![Windows Addtional Power Settings](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Windows-Additions-Power-settings.png)
3. 我們要關閉 Windows 的快速開機設定，這個選項被藏在 “Choose what the power butters do（選擇按下電源按鈕時的行為）” 並關閉電腦快速啟動。
    > ![Windows Addtional Power Settings](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Windows-Fast-Startup.png)

最後最後，我們來 Router 開啟發送 Magic Packet 的功能就大功告成啦！

### Router 設定

在 ASUS Router 中，進階設定中有一個 Network Tools，裡面會有 Wake on LAN 的選項，在裏面就會有相關設定了。在裡面給設備的 Mac Address 就可以指定喚醒的設備啦～
> ![ASUS Router WoL Settings](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-Wake-on-LAN.png)

## References

-  https://www.asus.com/support/FAQ/1045950/
