# Day-06 - 我多了一台小電腦 - PVE 的生存日記

宿舍網路現在基本上算是很完善了，心情激動之餘也想著應該沒有新東西需要設定了吧？結果故事往往就是在 Ending 之前出現了續集，那就是我原本買給我媽媽的準系統她不要了 QQ。

還記得那是個狂風暴雨的夜晚，我拖著疲憊的身軀回到了家中，就看到媽媽表情嚴肅的坐在沙發前等著我回家。「媽媽？不是說不用等我回來了嗎？這麼晚了你明天...」正詢問著為何還不休息的母親問題的我被粗暴地打斷了，「不是媽媽在說你，每次給我的電腦都會用一用就壞掉，你看，現在電腦又打不開了，媽媽這樣怎麼做事啊？」，聽著母親的抱怨，已經身心俱疲的我也不耐煩地回到「不要用不要用！我換一台給你，這台我帶走用...」。就這樣，當我返回宿舍時，身上多了一台準系統。（母上大人拿了一台電競筆電當桌機用，可喜可賀可喜可賀）

看著桌上多出來的小電腦，一瞬間也實在是不知道可以做什麼，已經有一台筆電跟一台 Windows 桌機的我，打遊戲也不會想用這個沒有辦法裝顯示卡的迷你電腦玩，工作的東西也都在筆電上沒什麼用小電腦的理由呀？這台電腦到底可以幹嘛呢？「裝成 PVE 啊，直接裝成 PVE 當 Server 用呀。」背後又飄來學長的聲音，似乎是看出了我的窘迫，他直接給了一個方案。「剛好我們的筆電現在是 arm 架構很多東西沒辦法跑，你就跑一台 PVE 直接用來做測試比較快吧。」聽著學長的說明，我打開了電腦開始下載 PVE...

## PVE 的生存之路

不免俗來介紹什麼是 PVE，PVE 的全名是 Proxmox Virtual Environment，是一個 Open-Source Virtualization Platform（開源虛擬化平台），是一個基於 Debian 所開發的伺服器虛擬化平台服務，你可以想像成他就是為了虛擬化而存在的 Operation System，跟常見的 VirtualBox / VMWare Player, Workstation 這種需要運作在 OS 的虛擬化應用來說，虛擬化的層級更低，因此相比一般在 Host OS 上另外在做虛擬化的系統來說，虛擬化的效能更好。

### 什麼是虛擬化層級和虛擬機？

一般來說，當我們完成電腦的零件組裝之後，為了更好的使用設備會需要靠 Operating System 也就是作業系統（例如 Windows / Mac OS 等等），而這時候，所有的硬體資源都會由這個 OS 做使用跟分配。這樣本身是沒有問題的，但如果今天我們只有一套硬體設備，卻又希望可以測試不同的環境或者運作多種服務呢？這時候硬體資源就不可以全部都被一個環境所取得，需要做其他狀況的分配，而這樣的概念就叫做虛擬化。

Hypervisor 虛擬化引擎應運而生，Hypervisor 可以想像成虛擬機的控制平台(或者引擎)，所有虛擬的作業系統都會安裝在 hypervisor 上面，而且由 hypervisor 去控制硬體（換句話說 Guest OS 並不會直接觸碰到硬體資源）

而 Hypervisor 會因為安裝的位置而分成兩的類別

- Type I: 又稱為 Bare-metal hypervisor ，大致上就是裸機直接安裝 hypervisor 而沒有 Host OS 的部分，那 VM 在這樣的狀況下接觸硬體資源的效率會比較高，因為只要透過 Hypervisor 就可以操作資源了
- Type II: 又稱為 Hosted Hypervisor ，跟 Type I 相反，必須先有 Host OS 之後再安裝 hypervisor ，通常都負載虛擬機軟體中，而 Hypervisor 調用資源的部分則會交由 Host OS 來協助處理。
  **Host OS 代管硬體分配最大的問題就是 Host OS 本身會扣留一部份硬體資源，且 Host OS 託管硬體分配也會造成硬體資源消耗較嚴重**

而我們要使用的 PVE 就屬於 Type I 的虛擬化。

### PVE 的安裝之路

那有鑒於 PVE 精美的全 Web 話管理介面、多種方便好用的功能，我們就開始將準備我們的 PVE 吧！

首先我們先準備一個 static IP 給 PVE，因為身為一個會有直接連線需求的 Server 服務，如果交由 DHCP 分配動態 IP，很有可能因為時限到期或者 Server 關機之後，原本的 IP 被 DHCP 所釋放出去，雖然這個機會不算太高，但為了設備服務的穩定我們就直接設定 static IP `192.168.50.252` 了。

那 IP 等東西都準備好了之後，讓我們開始準備安裝吧！

1. 下載 ISO 檔案，在他們的官方網站就可以直接取得 ISO 了，https://www.proxmox.com/en/downloads
   > ![PVE Official Website Download Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Official-Website-Download-Page.png)
   > 由於我已經安裝過了，接下來的說明跟截圖都會用 VM 示範，基本上完全相同，不用切換。
2. 開機之後會看到這樣的安裝畫面，那 `Graphical` 跟 `Console` 的差異就是有沒有漂漂亮亮的 GUI 畫面而已（這個白底），所以是電腦設備狀況選擇，基本上直接 Enter 即可
   > ![PVE ISO Main Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-ISO-Main-Page.png)
3. 關於使用者須知的部分我們直接下一步
   > ![PVE End User License Agreement](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-End-User-License-Agreement.png)
4. 接下來是選擇安裝位置，一般來說除非你有切分硬碟等需求，否則一樣就是選擇你想安裝的位置後下一步即可
   > ![PVE Target Harddisk](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Target-Harddisk.png)
5. 下一步是地點跟時區的選擇，我們這邊 Country 選擇 `Taiwan` 即可，Timezone 選擇 `Asia/Taipei`（理論上順利都會由系統自己檢測出來），鍵盤配置的部分就隨意選擇，但一般來說就是 `U.S English`
   > ![PVE Location and Timezone Selection](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Location-and-Timezone-Selection.png)
6. 由於 PVE 算是一個管理系統，因此當一些異常狀況發生時，是可以提供警示 Email 的通知服務的，因此在這個部分我們設定了我們的密碼跟通知用 Email
   > ![PVE Admin Password and Email](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Admin-Password-and-Email.png)
7. 我們上面有提到過，PVE 由於算是一個管理服務，針對網卡 / IP 等設定皆是預設為 Static 也就是需要自己設定的狀態，那如果你的 Router 本身有 DHCP 服務，他仍然會先跟 Router 取得一組 IP 設定，但請注意這邊請改成自己的設定。
   從 Network Config 中可以看到，我們這邊可以選擇
   - Interface 也就是網路介面卡，這個設定預設不用調整，視你的硬體設備有多少網路介面而定。
   - Hostname 則是你設備未來可能提供的 Domain Name，可以看到他有標註 FQDN（Fully Qualified Domain Name，完整網域名稱），如果沒有可以不理他，其主要功能在於識別設備所使用。
   - IP Address（CIDR）則是你的設備 IP 和其所可以使用的子網域，那前面有提到我們期望把設備設定在 `192.168.50.252`，其子網域歸屬於 `192.168.50` 因此 CIDR 表示法為 `192.168.50.252` / `24`。
   - Gateway 也就是我們的 Router 是 `192.168.50.254`
   - DNS Server 為 `192.168.51.53`（這個地方是因為我有在 PVE 裡面開 DNS Server，一開始我是先選擇預設跟 Gateway 相同，或者也可以先設定成 `1.1.1.1` 或者 `8.8.8.8`，這部分我們後期會調整。）
   > ![PVE Network Config](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Network-Config.png)
8. 上述設定都完成了就可以開始安裝啦，最後這邊會跳一個總結頁面讓你做確認，確認沒問題就下一步了！
9. 在完成安裝並且重啟之後，你會直接看到系統通知你用其他設備連線到其提供的網頁 GUI 使用畫面
   > ![PVE Server](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Server.png)
10. 依照說明輸入 IP 後，進入此畫面及代表安裝完成！
    > ![PVE GUI](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-GUI.png)

那到這邊我們就完成安裝啦，明天讓我們聊聊我到底希望怎麼調整跟設定這台設備吧！
