---
title: 在 Azure 上執行適用於多層式架構應用程式的 Linux VM
description: 如何在 Microsoft Azure 上執行適用於多層式架構的 Linux VM。
author: MikeWasson
ms.date: 11/22/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 8d3e6e5124a0abb27a3c72e1ecbd52a1a1da2a33
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
---
# <a name="run-linux-vms-for-an-n-tier-application"></a>執行適用於多層式架構應用程式的 Linux VM

此參考架構顯示一組經過證實的作法，可用來執行適用於多層式架構應用程式的 Linux 虛擬機器 (VM)。 [**部署這個解決方案**。](#deploy-the-solution)  

![[0]][0]

下載這個架構的 [Visio 檔案][visio-download]。

## <a name="architecture"></a>架構

有許多方式可實作多層式架構。 下圖顯示典型的 3 層式架構 Web 應用程式。 此架構是根據[執行負載平衡的 VM 以獲得延展性和可用性][multi-vm]建置的。 Web 及 Business 層會使用負載平衡的 VM。

* **可用性設定組。** 針對每一層建立[可用性設定組][azure-availability-sets]，並且在每一層中至少佈建兩個 VM。  這讓 VM 能夠符合適用於 VM 之較高[服務等級協定 (SLA)][vm-sla] 的資格。 您可以在可用性設定組中部署單一 VM，但該單一 VM 無法當做 SLA 保證使用，除非該單一 VM 的所有作業系統及資料磁碟是使用 Azure 進階儲存體。  
* **子網路。** 針對每一層建立不同的子網路。 使用 [CIDR] 標記法來指定位址範圍和子網路遮罩。 
* **負載平衡器。** 使用[網際網路面向的負載平衡器][load-balancer-external]，將連入的網際網路流量散發到 Web 層，並使用[內部負載平衡器][load-balancer-internal]，將來自 Web 層的網路流量散發到 Business 層。
* **Azure DNS**。 [Azure DNS][azure-dns] 是 DNS 網域的主機服務，採用 Microsoft Azure 基礎結構提供名稱解析。 只要將您的網域裝載於 Azure，就可以像管理其他 Azure 服務一樣，使用相同的認證、API、工具和計費方式來管理 DNS 記錄。
* **Jumpbox。** 也稱為[防禦主機]。 網路上系統管理員用來連線到其他 VM 的安全 VM。 Jumpbox 具有 NSG，能夠儘允許來自安全清單上公用 IP 位址的遠端流量。 NSG 應該允許安全殼層 (SSH) 流量。
* **監視。** 監視軟體 (例如 [Nagios]、[Zabbix] 或 [Icinga]) 可讓您深入了解回應時間、VM 執行時間，以及系統的整體健康情況。 在位於個別管理子網路中的 VM 上安裝監視軟體。
* <strong>NSG。</strong> 使用[網路安全性群組][nsg] (NSG) 來限制 VNet 內的網路流量。 例如，在如下所示的 3 層式架構中，資料庫層不接受來自 Web 前端的流量，只接受來自 Business 層和管理子網路的流量。
* **Apache Cassandra 資料庫**。 藉由啟用複寫和容錯移轉，在資料層提供高可用性。

## <a name="recommendations"></a>建議

您的需求可能和此處所述的架構不同。 請使用以下建議作為起點。 

### <a name="vnet--subnets"></a>VNet / 子網路

當您建立 VNet 時，請判斷您在每個子網路中的資源需要多少個 IP 位址。 使用 [CIDR] 標記法，針對所需的 IP 位址指定子網路遮罩和夠大的 VNet 位址範圍。 使用落在標準[私人 IP 位址區塊][private-ip-space]內的位址空間，其為 10.0.0.0/8、172.16.0.0/12 及 192.168.0.0/16。

選擇不會與您的內部部署網路重疊的位址範圍，以防您稍後需要在 VNet 和您的內部部署網路之間設定閘道。 一旦建立 VNet 之後，就無法變更位址範圍。

請記得以功能和安全性需求來設計子網路。 位於相同層或角色的所有 VM 都應移入相同的子網路，它可以是安全性界限。 如需設計 VNet 和子網路的詳細資訊，請參閱[規劃和設計 Azure 虛擬網路][plan-network]。

針對每個子網路，以 CIDR 標記法來指定子網路的位址空間。 例如，'10.0.0.0/24' 會建立 256 個 IP 位址範圍。 VM 可以使用這其中的 251 個；保留五個。 確定位址範圍不會跨子網路重疊。 請參閱[虛擬網路常見問題集][vnet faq]。

### <a name="network-security-groups"></a>網路安全性群組

使用 NSG 規則來限制各層之間的流量。 例如，在如上所示的 3 層式架構中，Web 層不會直接與資料庫層通訊。 若要強制執行此動作，資料庫層應該封鎖來自 Web 層子網路的連入流量。  

1. 建立 NSG，並將它關聯至資料庫層子網路。
2. 新增規則以拒絕來自 VNet 的所有輸入流量 (在規則中使用 `VIRTUAL_NETWORK` 標記)。 
3. 新增具有較高優先順序的規則，允許來自 Business 層子網路的輸入流量。 此規則會覆寫上一個規則，並讓 Business 層可與資料庫層對話。
4. 新增規則，允許來自資料庫層子網路本身的輸入流量。 此規則允許資料庫層中 VM 之間的通訊，是進行資料庫複寫和容錯移轉所需的。
5. 新增規則，允許來自 Jumpbox 子網路的 SSH 流量。 此規則讓系統管理員能夠從 Jumpbox 連線到資料庫層。
   
   > [!NOTE]
   > NSG 的[預設規則][nsg-rules]允許所有來自 VNet 的輸入流量。 這些規則無法刪除，但您可藉由建立較高優先順序的規則來覆寫它們。
   > 
   > 

### <a name="load-balancers"></a>負載平衡器

外部負載平衡器會將網際網路流量散發到 Web 層。 建立此負載平衡器的公用 IP 位址。 請參閱[建立網際網路面向的負載平衡器][lb-external-create]。

內部負載平衡器會將網路流量從 Web 層散發到 Business 層。 若要為此負載平衡器提供私人 IP 位址，請建立前端 IP 設定，並將它關聯至 Business 層的子網路。 請參閱[開始建立內部負載平衡器][lb-internal-create]。

### <a name="cassandra"></a>Cassandra

我們建議針對生產環境使用 [DataStax Enterprise][datastax]，但這些建議適用於所有的 Cassandra 版本。 如需在 Azure 中執行 DataStax 的詳細資訊，請參閱[適用於 Azure 的DataStax Enterprise 部署指南][cassandra-in-azure]。 

將適用於 Cassandra 叢集的 VM 放置於可用性設定組，以確保會將 Cassandra 複本散發到多個容錯網域和升級網域。 如需容錯網域和升級網域的詳細資訊，請參閱[管理虛擬機器的可用性][azure-availability-sets]。 

針對每個可用性設定組設定三個錯誤網域 (最大值)，以及針對每個可用性設定組設定 18 個升級網域。 這提供仍能平均散發到容錯網域的升級網域數目上限。   

在機架感知的模式中設定節點。 在 `cassandra-rackdc.properties` 檔案中，將容錯網域對應到機架。

您不需要叢集前方的負載平衡器。 用戶端會直接連線至用戶端中的節點。

### <a name="jumpbox"></a>Jumpbox

Jumpbox 將具備最低效能需求，因此，請針對 Jumpbox 選取較小的 VM 大小，例如標準 A1。 

針對 Jumpbox 建立[公用 IP 位址]。 將 Jumpbox 放置於與其他 VM 相同的 VNet，但在個別的管理子網路中。

不允許從公用網際網路對執行應用程式工作負載的 VM 進行 SSH 存取。 對這些 VM 所做的所有 SSH 存取都必須改為透過 Jumpbox 進行。 系統管理員會登入 Jumpbox，然後從 Jumpbox 登入其他 VM。 Jumpbox 允許來自網際網路，但只來自已知且安全 IP 位址的 SSH 流量。

若要保護 Jumpbox，請建立 NSG 並將它套用到 Jumpbox 子網路。 新增 NSG 規則，只允許來自一組安全公用 IP 位址的 SSH 連線。 NSG 可以連接到子網路或 Jumpbox NIC。 在此情況下，我們建議將它連接到 NIC，如此一來，就只允許 SSH 流量到 Jumpbox，即使您將其他 VM 新增至相同的子網路也一樣。

設定其他子網路的 NSG，允許來自管理子網路的 SSH 流量。

## <a name="availability-considerations"></a>可用性考量

將每個層或 VM 角色放入個別的可用性設定組。 

在資料庫層，具有多個 VM 不會自動會轉譯成高度可用的資料庫。 針對關聯式資料庫，您通常需要使用複寫和容錯移轉來實現高可用性。  

如果您需要比[適用於 VM 的 Azure SLA][vm-sla] 所提供更高的可用性，請跨兩個區域複寫應用程式並使用 Azure 流量管理員進行容錯移轉。 如需詳細資訊，請參閱[在多個區域中執行 Linux VM 以獲得高可用性][multi-dc]。  

## <a name="security-considerations"></a>安全性考量

請考慮新增網路虛擬設備 (NVA)，在公用網際網路和 Azure 虛擬網路之間建立 DMZ。 NVA 是虛擬設備的通稱，可以執行網路相關的工作，例如防火牆、封包檢查、稽核和自訂路由傳送。 如需詳細資訊，請參閱[實作 Azure 和網際網路之間的 DMZ][dmz]。

## <a name="scalability-considerations"></a>延展性考量

負載平衡器會將網路流量散發到 Web 和 Business 層。 藉由新增 VM 執行個體進行水平調整。 請注意，您可以根據負載，分別調整 Web 和 Business 層。 為了降低因需要維護用戶端親和性而造成的可能複雜性，Web 層中的 VM 應該是無狀態的。 裝載商務邏輯的 VM 也應該是無狀態的。

## <a name="manageability-considerations"></a>管理性考量

使用集中式管理工具 (例如 [Azure 自動化][azure-administration]、[Microsoft Operations Management Suite][operations-management-suite]、[Chef][chef] 或 [Puppet][puppet]) 來簡化整個系統的管理。 這些工具可以合併擷取自多個 VM 的診斷和健康情況資訊，以提供系統的整體檢視。

## <a name="deploy-the-solution"></a>部署解決方案

此參考架構的部署可在 [GitHub][github-folder] 上取得。 

### <a name="prerequisites"></a>先決條件

在您可以將參考架構部署到自己的訂用帳戶之前，必須執行下列步驟。

1. 複製、派生或下載適用於[參考架構][ref-arch-repo] GitHub 存放庫的 zip 檔案。

2. 確定您已在電腦上安裝 Azure CLI 2.0。 若要安裝 CLI，請依照[安裝 Azure CLI 2.0][azure-cli-2] 中的指示進行。

3. 安裝 [Azure 建置組塊][azbb] npm 封裝。

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

4. 從命令提示字元、bash 提示字元或 PowerShell 提示字元中，使用下列其中一個命令登入 Azure 帳戶，並遵循提示進行。

   ```bash
   az login
   ```

### <a name="deploy-the-solution-using-azbb"></a>使用 azbb 部署解決方案

若要以多層式架構 (N-Tier) 應用程式參考架構部署 Linux VM，請依以下步驟進行：

1. 瀏覽至您在上述必要條件步驟 1 中複製存放庫的 `virtual-machines\n-tier-linux` 資料夾。

2. 參數檔案會指定部署中每個 VM 的預設管理員使用者名稱及密碼。 您必須在部署參考架構前先變更這些內容。 開啟 `n-tier-linux.json` 檔案，並用新設定取代每個 **adminUsername** 和 **adminPassword** 欄位。   儲存檔案。

3. 使用如下所示的 **azbb** 命令列來部署參考架構。

   ```bash
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
   ```

如需使用 Azure 組建區塊部署此範例參考架構的詳細資訊，請瀏覽 [GitHub 存放庫][git]。

<!-- links -->
[multi-dc]: multi-region-application.md
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[防禦主機]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://docs.datastax.com/en/datastax_enterprise/4.5/datastax_enterprise/install/installAzure.html
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[datastax]: http://www.datastax.com/products/datastax-enterprise
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-linux
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-rules]: /azure/azure-resource-manager/best-practices-resource-manager-security#network-security-groups
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[公用 IP 位址]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[0]: ./images/n-tier-diagram.png "使用 Microsoft Azure 的多層式架構"

