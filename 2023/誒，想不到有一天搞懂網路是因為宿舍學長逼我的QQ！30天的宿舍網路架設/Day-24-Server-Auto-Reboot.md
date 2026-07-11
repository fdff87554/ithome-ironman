# Day-24 - 停電怎麼辦！Server 的自動復電重啟設定

在歷經了快樂的 23 天文章撰寫的時候，正當我開開心心地準備完成下一個主題時，赫然發現我的電腦竟然無法連線了！適逢國慶連假在老家渡假的我一時間冷汗直流，怎麼了？我把服務搞壞了嗎？我的 PVE 就這樣壞掉了嗎？我實驗室的實驗資料都還在裡面啊！我東西都還沒有測試完！我的環境啊啊啊啊！

這時我的手機跳出了一封 Email，「各位住在宿舍的同學夥伴你們好，很抱歉宿舍這邊發生了大跳電，目前已經聯繫學校並且已經在處理中了，台電人員已經回報正在檢測，請同學...」，原來是宿舍跳電而已呀？還好...好個毛線呀！我的服務都還沒架設好！我的東西都沒有關閉呀！更重要的事情是我要如何在電力恢復時讓我的 Server 自動重啟呢？

還記得我們 Day 4 的時候有聊過 Wake on LAN 嗎？這個功能利用 Router 作為遠端互動的節點，利用這個節點喚醒內網中的其他設備，這樣的話我們就可以在外面透過 Router 喚醒我們的 Server 沒錯，但現在我們有另外一個問題要思考，究竟能不能讓重啟服務這件事情是自動完成的呢？其實可以，那就是利用 BIOS 的設定，讓 Server 在電力恢復時自動重啟。

## Power on after power failure

今天我們先來聊聊為什麼我們不只需要 Wake on LAN，還需要所謂的復電自動重新啟動的功能呢？差別在以下幾點：

1. 今天設備是否能在恢復電力的第一時間完成啟動
2. 今天我們 PVE 的角色跟維護的服務
3. 如果今天沒有另外一個在恢復電力時會啟動的 Router 在前面可以做喚醒

為什麼會有上述幾點呢？

首先當我們今天的服務維護是需要盡可能在電力恢復時進行重啟時，想必我們就不可能自己發現電力恢復了，然後才去開 VPN 連線 Router，並且再透過 Wake on LAN 去喚醒 Server。那想必對於服務恢復的即時性來說，明顯不是最佳解，也不是我們想要的，因為我們要無時無刻關注相關資訊，確保電力或者各種其他狀況被排除，並且在第一時間完成剛剛所述，這樣真的不是很理想且合理的作法。

其次今天可以做到 WoL 本身就需要有一個服務可以在電力恢復時自動完成服務的啟動，那今天剛好我們就是已經有一台 ASUS Router 來幫我扮演復電啟動的角色了，因此才能做到，那當今天一個環境中沒有這樣的角色呢？我就剛好直接利用 PVE Server 直接作為軟路由時又該怎麼處理？這種時候還有辦法使用 WoL 嗎？想必也是沒有辦法的對吧？

最後則是今天 PVE 所扮演的角色其實重要性跟 ASUS Router 已經平等了，還記得我新的 VPN 服務都已經交由 PVE 上的 RouterOS 做維護運行了，那如果今天沒有 PVE 一起在第一時間恢復運作，其實我也是無法跟 ASUS Router 做連線互動了（如果我沒有保留 Router 的 VPN 連線）。

因此當今天 PVE 也已經是核心節點的狀況下，維護好 PVE 的運作狀態，讓服務在電力恢復時可以自動啟動，就是我們今天要做的事情了。

### 「Poweron after PWR-Fail」功能

現在的機器 BIOS 已經越來越細膩了，從之前說過的 WoL 功能的介紹應該可以發現，針對很多情境都已經有對應的功能可以設定，而今天的故事狀況正是有一個叫做「Poweron after PWR-Fail」的功能。

「Poweron after PWR-Fail」這個選項的意思是電源突然失效後（停電、直接切斷電源），當再次有電源供應時，系統應該做什麼行為的一個設定區域。其中一般來說提供了兩個主要的選項，分別是：

1. 「Power On」：當電源恢復時，不管之前是關機狀態或者開機狀態，都自動啟動系統。
2. 「Last State」：當電源恢復時，如果之前是關機狀態，則保持關機狀態；如果之前是開機狀態，則保持開機狀態。

那另外一個隱藏選項就是關閉此功能，因此不會有任何行為。

那上述兩者最大的差異在於，Power On 選項有時候會造成停電後復電的靈異電腦自動開機事件，那一般來說還是會建議額外加裝其他的電源保護設備，例如 UPS，這樣才能保護電腦不會因為電力不穩造成的損壞。「偉大的管理人員可能會反覆開啟電源閘，這時候電源供應器還是有可能直街被送走的 QQ」。

### 如何設定「Poweron after PWR-Fail」

其實這個功能的設定一定非常簡單，而且由於不需要其他設備的配合，因此只要進入 BIOS 設定就可以完成了。

我們一樣用 ASUS 主機板的 BIOS 做介紹說明，

1. 首先進入 BIOS 設定畫面，並且進入「Advanced Mode 進階模式」中的「Advanced 高級」選項。
   > ![ASUS BIOS Advanced Mode Advanced Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-BIOS-Advanced-Mode-Advanced-Page.png)
2. 找到「APM Configuration」這個選項，並且進入。
   > ![ASUS BIOS Advanced Page ARM Configuration Option](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-BIOS-Advanced-Page-ARM-Configuration-Option.png)
3. 「Restore AC Power Loss」選項就是這次我們上面提到的「Poweron after PWR-Fail」在 ASUS BIOS 中的名稱，我們可以選擇剛剛所提到的三種模式，「Power Off」、「Power On」跟「Last State」，這樣可以確保其達到我們希望的復電後行為。
   > ![Restore AC Power Loss Options](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Restore-AC-Power-Loss-Options.png)

> 所有圖片由 ASUS 官方網站範例提供：https://www.asus.com/tw/support/FAQ/1049855/

基本上完成上述設定後，就可以完成我們的機器自動重啟了，但現在我們的 PVE 已經重新啟動了，卻不代表我們的 VM 們也跟著重新啟動，因此現在讓我們再看看 PVE 怎麼設定 VM 在 PVE 啟動時就同步啟動吧～

## PVE VM 同步啟動

今天要幫 PVE 中的 VM 設定同步啟動其實非常簡單，

1. 首先，我們先選擇一個目標的 VM > 點選 Options，會看到關於 VM 的相關設定。
   > ![PVE VM Options Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-VM-Options-Page.png)
2. 找到 `Start at boot` 選項，選擇 Yes 即代表當 PVE 啟動時，這個 VM 也會同步啟動。 > ![VM Start at boot Options](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/VM-Start-at-boot-Options.png)

這樣設定就完成啦！是不是非常簡單呢？

## 小結論

其實針對生活中的場景與想像很多都已經被考慮在各種服務中了，因此了解如何開啟往往就已經足夠我們好好的守護我們的服務。今天這樣的復電啟動策略其實也有它的好處與壞處，好處當然是當電力恢復時，所以服務會在第一時間運作與恢復，但也相對的，當電力短時間持續有問題發生時，設備也會更加容易損毀，檔案也更危險，因此如果沒有一定的備援機制跟電力保護機制，這些功能的開啟其實也會需要額外的評估。
