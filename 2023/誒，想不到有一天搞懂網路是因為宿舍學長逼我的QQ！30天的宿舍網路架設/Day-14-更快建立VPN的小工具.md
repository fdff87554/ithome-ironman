# Day-14 - 更快建立 VPN 的小工具

到現在我們基本上網路、硬體等等有的沒有的東西都算是完成設定了，所以明天開始，我們就開始梳理前面提到的各種網路知識跟概念，甚至看看有沒有機會藉由這次機會看點安全相關的設定，但在這之前，我們來幫我們的環境拼圖的最後一塊補上，那就是能不能更簡單的建立 VPN 通道。

## VPN 的建立是不是太過繁瑣了？

還記得前幾天我們用了 MikroTik RouterOS 的 WireGuard VPN 來跟 `52` 網段的 VM 們做互動，然後也有一些其他設定，但簡單來說就是利用 WireGuard VPN 建立了跟宿舍網路互動的連線通道，但有沒有發現正個建立通道的流程有點過於繁瑣？而且跟之前使用 ASUS Router 的 OpenVPN 最大的差別是，我竟然需要跟 Client 端先取得 Public Key 然後才能再回到 RouterOS 的 Peer 設定建立連線通道，究竟整個流程就沒有辦法跟 OpenVPN 一樣直接取得一個設定檔在 Client 端匯入就好嗎？我們明明就有在 WireGuard VPN 的 Client 端看到匯入資料才對呀！

那其實可以，但會需要我們捨棄由 RouterOS 所維護的 WireGuard VPN Server，並且會有其他額為的網路設定，因此我們先從另外一個角度，也就是假設我們是一個雲端網路服務，然後我們希望建立屬於自己的 WireGuard VPN 的時候，可以怎麼建制。

## 建置 VPS 上的 WireGuard VPN Server + WireGuard Portal

首先準備好我們的 OS，這邊我使用 `Debian 12` 作為我的 OS，那我準備了一個基本環境設定 + SSH Server 設定的快速安裝腳本，大家可以參考一下直接完成基本所需要的工具安裝跟部署。

- `Debian-WireGuardServer-BaseSetup.sh`

  ```shell
  #!/bin/sh
  # Update & upgrade apt packages
  echo "Updating apt packages..."
  sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y

  # Install git, vim, curl, wget, tmux, ssh-server
  echo "Installing git, vim, curl, wget, tmux, ssh-server..."
  sudo apt install git vim curl wget tmux openssh-server -y

  # Setup ssh server
  echo "Setting up ssh server..."
  sudo systemctl start ssh
  sudo systemctl enable ssh
  # - Disable root login
  sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/g' /etc/ssh/sshd_config
  # - Disable password login
  sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config
  # - Enable public key login
  sudo sed -i 's/#PubkeyAuthentication no/PubkeyAuthentication yes/g' /etc/ssh/sshd_config
  # - Add public key into authorized_keys
  mkdir -p ~/.ssh
  touch ~/.ssh/authorized_keys
  # <paste your public key inside>
  sudo systemctl restart ssh

  # Setup docker
  echo "Setting up docker..."
  curl -fsSL https://get.docker.com -o get-docker.sh
  sh get-docker.sh
  sudo usermod -aG docker $USER
  newgrp docker
  rm get-docker.sh

  # Setup wireguard
  echo "Setting up wireguard..."
  # - Install wireguard
  sudo apt install wireguard -y
  # - Get WireGuard Portal
  git clone https://github.com/h44z/wg-portal.git
  ```

現在我們有 WireGuard 跟準備好 WireGuard Portal 了，我們先準備好我們的 WireGuard Server 吧。

### 設定 WireGuard Server

其實核心概念與 RouterOS 上的 WireGuard 設定近似，因此我們當然要先幫環境準備好基本需要的東西。

1. WireGuard PrivateKey
   ```bash
   wg genkey | sudo tee /etc/wireguard/private.key
   sudo chmod go=/etc/wireguard/private.key
   ```
   - `sudo chmod go=...` 命令會刪除 root 使用者以外的使用者和群組對該檔案的所有權限，以確保只有 root 使用者可以存取私鑰。
   - 這時候我們會收到一行 `base64` 編碼的私鑰輸出，這把鑰匙也會同步儲存在 `/etc/wireguard/private.key`
2. 用剛剛的私鑰產生公鑰
   ```bash
   sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
   ```
   這句指令分成三個部分，分別是
   1. `sudo cat /etc/wireguard/private.key`：這個指令讀取私鑰檔案並將其輸出到標準輸出中。
   2. `wg pubkey`：將剛剛輸出的 private.key 作為輸入並對其進行處理以產生公鑰。
   3. `sudo tee /etc/wireguard/public.key`：最後一個指令取得公鑰產生指令的輸出並將其重新導向到名為 `/etc/wireguard/public.key` 中。
   - 這時候我們會收到一行 `base64` 編碼的公鑰輸出。
3. 跟在 RouterOS 中一樣準備 Interface 相關設定，那由於我們剛剛有私鑰了，因此我們直接創建設定檔案到 `/etc/wireguard/wg0.conf` 中。
   ```conf
   # /etc/wireguard/wg0.conf
   [Interface]
   PrivateKey = base64_encoded_private_key_goes_here
   Address = 10.8.0.1/24
   ListenPort = 12321
   SaveConfig = true
   ```

第四步驟的設定是當我們希望藉由 VPN 做到外網連線才需要設定的，如果不需要可以跳過

4. 設定 IP Forwarding，編輯 `/etc/sysctl.conf`
   ```conf
   net.ipv4.ip_forward=1
   ```
   並且可以使用下面指令檢查環境 `sudo sysctl -p`，可以看到我們的變更狀況
   > ![Command sysctl -p check](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Command-sysctl--p-check.png)
5. 設定基本的流量規則，需要將我們的接口流量給放行，因此這邊我們依照
   1. 使用 `ip route list default` 指令確認 WireGuard 伺服器的公共網路介面，Output 可能會長得像這樣 `default via 203.0.113.1 dev eth0 proto static`，裡面的 `eth0` 就會是我們要記得的部分。
   2. 在剛剛的 `/etc/wireguard/wg0.conf` 檔案中新增以下設定在剛剛的 `SaveConfig = true` 指令後面。
      ```conf
      PostUp = iptables -t nat -I POSTROUTING -o <interface_of_your_os> -j MASQUERADE
      PreDown = iptables -t nat -D POSTROUTING -o <interface_of_your_os> -j MASQUERADE
      ```
      `PostUp`是當 WireGuard 伺服器啟動虛擬 VPN 隧道時，這些線路將會運作。
      `PreDown` 則是當 WireGuard 伺服器停止虛擬 VPN 隧道時，規則將會運作。
6. 啟動我們的 WireGuard Server，使用 `sudo systemctl start wg-quick@wg0.service` 即可開啟，如果我們希望 Server 啟動時直接啟動，則使用 `sudo systemctl enable wg-quick@wg0.service` 來添加啟動清單。
7. 使用 `sudo systemctl status wg-quick@wg0.service` 其可確認運作狀態。

### 準備 WireGuard Portal 的環境

我們準備好我們的 WireGuard Server 了，是時候把可愛的 GUI 掛上去了，要準備好 WireGuard Portal 其實非常簡單，我們這次會使用 Docker 來協助運作這個環境，因此我們有兩個目標檔案的撰寫跟調整，分別是

1. `compose.yml` 這邊準備了所有關於 Docker Container 的相關需求，直接上範例：

```yaml
services:
  # 創建一個名為 wg-portal 的容器
  wg-portal:
    # 指定容器的 image 來源，這邊使用 h44z/wg-portal:v1.0.18
    image: h44z/wg-portal:v1.0.18
    # 指定容器的名稱，這邊使用 wg-portal
    container_name: wg-portal
    # 設定重啟策略，這邊使用 unless-stopped，也就是當容器不是被手動停止時，就會自動重啟
    restart: unless-stopped
    # 設定容器的 log 紀錄設定
    logging:
      options:
        # 設定 log 紀錄的最大行數
        max-size: "10m"
        # 設定 log 紀錄的最大檔案數
        max-file: "3"
    # 設定容器的環境變數
    cap_add:
      - NET_ADMIN
    # 設定容器的網路設定，這邊使用 host 模式，也就是使用主機的網路設定
    network_mode: host
    # 設定容器的掛載設定，這邊使用主機的 /etc/wireguard 資料夾掛載到容器的 /etc/wireguard 資料夾
    volumes:
      - /etc/wireguard:/etc/wireguard
      - ./data:/app/data
      - ./config.yml:/app/config.yml
```

2. `config.yml` 規定了關於 WG-Portal 的所有基本設定，一樣直接看設定：

```yaml
advanced:
  log_level: trace

core:
  admin_user: admin@crazyfirelee.tw
  admin_password: example
  editable_keys: true
  create_default_peer: false
  self_provisioning_allowed: false

web:
  external_url: https://wg-portal.crazyfirelee.tw
  listening_address: :8123
  request_logging: true

database:
  type: sqlite
  database: data/wg-portal.db
```

由於官方文件針對可以設定的參數的說明非常非常的清楚，我們就快速跳過了。

