# Day-28 - 藉由 Github Education Pack 提供的 Mailgun 額度，設定屬於自己的 Mail Service

前幾天我們在架設各種服務的過程中，許多服務都會要求使用者提供 Email 地址以便通知與驗證。不論是域名驗證、服務通知、密碼重置，Email 都是一個重要的溝通工具。今天我們來看看為何我們需要 Email 服務，以及如何使用 Mailgun 與 Github Education Pack 提供的免費額度來設置自己的 Email 服務。

## 關於 PVE 等服務的 Email 通知功能

當我們使用像是 PVE 這種虛擬化平台時，有時系統或應用程式會有需要通知管理員的情況，例如：資源不足、錯誤發生或任務完成。這些通知都會透過 Email 送出。

實際上，這些系統都內建了 SMTP（Simple Mail Transfer Protocol）客戶端功能，透過 SMTP 伺服器來發送 Email。在配置時，你只需要提供 SMTP 伺服器的地址、端口、認證資訊，系統就能將通知發送到指定的 Email 地址。

### 什麼是 SMTP

SMTP（Simple Mail Transfer Protocol）是一種用於發送電子郵件的協定。當我們按下「發送」按鈕時，我們的郵件客戶端（如 Outlook、Thunderbird 或 Gmail 的網頁界面）會使用 SMTP 將郵件傳送到 SMTP 伺服器。該伺服器再將郵件路由到接收者的電子郵件伺服器。SMTP 只負責發送郵件，不包括接收郵件。

### 怎麼接收 Email

當我們談到接收電子郵件時，最常見的兩種協定是 IMAP 和 POP3。

#### IMAP（Internet Message Access Protocol）

IMAP 是一種用於從伺服器接收和存儲電子郵件的協定。與 POP3 不同，當你使用 IMAP 時：

- 郵件始終保留在伺服器上，並只是暫時地下載到本地設備。
- 所有的動作，如標記為已讀、刪除等，都會同步回伺服器。
- 這意味著，如果你使用多個設備訪問你的郵件，所有的設備都會看到相同的郵件狀態。

#### POP3（Post Office Protocol version 3）

與 IMAP 相比，POP3 是用來從伺服器接收電子郵件的更傳統協定。當使用 POP3 時：

- 郵件將從伺服器下載到本地設備。
- 一旦下載完成，郵件通常會從伺服器上刪除。
- 這意味著，如果你使用多個設備訪問你的郵件，它們之間不會同步。

## 如何自己架設 Email Server

自建郵件伺服器可以提供數據隱私和完全控制，但這也涉及到許設定和維護。

- 選擇軟體: 有多種開源的郵件伺服器軟體可供選擇，例如 Postfix（用於 SMTP）和 Dovecot（用於 IMAP 和 POP3）。
  DNS 設定: 必須配置 DNS 的 MX（Mail Exchange）記錄，以告知其他伺服器如何找到你的郵件伺服器。
  安全措施: 設定 SSL/TLS 加密，確保安全連接。還應考慮使用 SPF, DKIM, 和 DMARC 以增加信譽度。
  垃圾郵件過濾: 使用工具如 SpamAssassin 幫助過濾垃圾郵件。

### 什麼是 MX 記錄

MX 記錄（Mail Exchange Record）是一種 DNS 記錄，用於指定接收電子郵件的伺服器。當我們向某人發送電子郵件時，我們的郵件伺服器會查詢 MX 記錄，以找到接收者的郵件伺服器。

### 什麼是信譽度？

信譽度是一種評估郵件伺服器可信度的指標。它是由接收者的郵件伺服器計算的，並用於決定是否接受郵件。如果你的信譽度太低，你的郵件可能會被標記為垃圾郵件，或者根本不會被接收。

### 藉由 Mailgun 來實現自己的 Email Server

由於從零自己架設 Mail Server 會有**信用度很低所以一直掉信的問題**，所以一般來說我們會盡量使用第三方的 Email 服務來寄送信件。而 Mailgun 就是一個提供 Email 服務的平台，並且還提供了一系列的功能與服務，例如信件追蹤、垃圾信件過濾、信件排程等等，那剛好如果你是學生，會跟我一樣有 github education pack 的話，就可以享有免費的使用額度。

就讓我們來看看如何使用 Mailgun 來實現自己的 Email Server 吧！

#### 添加 Domain 與 DNS 設定

剛剛有提到了，要成功使用 Mailgun 協助我們寄信 or 寫信，首先我們需要先註冊一個 Domain，並且將 DNS 設定好，讓 Mailgun 可以幫我們寄信。

那進入 Mailgun 的主畫面，可以看到一個漂漂亮亮的 Dashboard，那首先我們要先新增接下來要讓 Mailgun 幫我們寄信的 Domain，點選左邊的 Domains，然後點選 Add New Domain，輸入你的 Domain 名稱，然後點選 Add Domain。

> ![Mailgun Add New Domains](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Mailgun-Add-New-Domains.png)

在 Add Domain 的設定畫面中，我們可以看到要提供 Domain Name，非常簡單，那在右邊的說明中可以發現 Mailgun 是提倡不要直接使用 Root Domain 註冊，而是使用 Subdomain，例如：`mg.example.com`，這樣的好處是可以避免 Root Domain 的 DNS 設定被覆蓋，而且也可以避免 Root Domain 的信譽度被影響。但由於收信的時候，如果要收到來自 Root Domain 的信件，還是需要設定 Root Domain 的 DNS 設定，所以我們就直接使用 Root Domain 來註冊。

> ![Mailgun Add New Domain Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Mailgun-Add-New-Domain-Setting.png)

當點下 Add Domain 後，會進到 Domain 的 Overview 畫面，裡面這時候會開始有手把手的設定教學，但基本上就是要我們設定 DNS 記錄，讓 Mailgun 可以幫我們寄/收信。主要的 DNS 設定有

1. TXT 記錄：TXT 記錄通常用於存儲文字資訊，它可以用於各種用途，但在郵件服務中，它常常用於驗證域名所有權或設定反垃圾郵件策略。
2. MX 記錄：MX 記錄告訴其他郵件伺服器要將郵件傳送到哪裡。它指向負責處理你域名電子郵件的伺服器。
3. CNAME 記錄：CNAME 記錄用於將一個域名別名映射到另一個域名。在郵件設定中，它通常用於將子域名如 mail.example.com 重定向到 Mailgun 或其他郵件提供商的主機名。

那這邊的相關設定在 `Follow these steps to verify your domain` 都有詳細的說明，所以我們就直接依照說明來設定就可以了。

> ![Follow these steps to verify your domain](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Follow-these-steps-to-verify-your-domain.png)

那上面關於 DNS 的設定就要到我們的域名註冊商或者正在協助管理域名的 DNS 服務上做設定，全部設定完之後，就可以點選 Check `Verify DNS settings` 來確認 DNS 設定是否正確，下面所有項目的 `Unverified` 都變成 `Verified` 就代表設定成功了。

> ![Verified / Unverified Example](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Verified-Unverified-Example.png)

那到這邊我們就已經可以在等待 DNS 設定生效之後，開始使用 Mailgun 來寄信了！
