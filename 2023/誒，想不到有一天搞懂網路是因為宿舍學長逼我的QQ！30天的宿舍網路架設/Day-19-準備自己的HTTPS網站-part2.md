# Day-19 - 準備自己的 HTTPS 網站 - DNS Challenge / 自建 ACME Client

昨天我們成功利用 ACME 跟我們的免費好朋友 Let's Encrypt 做到自動取得證書的完美結合了，但還記得昨天有說到的幾個部分嗎？

1. 簽署證書不只有其他 CA，我們也可以自己建立 CA 來簽署證書
2. 目前使用的 Default HTTP-01 的 ACME 挑戰方式，可是非常非常可惜的，因為 `80` Port 就這樣被佔用了，雖然說當今天 Host 的服務希望有 HTTPS 的時候會開始轉移 Port 到 `443` Port，但這並不代表隨意犧牲一個 `80` Port 就是一個可以被接受的事情對吧？而且假如我們今天真的沒辦法空出 `80` Port 讓 ACME 進行挑戰的話，那難道就沒有別的方法可以做到了嗎？
3. 最後就是除了像 PVE 這種已經有預設 ACME Client 的服務外，我們是否有辦法幫其他自己架設的服務也準備一個 ACME Client 呢？

那今天我們就來看看這三個問題吧！

## ACME Client 的挑戰方式

ACME 真的很方便，可能是我沒有很仔細關注與學習，之前第一次接觸憑證相關知識時應該是沒有這麽方便簽署憑證的東西了，因此 ACME 能夠將憑證獲取的方式全部自動化確實真的很方便，但我們是否可以在這種方便中在更完善一些呢？不要讓資源有任何一絲的浪費吧！

昨天提到的 `HTTP-01` 挑戰的邏輯其實很簡單，可以想像成用原本的 HTTP 來進化成 HTTPS，那其實除了 `HTTP-01` 的挑戰方式外，還有另外一種常見的挑戰方式就是 `DNS-01`，那在昨天的[官方文件](https://letsencrypt.org/zh-tw/docs/challenge-types/)中有說明到，概念上今天 ACME 會藉由在 Let’s Encrypt 給予 ACME 客戶端 token，讓 Client 拿 token 與帳號金鑰進行運算，並產生一段文字，請將這段文字利用 TXT 紀錄放在 DNS 的主機名稱 `\_acme-challenge.<YOUR_DOMAIN>` 底下。

那簡單來說就是利用我們可以調整 DNS 服務商上我們的 Domain 資訊來進行擁有 Domain 的「證明」的概念，讓 Let's Encrypt 這些 CA 可以直接利用 DNS 資訊了解我們是擁有這個 Domain 的，但這邊會有一個小問題，也就是現在所使用的 DNS 服務商會需要提供這個 API 介面來協助我們做到這件事情。

那我們今天就直接利用我目前正在用的 CloudFlare 做示範來帶大家準備自己的 DNS-01 ACME Client 吧！

### DNS-01 with CloudFlare

首先我們要先幫昨天設定的 HTTP 挑戰調整一下，先到 `Datacenter` > `ACME` 找到裡面的 `Challenge Plugins`，選擇新增，我們要開始新增關於 CloudFlare DNS 的驗證了。

> 當然這邊前提要有 CloudFlare 的帳號，如果沒有的話可以先去申請一個，而且 CloudFlare 也有免費方案可以使用，所以不用擔心會有任何費用的問題。

> ![PVE's ACME DNS Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE's-ACME-DNS-Setting.png)

那我們仔細看下面的設定的部分，可以看到有很多要設定的項目，分別是 `CF_Account_ID` / `CF_Email` / `CF_Key` / `CF_Token` / `CF_Zone_ID`，其中我們要完成驗證，只會需要 `CF_Account_ID` / `CF_Token` / `CF_Zone_ID`，而其中 `CF_Account_ID` / `CF_Zone_ID` 也可以二擇一填寫，但我們這邊會示範這些東西要從哪裡取得。

首先打開 CloudFlare 的網站，選擇 Domain > 找到 API 的位置，可以看到直接就會看到 `Zone ID` 跟 `Account ID` 這兩個資訊了，這也正是我們說要放置在 `CF_Account_ID` / `CF_Zone_ID` 的資訊。

> ![CloudFlare Dash Main Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/CloudFlare-Dash-Main-Page.png)
>
> ![CloudFlare Dash Domain Main Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/CloudFlare-Dash-Domain-Main-Page.png)
>
> ![CloudFlare Dash Domain API](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/CloudFlare-Dash-Domain-API.png)

那至於 `CF_Token` 的話，可以看到 API 下面有一個 `Get your API token` 的按鈕，按下去就會到我們取得 API Token 的頁面，這邊我們就要使用 `Create Token` 來產生接下來會用到的 Token 了。

> ![CloudFlare User API Tokens Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/CloudFlare-User-API-Tokens-Page.png)
>
> ![CloudFlare Create User Token Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/CloudFlare-Create-User-Token-Page.png)

那選擇最上面的 `Edit zone DNS` 的 Template，然後依照下方示範調整

> ![CloudFlare Zone DNS Token Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/CloudFlare-Zone-DNS-Token-Setting.png)

在 `Continue to summary` 之後，確認設定都沒有問題之後就可以按下 `Create Token` 後即可看到本次需要的 Token，把他複製下來並且貼到 `CF_Token` 的位置即可。

> ![CloudFlare Get Create Token](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/CloudFlare-Get-Create-Token.png)

到這邊 CloudFlare 的相關設定都已經設定完了，回到 PVE 的 > PVE Node > System > Certificates > ACME 中，之前新增的 HTTP 調整成 DNS，並且在 Plugin 中選擇剛剛新增的 CloudFlare DNS，然後按下 `Save` 即可。

> ![PVE Certificates ACME Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Certificates-ACME-Setting.png)

之後一樣點下 `Order Certifficates Now` 重新取得憑證，即完成本次的 DNS-01 的挑戰方式的設定啦！

那如果今天我們不是使用 PVE 這種已經有預裝好甚至融合好 ACME 系統的，那我們要怎麼做到呢？就讓我們來自己準備自己的 ACME Client 吧！

## 為服務準備 ACME Client

Certbot 是 Let's Encrypt 官方推薦的 ACME Client。它支持多種 web 伺服器和系統。你從 [Certbot 官網](https://certbot.eff.org/)可以找到你的系統的安裝指南。那我們就直接準備一個全新的 VM 來示範 Certbot 的安裝吧！

假設我們的情境是一台 Ubuntu Server 並且用 Apache 維護一個網站前端，那當你在 Certbot 做選擇的時候可以選擇 Apache / Ubuntu 20，然後就可以依照下方的指示進行安裝了。那我們一樣準備基本需要的安裝指令。

```bash
# Prepare Basic Packages
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y

# Install snapd
sudo apt install snapd -y
# - Run test
sudo snap install hello-world
hello-world

# Remove certbot-auto and any Certbot OS packages
sudo apt-get remove certbot # to mack sure when running certbot is on snapd

# Install Certbot
sudo snap install --classic certbot

# Prepare Certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

# Run Certbot
sudo certbot --apache
```

接下來會冒出基本的設定詢問，包含之前的 Domain / Email，相關設定跟 PVE 中的沒有任何差別，等完成後看到以下畫面就代表成功了！

> ![Certbot ACME Client Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Certbot-ACME-Client-Setting.png)

## 最後的最後

我們剩最後一個題目了，也就是如何自建 CA，這部分就讓我們明天來了結他吧！
