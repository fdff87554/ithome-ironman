# Day-09 - 我的網路不見了！RouterOS 的設定小插曲

當我滿心期待的按下 `Apply Configuration` 之後，我開心的瀏覽器中輸入了新的 IP `192.168.50.250`，卻發現我失去了與網路的連線 QQ，原本以為是設定檔案沒有正確被執行，但回去找尋 `192.168.50.140` 時卻發現也連不回去 RouterOS 了！到底是什麼回事呢？

## Ethernet Quick Set 的小 Bug

在失去網路連線的那一刻我是真的慌了，立刻打開了 PVE 檢查我們了 RouterOS 的 IP 設定，還記得一開始我們查詢 IP 的時候是長怎樣嗎？沒錯，就是下圖，由 DHCP 發了一個 Dynamic IP 給我的 RouterOS。

> ![RouterOS IP from DHCP](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-IP-from-DHCP.png)

但是當我 Apply Configuration 之後，我的 IP 竟然並沒有跟我預期的一樣出現！

> ![RouterOS IP after Apply Config](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-IP-after-Apply-Config.png)

從畫面標注位置可以發現，我們關於第二網卡的 LAN 設定是有正確被設定的，也就是他已經有一個 LAN 的 IP `192.168.52.254`了，但我跟 ASUS Router 連線的 WAN 竟然離奇地消失了！

由於我們昨天針對 `192.168.50` 跟 `192.168.52` 網段的 Routes 設定尚未完全，且目前 RouterOS 在 ASUS Router 所管理的 `50` 網段中算是不存在的，因此我們基本上已經無法使用手上已有的設備與其互動，訪問不到 Web GUI，所以為了延續我們的操作跟說明，我們先手動添加 IP 給 RouterOS。

在指令列中輸入 `/ip/address add address=192.168.50.250/24 interface=ether1` 來將第一個網卡（ether1）賦予 `192.168.50.250` 的 IP，來重新給予 WAN IP 並確保可以藉由 `192.168.50.250` 訪問 Web GUI。

> ![RouterOS Add IP from Console](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-Add-IP-from-Console.png)

重新連線回到 IP `192.168.50.250` 之後，讓我們來看看問題到底出在哪裡了？回到 Quick Set 頁面可以發現所有相關設定並沒有跳走，那位了 Debug 我們試試看把 Internet 的 IP Address 的 `50.250` 的 IP 改成另外一個沒有用過的 `50.249`，並且點選 Apply 之後回到 PVE 的 Console 中再檢查一次，可以發現 `/ip/address print` 出來的 IP 竟然完全沒有變化！也就是代表這個頁面針對設定 User IP Address 這件事情的操作是有問題的，那到底怎辦？我總不能所有設定都還要回來一個指令一個指令輸入啊？要不然 GUI 還有什麼存在的意義 QQ。

> ![RouterOS Change IP from GUI Quick Set](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-Change-IP-from-GUI-Quick-Set.png)

## 從 WebFig 設定 RouterOS

雖然我們剛剛學過了使用指令的方式為我們添加需要的網路連線，但對於像我這種小白來說，這種特殊 OS 的操作也會是另外的學習成本，因此還是希望可以靠 GUI 調整，讓我們看看如果要調整要怎麼設定吧！

### Interface 網路介面卡設定

我們先忘掉之前所有用 Quick Set 的設定，把環境恢復一下，也就是只有 DHCP 分配 IP 的狀態。當我們點擊進入 WebFig 中時，會先到 Interface 的管理頁面，那這部分就是關於網卡相關的設定了，可以看到我們這邊有兩個網卡，分別是跟 ASUS 連線的 ether1 跟前天創建的第二張虛擬網卡 ether2。

> ![RouterOS WebFig Interface Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-WebFig-Interface-Page.png)

那我們點選 Interface List 後可以發現，RouterOS 已經預設幫我們把 ASUS 提供的 ether1 預設為 WAN 了，這是很合理的設定，原因是因為對外實體 IP 是由 ASUS Router 的 ether1 網卡送過來的，但可以發現 LAN 這邊他並沒有設定，所以讓我們先手動設定一下，給 ether2。

> ![RouterOS WebFig Interface InterfaceList Setting - 1](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-WebFig-Interface-InterfaceList-Setting_1.png)
>
> ![RouterOS WebFig Interface InterfaceList Setting - 2](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-WebFig-Interface-InterfaceList-Setting_2.png)
>
> ![RouterOS WebFig Interface InterfaceList Setting - 3](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-WebFig-Interface-InterfaceList-Setting_3.png)

### IP Address 相關設定

那現在我們要把 IP Address 都設定好，一樣是要把 DHCP 的動態 IP 改成靜態 IP，另外就是添加 `192.168.52` 網段的子網域設定。先讓我們從 IP > Address 找到相關設定頁面。操作如下圖

> ![RouterOS WebFig IP Address Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-WebFig-IP-Address-Setting.png)

這個畫面可以看到由 DHCP 所核發的 `50.140` IP，我們先用左上角的 `Add New` 添加我們要設定的 Static IP 設定，看到畫面中有 `Address` / `Network` / `Interface` 等設定，我們就依照我們的設備狀況做設定

1. Address: `192.168.50.250/24`
2. Network: `192.168.50.0`
3. Interface: `ether1`

> ![RouterOS WebFig IP Address ADD NEW](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-WebFig-IP-Address-ADD-NEW.png)
>
> ![RouterOS WebFig IP Address Static IP Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-WebFig-IP-Address-Static-IP-Setting.png)

這個時候讓我們先看看 Console 那邊是否有 IP 狀況被設定了，並檢查是否可以用 `192.168.50.250` 連線到 GUI，都可以之後我們就移除原本的 DHCP IP。

- 用瀏覽器檢查 IP 狀況
  > ![Check RouterOS IP](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Check-RouterOS-IP.png)
- 用 PVE Console 檢查設定檔案
  > ![Check PVE Console IP](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Check-PVE-Console-IP.png)
- 移除原本的 DHCP IP
  > ![Remove RouterOS Dynamic IP](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Remove-RouterOS-Dynamic-IP.png)

順便把 `52.254` 的 IP 設定給設定好，一樣的流程做添加即可。

> ![Add RouterOS 52 IP](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Add-RouterOS-52-IP.png)
>
> ![RouterOS WebFig IP Address Static IP & Subnet IP](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-WebFig-IP-Address-Static-IP-&-Subnet-IP.png)

### 子網域 DHCP Server 設定

我們希望我們可以由 RouterOS 分發給 52 網段的 VM 的 IP，並且當今天沒有要求 Static IP 時，請求 DHCP 的範圍為 `192.168.52.101` ~ `192.168.52.199`，我們先看到 DHCP Server 的設定，可以看到有 `Name` / `Interface` 等等設定可以調整。那我們 Mapping 回 Quick Setting 的部分，還記得 DHCP Server 打開之後，就只有多設定一個 DHCP Server Range 的部分，那這個選項是 Mapping 回這邊的 Address Pool 這個部分，那可以看到這邊有一個 dhcp 的 Pool 了（你也可能沒有），但為什麼他是一個 Pool 而不是跟 Quick Setting 一樣讓我們輸入範圍呢？我又要在哪裡調整呢？

> ![RouterOS WebFig IP DHCP Server Adding](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-WebFig-IP-DHCP-Server-Adding.png)

在 IP > Pool 的部分，可以設定各種 IP Pool，你可以設定一個特定的 IP 池來維護一種特定的狀況，也可以為 DHCP 設定一個分配範圍，那這邊會有這個名叫 `dhcp` 的 Pool 就是因為之前我們在設定 Quick Set 的時候有劃分一個範圍出來，因此它的作用範圍就做動到這裡生成一個 Pool 了。那如果你要自己設定也沒問題，相關設定畫面如下。

> ![RouterOS WebFig IP Pool Main Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-WebFig-IP-Pool-Main-Page.png)
>
> ![RouterOS WebFig IP Pool Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-WebFig-IP-Pool-Setting.png)

回到 DHCP Server 的設定，我們就會在剛剛的設定頁面基本上把 Name / Interface / Address Pool 設定好其他維持 Default 後，關於 RouterOS 幫我核發 DHCP 的設定就已經完成了。

> ![RouterOS WebFig IP DHCP Server Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-WebFig-IP-DHCP-Server-Setting.png)

## 從 `50` 連接到 `52`

最後的最後，RouterOS 的基本設定都已經完成之後，我們最後的任務就是當我們今天用電腦查詢 `52` 網段時，可以正確地走到。什麼叫正確地走到？還記得最前面幾天在討論 WAN / LAN 跟 Subnet 概念的時候，有說到當今天我們在把子網域遮罩蓋下去的時候，就決定了這個網域的大小，跟這個網域內可以查詢到的設備狀況。

先回到我們 ASUS Router 關於 LAN 的設定的部分，我們的設定如下，

1. IP: `192.158.50.254`
2. Subnet Mask: `255.255.255.0`

也就是說，我今天 ASUS Router 所維護的子網域就是 `192.168.50`，也因此，當我今天向他詢問 `192.168.52.xxx` 的設備時，這台 ASUS Router 是不知道應該去哪裡找到這個設備的。

那這件事情該怎麼解決？其實有兩種方法，但說明之前我們先重新劃分一下設備之間的關係。下方的圖示為現在我的網路設備的示意圖。

> ![Network Topology Graph with RouterOS](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Network-Topology-Graph-with-RouterOS.png)
>

### 設定 Route

從上述看到的網路結構其實就會有一個很明顯的感覺，也就是當我今天要訪問 `52` 的環境時，好像應該經過 RouterOS 的 `192.168.50.250` **走** 過去，這也就是前幾天在說明跟介紹 Router 設定中的 Route 功能，我告訴了我的網路環境今天 `52` 這個子網域可以藉由 `192.168.50.250` 這個接口過去。也因此這個解決方案一會產生下面的設定。

在 ASUS Router 中的 LAN > Route 新增如下圖

> ![ASUS Router LAN ADD Route](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-Router-LAN-ADD-Route.png)

因此當我們要訪問任何 `192.168.52.xxx` 的 IP 時，會發現他會是走 `192.168.50.250` 這個位置過去的，並且建立連線。

### 擴大 ASUS Router 的管理範圍

我們看到 ASUS Router 原本跟我們維護的子網域為 `192.168.50` 做維護的，但其實似乎也可以擴大維護範圍，因此我們也可以直接把 ASUS LAN 的 Subnet Mask 改成 `255.255.0.0`，那這樣代表所有在 `192.168` 這個 Subnet 底下的設備這台 Router 都可以查找到，也不是不行，但這邊我們就不過多贅述這種直接影響道子網域切割範圍的做法了。

