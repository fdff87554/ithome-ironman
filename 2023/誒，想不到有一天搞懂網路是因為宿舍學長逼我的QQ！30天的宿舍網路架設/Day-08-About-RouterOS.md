# Day-08 - 第二個子網域維護！第一個軟路由之路，RouterOS！

硬體不夠軟體來湊！想當然今天在搞不到 Router 的狀況下，我們就自己搞一個！但究竟這台軟路由要由誰來架設？要怎麼架設？他可以擔任怎樣的角色呢？讓我們今天慢慢來聊聊吧！

## RouterOS 是什麼？

RouterOS 顧名思義是一個專門為 Router 所需要功能所設計的網路作業系統，他是基於 Linux Kernel 所延伸，並且由於其是由 MikroTik 公司所開發，因此也會預裝在所有由 MikroTik 生產的路由器上，因此我們等等會看到他就跟可愛的 ASUS Router 的管理頁面很像的 RouterOS 畫面。那為什麼我們這邊選擇 RouterOS 呢？當然是因為 RouterOS 是開放下載使用的啊！因此真的是我們建立軟路由很合適的作業系統。

再次對焦一下為什麼我們今天需要建立 RouterOS 的軟路由，我們希望將虛擬機器跟實體機器的環境做獨立與隔離，確保實驗的事情讓實驗自己來，原本的物理設備不管是 ASUS Router / Desktop / Laptop 等等都完全獨立開，甚至如果要把環境的位階來出來，應該是這些實體設備可以接觸虛擬機器，但反之虛擬機器的網路不能主動連線。

因此讓我們先從安裝 RouterOS 開始，一路講到如何達成目標吧！

### RouterOS 的安裝

RouterOS 的檔案可以直接從 MikroTik 的官方網站取得，https://mikrotik.com/software 。

> ![MikroTik RouterOS Download Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/MikroTik-RouterOS-Download-Page.png)

那我們先在 PVE 中開好虛擬機，讓我們來看看 PVE 開虛擬機的步驟。

1. 首先點擊 Create VM 來開啟全新的虛擬機器
   > ![PVE Create VM](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Create-VM.png)
2. 在設定頁面裡面，**General** 的設定的部分
   我們會需要先選擇 `Node`（由於 PVE 系統可以多設備串連，所以這邊會需要選擇你的節點）。
   `VM ID` 則可以想成設備的 ID 編號，那這邊這個編號是從 100~9999 這個範圍，那一般來說沒有特別規定與限制，但建議可以依照服務種類之類的來區分。
   `Name` 就是幫 VM 取名字，看怎麼識別都可以，我這邊就直接取名叫 `routeros`。
   **請一定要記得你的 `VM ID`，等等會用到**。
   > ![PVE Create VM General Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Create-VM-General-Page.png)
3. OS 的部分我們先不要選擇，選 `Do not use any media` 於後期我們在選擇。
   > ![PVE Create VM OS Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Create-VM-OS-Page.png)
4. System 則維持 Default 就好。
   > ![PVE Create VM System Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Create-VM-System-Page.png)
5. Disks 的部分一般來說是用來設定硬體資源，但這邊也可以先跳過，等等會刪除。
   > ![PVE Create VM Disks Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Create-VM-Disks-Page.png)
6. CPU 的部分就是設定要飛配多少硬體資源做使用，那這邊可以依照設備有的資源做調配，另外就是 RouterOS 需要的環境壓力其實蠻小的，因此如果資源拮据就給 1 Cores 就好，要不然也可以給 2~4 個。
   > ![PVE Create VM CPU Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Create-VM-CPU-Page.png)
7. Memory 的部分則是更隨意，由於 RouterOS 平時所需要的 Memory 平均在 170~200 MiB 左右而已，因此如果設備侷限，給 256 MB 就已經很足夠了，我是給到了 2GB。
   > ![RouterOS Memory Usage](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-Memory-Usage.png)
   >
   > ![PVE Create VM Memory Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Create-VM-Memory-Page.png)
8. 接下來網卡的部分，一樣預設就可以了，因為現在我們的 PVE 只有一個網卡，晚點我們在處理剩下的部分。
   > ![PVE Create VM Network Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-Create-VM-Network-Page.png)
9. 最後就是總結頁面，基本上沒問題就按下 Finish 即可。

接下來我們要開始替換我們的 VM Disk 成 RouterOS CHR 映像檔上去，因此讓我們點選我們設定完成的 VM。

1. 選擇 Hardware -> 選擇 Hard Disk -> 選擇 Detach 分離出來
   > ![PVE VM Hardware Settings - Detach Hardware](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-VM-Hardware-Settings_Detach-Hardware.png)
2. 點選剛剛分離出來的 Unused Disk -> 選擇 Remove 刪除
   > ![PVE VM Hardware Settings - Remove Disk](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-VM-Hardware-Settings_Remove-Disk.png)
3. 到剛剛的 MikroTik 官網，找到 CHR（Cloud Hosted Router） 的 Raw disk image 選擇 Stable 的任何一個開心版本都可以，那截圖時我是直接選最新版本的 Stable 版。右鍵複製 Download 的 Url 之後，我們等等會在 PVE 裡面用到。
   > ![RouterOS CHR Download Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-CHR-Download-Page.png)
4. 回到 PVE 的 Shell 中，依照下方指令完成
   1. 檔案下載
   2. 解壓縮
   3. 設定空間大小
   4. 掛載 disk
   ```bash
   # 下載剛剛的 Raw Disk Image 檔案
   $ wget https://download.mikrotik.com/routeros/7.11.2/chr-7.11.2.img.zip
   # 解壓縮檔案，並取得 chr-7.11.2.img
   $ unzip chr-7.11.2.img.zip
   # 幫檔案增加一些空間（也就是 Disk Size 的部分），其實 RouterOS 不大，看學長設定是只有 128M，啊網路上一般來說建議可以加到 1G
   $ qemu-img resize chr-7.11.2.img +1G
   # 把剛剛的 .img 掛載到 VM 底下，這邊會用到剛剛記得的 VM ID，我們這邊是剛剛示範的 102，請記得改成自己的編號
   $ qm importdisk 102 chr-7.11.2.img local-lvm
   ```
   > ![wget Download RouterOS to PVE](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/wget-Download-RouterOS-to-PVE.png)
   >
   > ![unzip RouterOS in PVE](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/unzip-RouterOS-in-PVE.png)
   >
   > ![resize RouterOS image in PVE](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/resize-RouterOS-image-in-PVE.png)
   >
   > ![importdisk of RouterOS image to VM](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/importdisk-of-RouterOS-image-to-VM.png)
5. 當我們看到 Successfully 的字樣之後，就可以回到 RouterOS VM 的頁面，可以看到一個 unused 的 Disk 被掛載完成。
   > ![Get unused disk 0 for RouterOS VM](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Get-unused-disk-0-for-RouterOS-VM.png)
   
   然後我們直接點擊兩下，直接把 Disk 加入即可。
   
   > ![Add unsed disk 0 to VM](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Add-unsed-disk-0-to-VM.png)
6. 把新的 Disk 掛到啟動順序中，到 Options -> 選擇 Boot Order -> 把剛剛掛載的 Disk 新增到啟動中。
   > ![Change Boot Order of VM](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Change-Boot-Order-of-VM.png)
7. 這時候我們順便再添加一個網卡進去，這個網卡是專門給 VM 使用的網卡，也就是另外一個子網域的虛擬網卡。再有這張網卡 + 有這個準備啟動的軟路由之後，我們就會取得可愛的 VM 子網域了！要生成這個第二個虛擬網卡，我們先回到 PVE 的 System -> Network -> 點擊 Create -> 選擇 Linux Bridge 來創建新的網卡。
   在 Edit 中，全部都不用填寫，我們剩下會用 RouterOS 來設定，這邊可以稍微用 Comment 註記你這個網卡是用來做什麼即可，從我的註記可以看到，我是希望提供 `192.168.52` 網域作為 VM 的子網域而開的網卡。
   > ![Create Linux Bridge Network 1](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Create-Linux-Bridge-Network-1.png) > ![Create Linux Bridge Network 2](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Create-Linux-Bridge-Network-2.png)
8. 回到 RouterOS 的 Hardware 設定，除了預設的 vmbr0 以外，我們在幫他添加我們剛剛新增的網卡。點選 Hardware -> 選擇 Add -> 選擇 Network Device -> 選擇 vmbr2（就是你剛剛新增的網卡，也可能是 vmbr1，請依照狀況調整）
   > ![Add Network to RouterOS VM 1](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Add-Network-to-RouterOS-VM-1.png)
   >
   > ![Add Network to RouterOS VM 2](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Add-Network-to-RouterOS-VM-2.png)
9. 到這邊我們所有設定就完成了，就讓我們啟動吧！看到 Mikrotik Login 即代表安裝完成！
   > ![Start RouterOS](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Start-RouterOS.png)

使用預設帳號 `admin` 跟空白密碼（直接按 Enter）來完成剩下安裝！

### RouterOS 的設定

我們完成安裝了，並且也設定完 `admin` 帳號的新密碼，接下來我們就要開始設定各種東西了，但首先，要連上 RouterOS 我們一樣要知道他的 IP，並完成初次設定。

在這邊我們依照以下步驟可以取得由 DHCP 給的第一次 IP，不管之後要不要使用 Static IP 都可以在接下來的 Web GUI 中設定，先讓我們找到現在的 IP。

```bash
# 找到 ip address
/ip/address
# 印出現在的 ip address
print
```

> ![find default RouterOS IP Address](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/find-default-RouterOS-IP-Address.png)

從圖上可以看到第一次取得的 IP 是 `192.168.50.140`，連線上之後可以看到 RouterOS 跳出了第一次的設定畫面（再輸入帳號密碼之後）。

> ![RouterOS First Setting Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-First-Setting-Page.png)

在設定頁面中，我們分別依照我們的需求完成 `Internet` / `Local Network` 等設定，這邊其實跟我們前幾天在設定 ASUS Router 時的概念一模一樣，請大家試著理解看看下方截圖的設定。

> ![RouterOS Router Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/RouterOS-Router-Setting.png)

到這邊我們再連線新給的 IP `192.168.50.250`，就可以重新看到我們的...誒，東西勒 OAO？

To Be Continue...

## Reference

- [虛擬機跑起來！RouterOS CHR 軟路由效能輕鬆突破 1000M！](https://www.jkg.tw/p2531/)
- https://wiki.mikrotik.com/wiki/Manual:IP/Address
- [Install MikroTik Cloud Hosted Router (CHR) on Proxmox](https://mikrotik-blog.com/install-mikrotik-cloud-hosted-router-chr-on-proxmox)
