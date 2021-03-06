---
title: "如何在 Azure 入口網站中建立、管理或刪除儲存體帳戶 | Microsoft Docs"
description: "在 Azure 入口網站中建立新的儲存體帳戶、管理帳戶存取金鑰，或刪除儲存體帳戶。 了解標準和進階儲存體帳戶。"
services: storage
documentationcenter: 
author: tamram
manager: timlt
editor: tysonn
ms.assetid: 87c37da0-6cc6-4d88-a330-ef2896a1531d
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
f1_keywords:
- sql13.swb.windowsazurestorage.connect.f1
ms.date: 10/11/2017
ms.author: tamram
ms.openlocfilehash: dde2ec3b68f5951e268c32b1c6551641f22a0511
ms.sourcegitcommit: b7adce69c06b6e70493d13bc02bd31e06f291a91
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/19/2017
---
# <a name="about-azure-storage-accounts"></a>關於 Azure 儲存體帳戶

[!INCLUDE [storage-selector-portal-create-storage-account](../../../includes/storage-selector-portal-create-storage-account.md)]

[!INCLUDE [storage-table-cosmos-db-tip-include](../../../includes/storage-table-cosmos-db-tip-include.md)]

## <a name="overview"></a>概觀
Azure 儲存體帳戶提供唯一命名空間來儲存及存取您的 Azure 儲存體資料物件。 儲存體帳戶中的所有物件會做為群組共同計費。 根據預設，您帳戶中的資料只有帳戶擁有者 (也就是您) 可以使用。

[!INCLUDE [storage-account-types-include](../../../includes/storage-account-types-include.md)]

## <a name="storage-account-billing"></a>儲存體帳戶計費

[!INCLUDE [storage-account-billing-include](../../../includes/storage-account-billing-include.md)]

> [!NOTE]
> 當您建立 Azure 虛擬機器時，如果您在部署位置中沒有儲存體帳戶，則會在該位置自動建立儲存體帳戶。 因此，您無須依照下方的步驟為虛擬機器磁碟建立儲存體帳戶。 儲存體帳戶名稱將以虛擬機器名稱為基礎。 如需詳細資訊，請參閱 [Azure 虛擬機器文件](https://azure.microsoft.com/documentation/services/virtual-machines/) 。
> 
> 

## <a name="storage-account-endpoints"></a>儲存體帳戶端點
每個儲存在 Azure 儲存體中的物件都有一個唯一 URL 位址。 儲存體帳戶名稱會構成該位址的子網域。 子網域和每個服務的特定網域名稱的組合，會構成儲存體帳戶的 *端點* 。

例如，如果您的儲存體帳戶名為 *mystorageaccount*，則儲存體帳戶的預設端點將是：

* Blob 服務：http://*mystorageaccount*.blob.core.windows.net
* 表格服務：http://*mystorageaccount*.table.core.windows.net
* 佇列服務：http://*mystorageaccount*.queue.core.windows.net
* 檔案服務：http://*mystorageaccount.file.core.windows.net*.file.core.windows.net

> [!NOTE]
> Blob 儲存體帳戶只會公開 Blob 服務端點。
> 
> 

用以存取儲存體帳戶中某物件的 URL，可藉由在端點後附加該物件在儲存體帳戶中的位置來建置。 例如，blob 位址可能會有如下格式︰http://*mystorageaccount*.blob.core.windows.net/*mycontainer*/*myblob*。

您也可以設定與儲存體帳戶搭配使用的自訂網域名稱。 如需詳細資訊，請參閱[針對 Blob 儲存體端點設定自訂網域名稱](../blobs/storage-custom-domain-name.md)。 您也可以使用 PowerShell 加以設定。 如需詳細資訊，請參閱 [Set-AzureRmStorageAccount](/powershell/module/azurerm.storage/set-azurermstorageaccount) cmdlet。  


## <a name="create-a-storage-account"></a>建立儲存體帳戶
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在 Azure 入口網站中，展開左側功能表以開啟服務的功能表，然後選擇 [更多服務]。 然後，向下捲動至 [儲存體]，然後選擇 [儲存體帳戶]。 在出現的 [儲存體帳戶] 視窗上，選擇 [新增]。
3. 輸入儲存體帳戶的名稱。 請參閱 [儲存體帳戶端點](#storage-account-endpoints) 以深入了解此儲存體帳戶名稱如何用來解析 Azure 儲存體中的物件。
   
   > [!NOTE]
   > 儲存體帳戶名稱必須介於 3 到 24 個字元的長度，而且只能包含數字和小寫字母。
   > 
   > 儲存體帳戶名稱必須在 Azure 中是獨一無二的。 Azure 入口網站會指出您選取的儲存體帳戶名稱是否已在使用中。
   > 
   > 
4. 指定所要使用的部署模型：[Resource Manager] 或 [傳統]。  是建議的部署模型。 如需詳細資訊，請參閱 [了解資源管理員部署和傳統部署](../../azure-resource-manager/resource-manager-deployment-model.md)。
   
   > [!NOTE]
   > 僅可使用資源管理員部署模型來建立 Blob 儲存體帳戶。

5. 選取儲存體帳戶的類型︰[一般用途] 或 [Blob 儲存體]。  是預設值。
   
    如果已選取 [一般用途]，則指定效能層︰[標準] 或 [進階]。 預設值是 [標準] 。 如需標準和進階儲存體帳戶的詳細資訊，請參閱 [Microsoft Azure 儲存體簡介](storage-introduction.md)和[進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體](../../virtual-machines/windows/premium-storage.md)。
   
    如果已選取 **Blob 儲存體**，則指定存取層︰[經常存取] 或 [不常存取]。 預設值為 [經常存取] 。 如需詳細資訊，請參閱 [Azure Blob 儲存體：經常存取及不常存取層](../blobs/storage-blob-storage-tiers.md) 。
6. 選取儲存體帳戶的複寫選項︰[LRS]、[GRS]、[RA-GRS] 或 [ZRS]。 預設值是 [RA-GRS] 。 如需 Azure 儲存體複寫選項的詳細資訊，請參閱 [Azure 儲存體複寫](storage-redundancy.md)。
7. 選取您要在其中建立新儲存體帳戶的訂用帳戶。
8. 指定新的資源群組，或選取現有的資源群組。 如需資源群組的詳細資訊，請參閱 [Azure Resource Manager 概觀](../../azure-resource-manager/resource-group-overview.md)。
9. 選取儲存體帳戶的地理位置。 如需各區域可用服務的詳細資訊，請參閱 [Azure 區域](https://azure.microsoft.com/regions/#services) 。
10. 按一下 [建立]  建立儲存體帳戶。

## <a name="manage-your-storage-account"></a>管理儲存體帳戶
### <a name="change-your-account-configuration"></a>變更帳戶組態
建立儲存體帳戶之後，您可以修改其組態，例如變更帳戶所用的複寫選項，或變更 Blob 儲存體帳戶的存取層。 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至您的儲存體帳戶，尋找並按一下 [設定] 之下的 [組態] 以檢視和/或變更帳戶組態。

> [!NOTE]
> 視您在建立儲存體帳戶時選擇的效能層而定，可能無法使用某些複寫選項。
> 
> 

變更複寫選項，將會變更您的價格。 如需詳細資訊，請參閱 [Azure 儲存體價格](https://azure.microsoft.com/pricing/details/storage/) 頁面。

針對 Blob 儲存體帳戶，變更存取層除了會變更您的價格之外，可能還會產生費用的變更。 如需詳細資訊，請參閱 [Blob 儲存體帳戶 - 價格和計費](storage-account-options.md#pricing-and-billing) 。

### <a name="manage-your-storage-access-keys"></a>管理儲存體存取金鑰
當您建立儲存體帳戶時，Azure 會產生兩個 512 位元的儲存體存取金鑰，做為存取儲存體帳戶時的驗證憑藉。 透過提供這兩個儲存體存取金鑰，Azure 讓您可重新產生金鑰，同時又不需中斷儲存體服務或對該服務的存取。

> [!NOTE]
> 建議您避免將儲存體存取金鑰透露給其他任何人。 若要允許存取儲存體資源但不要公開您的存取金鑰，您可以使用「共用存取簽章」 。 共用存取簽章可在您定義的間隔期間內，使用您所指定的權限，來存取帳戶中的資源。 如需詳細資訊，請參閱 [使用共用存取簽章 (SAS)](storage-dotnet-shared-access-signature-part-1.md) 。
> 
> 
<a id="view-and-copy-storage-access-keys"/></a>
#### <a name="view-and-copy-storage-access-keys"></a>檢視並複製儲存體存取金鑰
在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至您的儲存體帳戶，按一下 [所有設定]，然後按一下 [存取金鑰] 圖示來檢視、複製和重新產生帳戶存取金鑰。 [存取金鑰]  刀鋒視窗也包含使用您主要與次要金鑰的預先設定連接字串，讓您可以複製以在應用程式中使用。

#### <a name="regenerate-storage-access-keys"></a>重新產生儲存體存取金鑰
建議您定期變更儲存體帳戶的存取金鑰，保護儲存體連線的安全。 指派了兩個存取金鑰，因此您可以在重新產生一個存取金鑰的同時，使用另一個存取金鑰維持儲存體帳戶連線。

> [!WARNING]
> 重新產生存取金鑰會影響 Azure 中的服務，以及您自己的相依於儲存體帳戶的應用程式。 所有使用存取金鑰來存取儲存體帳戶的用戶端，都必須更新為使用新的金鑰。
> 
> 

**媒體服務** - 如果您有媒體服務相依於儲存體帳戶，您必須在重新產生金鑰之後，將存取金鑰與媒體服務重新同步。

**應用程式** - 如果您有 Web 應用程式或雲端服務使用儲存體帳戶，除非您變換金鑰，否則會在重新產生金鑰後失去連線。

**儲存體總管** - 如果您使用任何 [儲存體總管應用程式](storage-explorers.md)，可能需要更新這些應用程式所使用的儲存體金鑰。

以下是替換儲存體存取金鑰的程序：

1. 更新應用程式程式碼中的連接字串，以參考儲存體帳戶的次要存取金鑰。
2. 重新產生儲存體帳戶的主要存取金鑰。 按一下 [存取金鑰] 刀鋒視窗上的 [重新產生 Key1]，然後按一下 [是] 確認您要重新產生新的金鑰。
3. 更新程式碼中的連接字串，以參考新的主要存取金鑰。
4. 以同樣的方式重新產生次要存取金鑰。

## <a name="delete-a-storage-account"></a>刪除儲存體帳戶
若要移除不再使用的儲存體帳戶，請在 [Azure 入口網站](https://portal.azure.com)中瀏覽至儲存體帳戶，然後按一下 [刪除]。 刪除儲存體帳戶會刪除整個帳戶，包括帳戶中的所有資料。

> [!WARNING]
> 您無法還原已刪除的儲存體帳戶，也無法擷取刪除之前所包含的任何內容。 請務必先備份您想要儲存的任何資料，再刪除帳戶。 這也適用於帳戶中的任何資源 - 一旦刪除 Blob、資料表、佇列或檔案，就是永久刪除。
> 

如果您嘗試刪除與 Azure 虛擬機器相關聯的儲存體帳戶，您可能會收到儲存體帳戶仍在使用中的相關錯誤。 如需此錯誤的疑難排解協助，請參閱[針對刪除儲存體帳戶時的錯誤進行疑難排解](../common/storage-resource-manager-cannot-delete-storage-account-container-vhd.md)。

## <a name="next-steps"></a>後續步驟
* [Microsoft Azure 儲存體總管](../../vs-azure-tools-storage-manage-with-storage-explorer.md) 是一個免費的獨立應用程式，可讓您在 Windows、MacOS 和 Linux 上以視覺化方式處理 Azure 儲存體資料。
* [Azure Blob 儲存體：經常存取及不常存取層](../blobs/storage-blob-storage-tiers.md)
* [Azure 儲存體複寫](storage-redundancy.md)
* [設定 Azure 儲存體連接字串](../storage-configure-connection-string.md)
* [使用 AzCopy 命令列公用程式傳輸資料](storage-use-azcopy.md)
* 造訪 [Azure 儲存體團隊部落格](http://blogs.msdn.com/b/windowsazurestorage/)(英文)。

