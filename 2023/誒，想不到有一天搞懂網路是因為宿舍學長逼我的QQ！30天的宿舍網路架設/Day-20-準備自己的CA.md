# Day-20 - 準備自己的 HTTPS 網站 - 自建 CA

昨天我們已經成功的利用 `DNS` 的挑戰方式來釋放我們的 Server 的 `80` Port，也學到了除了利用 PVE 的 ACME 以外，可以如何利用 Certbot 建立自己的 ACME Client，但到現在我們都仍然是跟 Let's Encrypt 這個 CA 獲取簽證，但總是當伸手牌也不太好，所以就讓我們維護自己的 CA 吧！

## 為什麼會需要自建 CA

在我們開始之前，先來了解一下為什麼我們會想要自建 CA。當然，使用像 Let's Encrypt 這樣的公共 CA 是非常方便的，但在某些情境下，自建 CA 可能更為合適：

1. 私有網絡：在內部網絡中，可能有些服務或應用程序不需要公開的 SSL 證書，這時自建 CA 就很有用。
2. 測試和開發：在開發和測試階段，我們可能需要大量的 SSL 證書，自建 CA 可以快速和容易地發放這些證書。
3. 更多的控制：自建 CA 可以讓我們有更多的控制權，例如證書的有效期、擴展屬性等。

那就讓我們看看要怎麼準備一個 CA 所需要的環境吧！

> - 其實第二點關於測試與開發的部分，目前第三方 CA 如 Let's Encrypt 也是有提供測試用憑證的獲取，這類型的憑證不會受到 Prod 的限制，也可以大量獲取跟申請，如果還是不希望建立自己的 CA 也是個不錯的選擇。
> - 要獲得這類型的憑證也非常簡單，只需要調整一下 command，如 `sudo certbot certonly --staging` 就可以了，這樣獲取的憑證就是從 Let's Encrypt 的 staging 環境獲取證書，而不是產品環境。

### 建立 CA

首先要建立個 CA 一樣要有個地方放他，所以請準備一個舒服的 OS 並準備一台 VM 吧！那這邊我會使用 Debian OS 作為我們的 CA Server，所以就讓我們來建立一台 VM 吧！

```bash
# Update Packages
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y

# Prepare Need Tools. Openssl
sudo apt install vim curl openssl
```

那工具都準備好之後，我們就準備開始準備需要的內容吧！

首先要維護一個 CA 為了能派發憑證，本身也必須有一個對照驗證的私鑰，因此我們需要先準備基本需要的 Keys。

先準備一個位置來存放憑證，

```bash
mkdir ~/myCA
cd ~/myCA
```

接下來建立私鑰跟根證書

```bash
# Generate Private Key
openssl genrsa -out rootCA.key 4096
# Generate Root Certificate
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem
```

這時，系統會詢問一些問題，例如國家、組織名稱等，這些資訊將被包含在證書中。

> ![Generate Root Certificate](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Generate-Root-Certificate.png)

那接下來我們就可以開始準備我們的 Server 要用的憑證了！

首先幫 Server 產生一個私鑰跟證書簽名請求（CSR）：

```bash
# Generate Private Key
openssl genrsa -out server.key 4096
# Generate CSR
openssl req -new -key server.key -out server.csr
```

這邊一樣會詢問一些問題，這邊可以依照自己的需求填寫，但最重要的是 Common Name，這邊要填寫你的 Server 的 Domain Name。

> ![Generate Server CSR](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Generate-Server-CSR.png)

接下來就是利用 CA 的私鑰來簽署 CSR 了。

```bash
# Sign CSR
openssl x509 -req -in server.csr -CA ~/myCA/rootCA.pem -CAkey ~/myCA/rootCA.key -CAcreateserial -out server.crt -days 500 -sha256
```

那到這邊就是一個完整的 CA 建立跟簽署給 Server 的憑證的流程了，但這邊有幾個部分要注意，

1. 接下來會需要把這個 `server.crt` 跟 `server.key` 遷移到要使用的 Server 上，而非放置在 CA Server 上。
2. 由於我們自建的 CA 並不會出現在一般的瀏覽器的信任清單中，所以在瀏覽器上會出現不安全的提示，這時可以將 `rootCA.pem` 匯入瀏覽器的信任清單中，這樣就可以避免這個問題了。

那現在的發派憑證的方式還有些原始，因此我們將嘗試把剛剛的 CA Server 在添加 ACME Server 的功能。

### 添加 ACME Server

我們今天選擇 `Boulder` 這個 ACME Server 工具，那很開心他們可以運作在 Docker 上，因此我們就依照下方步驟來建立一個 ACME Server 吧！

1. 幫環境準備 Docker

```bash
# Get install script from docker and run it
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# Create docker group and add current user into it
# sudo groupadd docker # Not need if install by docker script
sudo usermod -aG docker $USER
newgrp docker

# Test docker install
docker run hello-world

# Remove install script
rm get-docker.sh
```

2. 下載 Boulder，並且跑起來

```bash
# Install need tools
sudo apt install git -y

# Clone Boulder
git clone https://github.com/letsencrypt/boulder.git
cd boulder

# Run Boulder
docker compose up
```

那這樣我們的 ACME Server 就算是完成了，但這邊有幾個地方要注意，

1. 我們沒有仔細調整 docker-compose.yml 中的設定值，因此目前 boulder 運行的都是準備在他們所提供的 `/test` 目錄中的設定值。
2. `boulder` 其實有個輕量版 `pebble`，但其並沒有支援 Production Use，但如果不想要像使用 Boulder 這麼大的系統，可以考慮使用 Pebble。

接下來就利用昨天的 ACME Client 來測試一下吧！

```bash
sudo certbot certonly --server http://<YOUR-BOULDER-SERVER-IP>:4000/directory --manual --preferred-challenges http --apache
```

就可以成功利用 `http` 挑戰的方式跟我們的 ACME Server 溝通了！

## 最後的第三種驗證法，`TLS` 挑戰

TLS 身為一個設計用於保護網際網路通訊的加密和驗證通訊協定。是會依靠 TLS 交握來啟動整個 TLS 通訊跟結束的。而 TLS 的 ACME 驗證就是藉由這個交握的啟動過程做驗證挑戰。

但是 `TLS` 挑戰有兩個版本，第一個版本 `TLS-SNI-01` 因為安全性問題而被廢棄，目前是使用 `TLS-ALPN-01` 作為獨立標準運作。運作方法相同於 `TLS-SNI-01`，都是利用 TLS 通訊的 `443` Port 做運作（也就是一般 HTTPS 的 Port），但是 `TLS-ALPN-01` 是利用 `ALPN` 這個 TLS 的擴展功能來運作的。

`TLS` 挑戰有一個非常明顯的優點，不佔用 `80` Port 且完全直接於 TLS 上完成，且無需如 DNS 挑戰需要繁複的設定跟 DNS 服務的支持，但當然也會有缺點，現在最大的缺點就是尚未完全被支援（說明文件時間為 2023.02.13），如 Apache / Nginx / Cerbot 等等短期內都不會支援此驗證法，也正是因為如此，我們把這個挑戰方式放到最後面來聊聊。

## 總結

憑證的出現為網路世界的資料交換多了一層保護，我們不用再因為明文的資訊傳輸而害怕帳號密碼外洩、隱私資訊裸露，且由於大家的努力，現在取得憑證已不再是那麼困難的事情，而且也不需要花費太多的金錢，甚至是完全免費的，因此我們可以更加的安心的為自己的服務上一個新的保護。
