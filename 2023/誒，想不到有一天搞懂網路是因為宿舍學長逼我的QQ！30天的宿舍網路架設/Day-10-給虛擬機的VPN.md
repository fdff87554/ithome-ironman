# Day-10 - 如何連線虛擬機？RouterOS 的 VPN 設定

昨天我們總算把所有網路問題都搞定啦！從今天開始我們的 VM 就不會摸回到我的其他實體設備了！（這邊可以考考大家為什麼 `50` 網段可以摸到 `52` 但 `52` 摸不回去）。那我們就剩下最後一個終極任務了！也就是準備專屬於 VM 的 VPN！

身為一個悶騷人士，當我今天準備好我的 PVE 小 Server 的時候，就是要跟我的好友分享了對吧！狠狠的給他炫耀一波！友人 A：「啊你有 Linux 的環境可以開？這不是 VM 就可以做到了嗎？」友人 B：「PVE 歐？還要連回宿舍感覺好麻煩，你也不能借有我什麼用？」...，說好的稱讚呢？心中信仰逐漸崩塌的我開始思考為何我無法取得其他人的認同，~~用 Mac 錯了嗎？我就沒有 x86 的 VM 可以開啊...~~。逐漸扭曲和崩塌的我正在思考著為何我花了那麼多心力架設 PVE 時...。「你的 PVE 可以借我來寫實驗室專案嗎？Windows 的 Docker 好像會有特殊的問題，但我的電腦沒空間開 VM 了...」，萬念俱灰的我在聽到這句話的那一刻，不假思索地就回了「當然沒問題！」嗎？我究竟要怎麼讓我的 VM 被連線呢？

## RouterOS 的 WireGuard VPN

跟 ASUS Router 一樣，今天當我們要訪問我們的子網域時，相比直接把環境暴露到外網這個做法，使用 VPN 是一個很好的選擇。RouterOS 也跟 ASUS 一樣，提供了以 RouterOS 為基底的 VPN 服務，但他不是前幾天提到的 OpenVPN，而是 WireGuard VPN。

### 什麼是 WireGuard VPN？

WireGuard VPN 跟 OpenVPN 一樣，是一種 VPN 程式與協定，其不只開源，更標榜著輕量、快速、安全，在後續新的 ASUS Router 設備也慢慢支援此應用。那 WireGuard VPN 有許多屬於它的特色，但我們這邊就先跳過不過多贅述。

今天我們的目標很簡單，就是成功為我們的 RouterOS 開啟 WireGuard VPN Server 並使用我們的電腦在外網連接到 `192.168.52` 網段。讓我們開始看看怎麼設定吧！

### RouterOS 的 WireGuard VPN Server 設定

> ![RouterOS WireGuard Setting Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-WebFig-WireGuard-Setting-Page.png)

整體邏輯跟前設定 ASUS Router 的 Open VPN 近似，一樣先找到 WireGuard VPN 的位置，點開之後會看到 WireGuard 的兩個主要設定選項，分別是接口 Interface 的設定跟連線設備 Peer 的設定。

#### VPN Interface Setting

在第一個頁面的設定選項中，主要是在設定 VPN Interface 的基本資訊跟設定，我們點擊 Add New 後可以看到所要設定的資訊不多，主要有

1. Name：通道名稱，依照自己的命名習慣給予可辨識名稱即可
2. MTU：Maximum Transmission Unit，為 Frame / Packet 的最大大小，當數值越大時，同樣大小的資料就可以用比較少的封包數來傳輸，**但是如果網路通道不允許這麼大的 Packet 傳輸時，容易掉包造成傳輸上的問題，因此並非一味地調大**
3. Listen Port：就是 VPN Interface 在建立時所使用的 Port，預設為 `13231`，可以依照需求調整，這邊維持 Default 即可。
4. Private Key：VPN 連線加密所使用的私鑰，如果已經有私鑰要重新建立通道可以填寫，如果是全新的通道留空即可，當 Apply 時會自動生成。

所有設定完之後如下圖，我們就建立好屬於我們的 VPN 基本連線資訊了。

> ![WireGuard VPN Interface Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/WireGuard-VPN-Interface-Setting.png)

#### VPN Peer Setting

再討論 Peer Setting 之前我們來聊一下一個關於 VPN Server 的概念，還記得前幾天我們有討論過 VPN 的概念其實就是將我的數據從我使用的設備傳送到 VPN Server 之後，在用 VPN Server 向外傳送或者就不像外傳送了對吧？

那這樣的概念其實也代表了一個意思，也就是為了讓我們的設備能藉由這個 Server 像外傳輸，我們其實也等於在這個環境中擔任一個設備的角色，因此想當然我們也會在這個環境中有一個屬於我們的 IP 位置。

為什麼會需要特別提到這個，讓我們看看在 Peers 中的設定，當按下 Add New 之後可以看到裡面有幾個基本設定

1. Interface：接口，也就是我們剛剛設定的基本連線設定，這個數值等等會是 VPN Client 端找到 VPN 服務的基本設定。
2. Public Key：公鑰，這邊要填寫 VPN Client 端要跟 VPN Server 連線時驗證身份的公鑰，因此這邊 Peer 的設定會配合 Client 端的設定一起設定。
3. Endpoint：端點，這邊基本上可以留空，除非你的設備對此的連線 IP 是非常故定的，我們可以指定設備的連線 IP 來做保護，如果是如筆電等隨時會有位置變更的設備，就留空即可。
4. Endpoint Port：同等上述，如果相關設備有固定的 Port 也可以設定，一般來說留空。
5. Allow Address： 這指定了哪些 IP 地址和子網可以從該 peer 通過 VPN 進行通信。這是一種安全措施，確保只有預期的流量可以通過。那一般來說為了限制訪問，我們會只開通這個 VPN Client 的 Address，等等會示範。
6. Preshared Key： 這是一個額外的共享密鑰，用於增強與特定 peer 的通信的安全性。它是一個額外的加密層。
7. Persistent Keepalive：這是一個定時發送的訊息，確保連接保持活躍，特別是在某些 NAT 環境中。例如，如果設定為 25 秒，則每 25 秒會發送一次訊息以保持連接。

因此如下圖所示，我們示範的設定如下，可以看到我們 Interface 就是選擇之前設定的，而 Allowed Address 則是給 `10.0.0.2/32` 也就是 `10.0.0.2` 這個 IP，其他都不允許使用這個 Peer，那 Public Key 要怎麼給？讓我們直接看 WireGuard Client。

> ![WireGuard VPN Peer Setting Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/WireGuard-VPN-Peer-Setting-Page.png)

當我們在 VPN Client 端點選 Add Empty Tunnel 的時候，會發現他預設會幫你準備一個 PrivateKey 跟生成這個 Public Key，上面的 Public Key 就是我們要貼到 Peer 那個 Public Key 欄位的，所以讓我們貼回去。

> ![WireGuard VPN Add Empty Tunnel](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/WireGuard-VPN-Add-Empty-Tunnel.png)
> 
> ![WireGuard VPN Peer Setting Page with PublicKey](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/WireGuard-VPN-Peer-Setting-Page-with-PublicKey.png)

那在 VPN Peer 建立完之後，我們快速確認 VPN Client 要怎麼設定，我們可以看到基本上都已將幫我們帶入 Key 了，我們剩下的還需要如以下，

1. Interface 的部分跟 VPN Server 的設定的街口很像，你要提供一個 VPN Client 跟 VPN Server 的建立連線接口，那這邊當然也要有基本的資訊，包含我們之前提過的，你在這個網路位址身份也就是 Address，環境的 DNS 跟 MTU，有沒有發現跟 VPN Server 很像。
2. 那第二個部分就是我們的 Peer 了啦，跟驗證身份當然缺少不了 Public Key 了，這邊 Server 的 Public Key 就是前面 Server Interface 產生的 Key，然後 Endpoint 就是我們的目標 VPN 的位置啦！Port 就是剛剛 VPN 的 Port，我們剛剛設定 `13231` 就照舊啦。AllowedIPs 是一個很迷人的參數，這邊說明有哪些流量會流向 VPN Server，這邊也就跟剛剛上面 VPN Server 的慨念一樣，因此當我們設定 `0.0.0.0/0` 時就代表所有網路流量都會走 VPN 通道像 VPN Server 過去，但如果假如我們今天只要 Subnet 的流量過去可以怎麼設定？`AllowedIPs = 10.0.0.0/24, 192.168.52.0/24` 其實就可以了！這代表我們 VPN 連線的相關流量跟 52 子網段的流量會通往 VPN，但剩下的流量都不會。最後一個 PersistentKeepalive 則是多久進行連線狀況檢查。

```conf
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY_HERE
Address = 10.0.0.2/32
DNS = 192.168.52.254
MTU = 1420

[Peer]
PublicKey = SERVER_PUBLIC_KEY_HERE
Endpoint = SERVER_IP_OR_DOMAIN:13231
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

所以今天連線可以怎麼設定，就是如下設定啦！

```conf
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY_HERE
Address = 10.0.0.2/32
DNS = 192.168.52.254
MTU = 1420

[Peer]
PublicKey = CwEnQYfAJztTKl9jwjWszt05YjJNb0CgDdDTDXTbb14=
Endpoint = 140.114.234.160:13231
AllowedIPs = 10.0.0.0/24, 192.168.52.0/24
PersistentKeepalive = 25
```



恭喜我們相關設定都設定完啦！可以連...等等，還差點什麼？我們似乎還沒有把 WireGuard VPN 的 IP 們放置到內網中，因此我們回到 IP 的部分把 Interface 相關的設定補齊！

![WireGuard VPN IPs](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/WireGuard-VPN-IPs.png)

我們似乎也還沒告訴網路流量可以怎麼進入 VPN！回到 ASUS Router 我們來把 VPN 連線的最後一部開通！

> ![ASUS VPN Port Forward](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/ASUS-VPN-Port-Forward.png)

成功啦！

> ![VPN Connect to 52 in Client](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/VPN-Connect-to-52-in-Client.png)
> 
> ![VPN Connect to 52 in Browser](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/VPN-Connect-to-52-in-Browser.png)