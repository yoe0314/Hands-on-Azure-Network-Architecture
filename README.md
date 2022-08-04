
# Hands-on : Azure Network Architecture
###### tags `Azure Network` `Workshop` `2022`
[![hackmd-github-sync-badge](https://hackmd.io/-Y0NjGc3RgK2_UobhexsUg/badge)](https://hackmd.io/-Y0NjGc3RgK2_UobhexsUg)
- [Hands-on : Azure Network Architecture](#hands-on---azure-network-architecture)
  * [Prerequisites](#prerequisites)
  * [1/ 基本介紹](#1-基本介紹)
  * [2/ 建立虛擬網路](#2-建立虛擬網路)
    + [建立VNet](#建立vnet)
    + [建立Subnet](#建立subnet)
  * [3/ Powershell部署](#3-powershell部署)
    + [Cloud Shell](#cloud-shell)
  * [4/ 建立NSG](#4-建立nsg)
    + [設定輸入安全性規則](#設定輸入安全性規則)
    + [NSG與Subnet結合](#nsg與subnet結合)
  * [5/ 服務端點-Service Endpoints](#5-服務端點-service-endpoints)
    + [建立Service Endpoint](#建立service-endpoint)
  * [6/ VNet Peering](#6-vnet-peering)
    + [新增對等互連](#新增對等互連)
  * [7/ 建立虛擬機器](#7-建立虛擬機器)
    + [建立兩台創虛擬機器-VM1、VM2](#建立兩台創虛擬機器-vm1vm2)
    + [網路配置](#網路配置)
  * [8/ 遠端連線測試](#8-遠端連線測試)
    + [新增自身IP](#新增自身ip)
    + [測試連線方法](#測試連線方法)
    + [測試 VM1 -> VM2連線](#測試-vm1---vm2連線)
    + [測試 VM2 -> VM1連線](#測試-vm2---vm1連線)
  * [9/ 刪除資源](#9-刪除資源)
  * [Reference](#reference)

此Hands-on Lab用來實作一簡單Azure網路架構圖。包含兩個VNet，兩個VNet中各有兩個Subnet，Subnet包含的資源有：Stroage Account、Virtual Machine、NSG。
## Prerequisites

* Azure Subscription (Azure 訂用帳戶)
     * 需指派使用者為訂用帳戶中 Owner/Contributor 權限：[使用 Azure 入口網站指派 Azure 角色-Azure RBAC | Microsoft Docs (建議提供訂閱帳戶這層的權限而非單一資源群組)](https://docs.microsoft.com/zh-tw/azure/role-based-access-control/role-assignments-portal?tabs=current)。

## 1/ 基本介紹
* 在此次練習中，將透過Azure Potal、Powershell逐步建立下方網路架構圖。![](https://i.imgur.com/4mxQrbH.png)

    * 每個VNet都由自己的網路安全性群組(NSG)所防護。
    * 有一個服務端點(Service Endpoint)會把Storage Account連結至VNet。
    * VNet1和VNet2配置對等連接-VNet Peering。
    
* CIDR

| VNet1 | 10.1.0.0/16 |     | VNet2 | 10.2.0.0/16 |
| ----- | ----------- | --- | ----- | ----------- |
| app1  | 10.1.2.0/24 |     | app2  | 10.2.2.0/24 |
| web1  | 10.1.1.0/24 |     | web2  | 10.2.1.0/24 |

## 2/ 建立虛擬網路
### 建立VNet
* 首先登入[Azure Portal](https://portal.azure.com/)，上方Search Bar搜尋`虛擬網路`，並點選建立。

![](https://i.imgur.com/04ccytT.png)

* 選擇欲使用的訂用帳戶，選擇要將接下來建立的資源放在哪個資源群組。
    * 為你的虛擬網路取名，並選擇要建立在哪個區域。

![](https://i.imgur.com/vg61f0o.png)

:::success
:bulb: **建議**：新建一個資源群組存放此次練習所建立的資源，方便後續刪除資源。
:::

* `IP位址`
    * 在IPv4位址填入架構圖規劃的CIDR (10.1.0.0/16)。

![](https://i.imgur.com/IwnyBq8.png)

### 建立Subnet

* 新增子網路(Subnet)
    * 新增兩個Subnet，web1(10.1.1.0/24)、app1(10.1.2.0/24)。

![](https://i.imgur.com/Pmm5aXb.png) ![](https://i.imgur.com/hhBrqDu.png)

* 完成後畫面如下：

![](https://i.imgur.com/Ch2igDw.png)
* 後續`安全性`和`標籤`皆用預設即可，並點選建立。

* 建立成功後，`前往資源`至虛擬網路->`圖表` 查看Azure提供的網路拓樸圖。
![](https://i.imgur.com/S00iUk5.png)



## 3/ Powershell部署
### Cloud Shell
* 剛才我們是使用Azure Portal來建立資源，現在試著透過PowerShell指令的方式來建立看看。
* Azure Portal有提供雲端版本的PowerShell，`Cloud Shell`讓我們可以在瀏覽器上就可以執行相關指令(支援Bash及PowerShell)。
* 點選Search Bar右方的 ![](https://i.imgur.com/4THLrv7.png) 來開啟Cloud Shell。



![](https://i.imgur.com/27XLSFN.png )
* 切換至PowerShell指令

![](https://i.imgur.com/GZXQmAN.png)

* 輸入下方PowerShell指令-新增虛擬網路`VNet2`。

```powershell
New-AzVirtualNetwork ` -ResourceGroupName AzureNetworkNote_HackMD ` -Location JapanEast ` -Name VNet2 ` -AddressPrefix 10.2.0.0/16
```

* Parameters
    * `ResourceGroupName`：資源群組名稱。
    * `Location`：區域。
    * `Name`：虛擬網路名稱。
    * `AddressPrefix`：網段。

![1](https://i.imgur.com/paQMB7g.png)

* 輸入下方PowerShell指令-新增Subnet`web2`。

```powershell
$subnetConfig1 = Add-AzVirtualNetworkSubnetConfig ` -Name web2 ` -AddressPrefix 10.2.1.0/24 ` -VirtualNetwork (Get-AzVirtualNetwork -Name VNet2)
```

```powershell
$subnetConfig1 | Set-AzVirtualNetwork
```
![](https://i.imgur.com/Mg2d4Ji.png)

* 輸入下方PowerShell指令-新增Subnet`app2`。

```powershell
$subnetConfig2 = Add-AzVirtualNetworkSubnetConfig ` -Name app2 ` -AddressPrefix 10.2.2.0/24 ` -VirtualNetwork (Get-AzVirtualNetwork -Name VNet2)
```

```powershell
$subnetConfig2 | Set-AzVirtualNetwork
```
![](https://i.imgur.com/ygRq8vx.png)

* 建立完成後，可至虛擬網路`VNet2`，查看剛才用指令建立的結果。

![](https://i.imgur.com/b7wbpzl.png)


## 4/ 建立NSG
* Azure Portal上方Search Bar搜尋`虛擬網路`，並點選建立。
    * 選擇同樣的資源群組，為NSG取名，選取相同的區域。
    * `標籤`用預設即可，並點選建立。
:::info
:memo: [**Azure 網路安全性群組-NSG**](https://docs.microsoft.com/zh-tw/azure/virtual-network/network-security-groups-overview)
* 網路安全性群組包含安全性規則，用來允許或拒絕進出多種 Azure 資源類型的輸入和輸出網路流量。
* 你可以為每個規則指定來源和目的地、連接埠及通訊協定。
:::

![](https://i.imgur.com/sDeSV5W.png)

### 設定輸入安全性規則
* 若要建立NSG並允許對內的HTTP流量通過(80 TCP Port)。
    * 點選剛才建立好的`NSG1`。
    * 點選`輸入安全性規則`->`新增`。
    * `來源`選擇Service Tag，`來源服務標籤`選擇Internet。
    * `目的地`選擇Service Tag，`目的地服務標籤`選擇VirtualNetwork。
    * `目的地連接埠範圍`輸入80。
    * `通訊協定`選擇TCP。
    * `動作`選擇允許。
    * `優先順序`輸入100。
    * `名稱`輸入"HTTP-IN-Allow"。
    * 點選新增。

![](https://i.imgur.com/IuZzRax.png)

* 為NSG1建立輸出安全性規則，此規則允許由web1 Subnet通往Storage Account的流量，Storage Account位於app1 Subnet。
    * 點選`輸出安全性規則`->`新增`。
    * `來源`選擇Service Tag，`來源服務標籤`選擇VirtualNetwork。
    * `目的地`選擇Service Tag，`目的地服務標籤`選擇Storage。
    * `目的地連接埠範圍`輸入*。
    * `通訊協定`選擇Any。
    * `動作`選擇允許。
    * `優先順序`輸入100。
    * `名稱`輸入"Storage-Out-Allow"。
    * 點選新增。

![](https://i.imgur.com/Dcjs4gZ.png)

* 新增完成後，可至`概觀`查看整體輸入輸出安全性規則。

![](https://i.imgur.com/tBJ8AAB.png)

:::info
:memo: [**服務標籤(service tags)**](https://docs.microsoft.com/zh-tw/azure/virtual-network/service-tags-overview)
* 是一種可以把IP網段(prefixes)集中標示的機制，讓我們在編寫NSG規則時更方便。
* 常見的服務標籤包括：
    * 虛擬網路 - VirtualNetwork
    * 網際網路 - Internet
    * Azure負載平衡器 - Azure LoadBalancer
    * Azure雲端 - AzureCloud
    * 儲存體 - Storage
:::

### NSG與Subnet結合

* NSG建立完成後，要與Subnet做關聯。
* 這個動作要做兩次，一次是VNet1/web1，一次是VNet2/web2。
* 至剛建立好的`NSG1`，點選`子網路`->`+關聯`，`虛擬網路`選擇VNet1，`子網路`選擇web1。

![](https://i.imgur.com/XSJ1yPg.png)
![](https://i.imgur.com/XcYuxSS.png)



## 5/ 服務端點-Service Endpoints

:::info
:memo: [**服務端點(Service Endpoints)**](https://docs.microsoft.com/zh-tw/azure/virtual-network/virtual-network-service-endpoints-overview)
* Service Endpoints會限制特定Azure服務與VM的連接，藉以保護服務。
* 如果你有一個Storage Account其中含有敏感資料，只能讓特定VNet的VM存取。在這個VNet上為該服務帳戶建立一個Service Endpoint就可以達成目的。
* 其他可以經由端點綁定VNet的Azure產品，包括SQL Database、Cosmos DB、金鑰保存庫(Ket Vault)和App Service等。
:::

### 建立Service Endpoint
* 在VNet1的app1 Subnet上建立一個Microsoft.Storage服務端點。
* 至`虛擬網路`點選VNet1，點選`服務端點`->`新增`，`服務`選擇Microsoft.Storage，`子網路`選擇app1。

![](https://i.imgur.com/B2gcwRo.png)

* 在剛建立好的Service Endpoint，點選`···`->`在儲存體帳戶中設定虛擬網路`。

![](https://i.imgur.com/k0pMFew.png)

* 頁面會導向到`儲存體帳戶`，選擇你的儲存體帳戶(若無，可以建立一個)，我們要在這邊做一些網路設定。
* 點選`網路`->`防火牆與虛擬網路`，公用網路存取點選`已從選取的虛擬網路和IP位址啟用`。
* 點選`+新增現有的虛擬網路`->`虛擬網路`選擇VNet1->`子網路`選擇app1，點選新增。

![](https://i.imgur.com/ymmjwgS.png)

* 需要`新增您的用戶端IP`選項加入自己的IP，才能夠存取到儲存體帳戶，否則無法存取。

![](https://i.imgur.com/86lDZ6r.png)


* 用下面這個選項，就可以讓其他的Azure服務(像是Key Vault、Azure AD等)可以和這個儲存體帳戶雙向溝通。

![](https://i.imgur.com/e8c6N2s.png)

## 6/ VNet Peering

* 連結虛擬網路，VNet對等互連(VNet Peering)。
* 在Azure Network中，VNET間預設是不通的，而VNET中的Subnet預設互通，因此我們需要VNet Peering來讓VNet1和VNet2連通。
* 至`虛擬網路`點選VNet1，點選`對等互連`->`+新增`。

![](https://i.imgur.com/FRmsaVh.png)

### 新增對等互連
>:bulb: 現在可以在一端的配置頁面同時完成兩端的配置。

* 輸入`此虛擬網路對等互連連結名稱`：vnet1-to-vnet2-peering。
* `對遠端虛擬網路的流量`及`對遠端虛擬網路轉送的流量`點選允許。
* 輸入`遠端虛擬網路對等互連連結名稱`：vnet2-to-vnet1-peering。
* `虛擬網路`選擇VNet2。

![](https://i.imgur.com/COvczmE.png)

* 我們可以在`VNet1`和`VNet2`的`對等互連`頁面中看到Peering狀態。

![](https://i.imgur.com/iUKtuWK.png)
![](https://i.imgur.com/OrBygkq.png)


## 7/ 建立虛擬機器
### 建立兩台創虛擬機器-VM1、VM2
* 以下步驟要做兩次，一次是VM1、一次是VM2。
    * `資源群組`：選擇一開始建立的。
    * `虛擬機器名稱`：輸入VM1/VM2。
    * `區域`：選擇Japan East(與先前相同)。
    * `可用性選項`：選擇不需要基礎結構備援(僅測試用，不需考慮備援機制)。
    * `安全性類型`：選擇標準。
    * `影像`：可任意選擇，因為比較熟悉Windows，所以選擇Windows Server 2019 Datacenter。
    * `大小`：選擇DS1_v2即可(待會只會登入進去看兩台VM的連線狀況，規格選擇最小的就好)。
    * Administrator帳戶設置(遠端登入所要使用的使用者名稱和密碼)。

![](https://i.imgur.com/WbxVCgE.png)

* `磁碟`預設即可

### 網路配置
* 網路介面

    * `虛擬網路`：選擇VNet1/VNet2。
    * `子網路`：選擇web1/web2。
    * `公用IP`：預設建立(遠端RDP連線使用)。

> VM1，虛擬網路選擇VNet1，子網路選擇web1
VM2，虛擬網路選擇VNet2，子網路選擇web2

![](https://i.imgur.com/J2QPDHl.png)

* `管理`、`進階`預設即可
* 點選建立

## 8/ 遠端連線測試
* 至剛才建立好的VM1，點選`連線`，應該會出現下面錯誤訊息。

![](https://i.imgur.com/hzm1Z11.png)

### 新增自身IP
* 測試連線前要將自己的IP加到NSG，否則就會出現上面的錯誤訊息。
* 點選`網路`->`新增輸入連接埠規則`。
* `來源`：選擇IP Address。
* `來源IP位址/CIDR 範圍`：輸入自己的IP。
    * > :memo: [**快速查詢你的IP**](https://myip.com.tw/)
* `目的地連接埠範圍`：輸入*。
* `通訊協定`：選擇Any。
* `動作`：選擇允許。
* `優先順序`：輸入100。

> :warning: 若與之前HTTP-IN-Allow規則重複，可以把他的優先順序改低，ex:110、120...。

* `名稱`：輸入MYIP-In-Allow。
* 點選新增。

![](https://i.imgur.com/4Io3kcj.png)

* 新增完後的畫面應如下：

![](https://i.imgur.com/WQRLf57.png)

* 在`連線`的部分就不會有錯誤訊息出現。

![](https://i.imgur.com/wCbGLcg.png)

* 點選`下載RDP檔案`來連線至VM1。
* 輸入創建VM時的使用者名稱和密碼登入。

![](https://i.imgur.com/dAcEsau.png)


### 測試連線方法

* 遠端連線進去VM後，我們可以使用PowerShell或PsPing來測試兩台VM間的連線狀況如何。
> [PsPing](https://docs.microsoft.com/en-us/sysinternals/downloads/psping)是微軟推出的一個免費測試IP、ICMP、TCP、頻寬的好工具。
> 下載後將解壓縮完的檔案放到c:\Windows\System32，開啟cmd即可使用PsPing指令。

### 測試 VM1 -> VM2連線
* VM1(10.1.1.4) -> VM2(10.2.1.4)
* PowerShell
```powershell
Test-NetConnection 10.2.1.4 -port 3389
```
![](https://i.imgur.com/nTHUr5w.png)
>VM1 Ping VM2成功 

* PsPing
```PsPing
psping 10.2.1.4:3389
```
![](https://i.imgur.com/AiUH1FU.png)
>VM1 Ping VM2成功 

### 測試 VM2 -> VM1連線
* VM2(10.2.1.4) -> VM1(10.1.1.4)
* PowerShell
```powershell
Test-NetConnection 10.1.1.4 -port 3389
```
![](https://i.imgur.com/EnYaZNF.png)

>VM2 Ping VM1成功 

* PsPing
```PsPing
psping 10.1.1.4:3389
```
![](https://i.imgur.com/dUFwpM1.png)
>VM2 Ping VM1成功

* 經過我們驗證後，VM1和VM2可以互相連線，代表我們在先前做的**VNet Peering**是有效的，能讓VNet1和VNet2互相溝通。


## 9/ 刪除資源
* 上方Search Bar輸入資源群組，然後選取該資源群組。

![](https://i.imgur.com/Cp89LZc.png)

* 點選`刪除資源群組`，並且輸入資源群組名稱即可刪除。

![](https://i.imgur.com/ZLMuflw.png)

![](https://i.imgur.com/S0lcFNk.png)

* 刪除資源後就不會有額外費用產生。

## Reference
* [使用 PowerShell 建立虛擬網路](https://docs.microsoft.com/zh-tw/azure/virtual-network/quick-create-powershell)
* [Azure 建立網路安全性群組](https://docs.microsoft.com/zh-tw/azure/virtual-network/tutorial-filter-network-traffic)
* [Azure 建立、變更或刪除服務端點](https://docs.microsoft.com/zh-tw/azure/virtual-network/virtual-network-service-endpoint-policies-portal)
* [Azure 對等互連](https://docs.microsoft.com/zh-tw/azure/virtual-network/tutorial-connect-virtual-networks-portal)
* [Azure 建立 Windows 虛擬機器](https://docs.microsoft.com/zh-tw/azure/virtual-machines/windows/quick-create-portal)
* [Azure 連線至虛擬機器](https://docs.microsoft.com/zh-tw/azure/virtual-machines/windows/quick-create-portal)
* Microsoft Azure for Dummies, Warner, Timothy L.
