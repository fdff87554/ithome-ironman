# Day-26 - 串連多個 PVE！PVE 的 Cluster / 叢集服務

那不知道大家跟我一起玩 PVE 到現在，有沒有發現一個很微妙的事情，就是雖然我們的操作大部分是 PVE 這個系統本身，但是 PVE 卻不是操作上的最高管理層級與結構，上面還有一層名叫 Data Center（資料中心）的東西，究竟為什麼會有這個層級呢？它在 PVE 的架構中扮演什麼樣的角色呢？

## PVE 的管理層級：從 Data Center 到 VM

首先我們要了解 PVE 的管理層級。當我們打開 PVE 的 Web UI，最上層的結構是「Data Center」。這是 PVE 的最高層級，它允許我們設定全域的設定，例如：網路、儲存、使用者/群組權限等。簡單來說，「Data Center」就是一個集中管理的地方，它包含了所有的節點和虛擬機器。

大家應該還記得當初設定 ACME 相關資訊的時候，ACME 的帳號跟插件就都是在這個位置做設定的，原因是因為不管哪一個 PVE Nodes 再做後續的憑證申請，都可以使用相同的帳號跟插件，這樣就不用每一個 PVE Nodes 都要設定一次，這就是 Data Center 的好處。

再來就是我們主要在設定跟創建虛擬機的單位了，也就是 Nodes（節點）。在 PVE 中，每一台實體伺服器都被視為一個節點。每個節點都可以獨立運行虛擬機器和容器，並且都會運行 PVE。這意味著，你可以在一個「Data Center」中管理多個節點，也就是多台實體伺服器。

因此藉由 Data Center 這個單位，可以一次管理多個服務群體，也正因為如此，他才會以資料中心為名稱，因為藉由這一個層級，我可以做到機房內部多設備的調整與統一管理。

最後就是我們的「VMs/Containers」。這是我們在每個節點上建立和管理的虛擬機器和容器。這些虛擬機器和容器可以運行各種作業系統和應用程式，並且都是在 PVE 的管理之下。

## PVE 的 Cluster / 叢集服務

那既然我們說可以做到一個 Data Center 管理多個節點，究竟要怎麼做到呢？PVE 又是利用什麼概念來完成這個操作的？

在 PVE 中，把所有設備串連起來的概念，就是所謂的 Cluster（叢集），他將每一個設備視為群體中的一個點 Node，並將這些點匡起來 Cluster，這樣就可以做到一次管理多個節點的操作。

那其實要在 PVE 上設定叢集 Cluster 現在是非常方便了（使用版本為 8.0.4 做範例），我們 PVE 已經將整個流程融入到 Web UI 中，只要在 Data Center 的設定中，找到 Cluster 選項就可以開始相關設定了。

> ![PVE DataCenter Cluster Setting Page](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/PVE-DataCenter-Cluster-Setting-Page.png)

### Cluster 的設定

在 Cluster 的概念中，會有幾個角色，分別是：

1. Init Cluster：初始的 Cluster，身為其他所有設備的中心，負責讓其他節點知道應該向誰靠攏。
2. Nodes：設備們，會像 Cluster 做聚攏，會藉由 Init Cluster 來做設定與管理。

但這邊有一個重點，在 Cluster 的世界中，所有 Nodes 都是平等的，這個 Init Cluster 存在的意義只有讓其他 Nodes 知道找誰註冊成為 Cluster 的一部份，而不是說 Init Cluster 會有比其他 Nodes 更高的權限。

那作法很簡單，我們先選定一個設備，然後在剛剛的 Cluster 設定頁面中，點選「Create Cluster」，將自己註冊成為 Init Cluster，作為其他設備的中心。我們下方的 Example 會直接用兩台 PVE 做示範，下方截圖中，會由畫面左邊的 `pve-1` 作為 Init Cluster，右邊的 `pve-2` 則是 Nodes。

> 有一個小重點，就是兩台 PVE Node 的名稱要不一樣，要不然會因為衝突的識別名稱而無法設定。
>
> ![Examples for PVE Cluster Setting](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Examples-for-PVE-Cluster-Setting.png)

首先要到 Data Center 的 Cluster 選項中，先建立 Cluster，這邊我們選擇 `pve-1` 作為 Init Cluster，所以我們在 `pve-1` 的 Cluster 設定頁面中，點選「Create Cluster」，然後就會跳出一個視窗，讓我們輸入 Cluster 的名稱，這邊我們就直接輸入 `Example-Cluster`，然後點選「Create」，就會為我們創建。

> ![Examples for Create Cluster](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Examples-for-Create-Cluster.png)

那需要邀請其他設備加入 Cluster 的時候，需要一些資訊，這些資訊會在「Join Information」中，裡面會有其他設備加入這個 Cluster 會需要的連線資訊等等，等等會用到。

> ![Examples of Join Information](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Examples-of-Join-Information.png)

那今天要加入 Cluster 的設備，就要到 Data Center 的 Cluster 選項中，點選「Join Cluster」後，把剛剛的「Join Information」貼到畫面中的「Information」欄位，就可以看到不管是連線資訊或者其他資訊都會被自動帶入。

> ![Examples for Join Information Flow](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Examples-for-Join-Information-Flow-1.png)
>
> ![Examples for Join Information Flow](https://raw.githubusercontent.com/fdff87554/ithome-ironman/main/2023/%E8%AA%92%EF%BC%8C%E6%83%B3%E4%B8%8D%E5%88%B0%E6%9C%89%E4%B8%80%E5%A4%A9%E6%90%9E%E6%87%82%E7%B6%B2%E8%B7%AF%E6%98%AF%E5%9B%A0%E7%82%BA%E5%AE%BF%E8%88%8D%E5%AD%B8%E9%95%B7%E9%80%BC%E6%88%91%E7%9A%84QQ%EF%BC%8130%E5%A4%A9%E7%9A%84%E5%AE%BF%E8%88%8D%E7%B6%B2%E8%B7%AF%E6%9E%B6%E8%A8%AD/Images/Examples-for-Join-Information-Flow-2.png)

補讓密碼跟連線選項後就可以完成加入！

這時候就可以看到我們的設備們被完成串連啦～
