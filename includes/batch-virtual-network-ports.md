- VNet 必須與 Batch 帳戶位於相同的 Azure **區域**和**訂用帳戶**。

- 針對使用虛擬機器設定所建立的集區，僅支援以 Azure Resource Manager 為基礎的 VNet。 針對使用雲端服務設定所建立的集區，僅支援傳統 VNet。 
  
- 針對指定的 VNet，若要使用傳統 VNet，`MicrosoftAzureBatch` 服務主體必須有 `Classic Virtual Machine Contributor` 角色型存取控制 (RBAC) 角色。 若要使用以 Azure Resource Manager 為基礎的 VNet，您必須有權存取 VNet 並將 VM 部署在子網路中。

- 針對集區指定的子網路必須有足夠的未指派 IP 位址，可容納目標設為集區的 VM 數目；也就是集區之 `targetDedicatedNodes` 和 `targetLowPriorityNodes` 屬性的總和。 如果子網路沒有足夠的未指派 IP 位址，集區會局部配置計算節點，並發生調整大小錯誤。 

- VNet 必須允許來自 Batch 服務的通訊，才能在計算節點上排程工作。 這個的確認方式是檢查 VNet 是否有任何相關聯的網路安全性群組 (NSG)。 如果 NSG 拒絕對所指定子網路中計算節點的通訊，則 Batch 服務會將計算節點的狀態設為 [無法使用]。 

- 如果指定的 VNet 有相關聯的網路安全性群組 (NSG) 和 (或) 防火牆，請設定輸入和輸出連接埠，如下列各表格所示：


  |    目的地連接埠    |    來源 IP 位址      |   來源連接埠    |    Batch 會新增 NSG 嗎？    |    VM 是否需要有才可使用？    |    使用者的動作   |
  |---------------------------|---------------------------|----------------------------|----------------------------|-------------------------------------|-----------------------|
  |   <ul><li>若為使用虛擬機器組態建立的集區：29876、29877</li><li>若為使用雲端服務設定建立的集區：10100、20100、30100</li></ul>        |    * 或者，為了提高安全性，請指定批次服務的 IP 位址。 若要取得 Batch 服務的 IP 位址清單，請連絡 Azure 支援。 | * 或 443 |    是。 Batch 會在 VM 連結的網路介面 (NIC) 層級新增 NSG。 這些 NSG 只允許來自 Batch 服務角色 IP 位址的流量。 即使您對整個 Web 開啟這些連接埠，流量將會在 NIC 遭到封鎖。 |    yes  |  您不需要指定 NSG，因為 Batch 只允許 Batch IP 位址。 <br /><br /> 不過，如果您指定 NSG，請確定這些連接埠已對輸入流量開啟。 <br /><br /> 如果您指定 * 作為您 NSG 中的來源 IP，Batch 仍會在 VM 連結的 NIC 層級新增 NSG。 |
  |    3389 (Windows)、22 (Linux)               |    使用者電腦 (用於偵錯目的)，以便您從遠端存取 VM。    |   *  | 否                                    |    否                    |    如果您想要允許遠端存取 (RDP 或 SSH) VM，請新增 NSG。   |                                


  |    輸出連接埠    |    目的地    |    Batch 會新增 NSG 嗎？    |    VM 是否需要有才可使用？    |    使用者的動作    |
  |------------------------|-------------------|----------------------------|-------------------------------------|------------------------|
  |    443    |    Azure 儲存體    |    否    |    yes    |    如果您新增任何 NSG，請確定已對輸出流量開啟此連接埠。    |

   此外，請確定為 VNet 提供服務的任何自訂 DNS 伺服器，都能解析您的 Azure 儲存體端點。 具體而言，`<account>.table.core.windows.net`、`<account>.queue.core.windows.net` 和 `<account>.blob.core.windows.net` 形式的 URL 應該可解析。 

   如果您新增以 Resource Manager 作為基礎的 NSG，可以使用[服務標記](../articles/virtual-network/security-overview.md#service-tags)來選取特定地區的儲存體 IP 位址以進行輸出連線。 請注意，儲存體 IP 位址必須與您的 Batch 帳戶和 VNet 位於相同的區域。 服務標記目前在所選的 Azure 區域中為預覽狀態。