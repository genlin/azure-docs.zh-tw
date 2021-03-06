---
title: 修改 Azure 虛擬機器擴展集 | Microsoft Docs
description: 修改 Azure 虛擬機器擴展集
services: virtual-machine-scale-sets
documentationcenter: ''
author: gatneil
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: e229664e-ee4e-4f12-9d2e-a4f456989e5d
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/14/2018
ms.author: negat
ms.openlocfilehash: fcca912a8120a51d2f0a454ef0a6341cd5882015
ms.sourcegitcommit: 8aab1aab0135fad24987a311b42a1c25a839e9f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/16/2018
---
# <a name="modify-a-virtual-machine-scale-set"></a>修改虛擬機器擴展集
本文說明如何修改現有的虛擬機器擴展集。 工作包括如何變更擴展集的設定、如何變更在擴展集上執行之應用程式的設定、如何管理可用性等。

## <a name="fundamental-concepts"></a>基本概念

### <a name="scale-set-model"></a>擴展集模型

擴展集具有模型，可擷取擴展集整體的「預期」狀態。 若要查詢擴展集的模型，您可以使用：

* REST API： 

  `GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version={apiVersion}` 
   
  如需詳細資訊，請參閱 [REST API 文件](https://docs.microsoft.com/rest/api/compute/virtualmachinescalesets/get)。

* PowerShell：

  `Get-AzureRmVmss -ResourceGroupName {resourceGroupName} -VMScaleSetName {vmScaleSetName}`
   
  如需詳細資訊，請參閱 [PowerShell 文件](https://docs.microsoft.com/powershell/module/azurerm.compute/get-azurermvmss)。

* Azure CLI： 

  `az vmss show -g {resourceGroupName} -n {vmSaleSetName}` 
   
  如需詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure/vmss?view=azure-cli-latest#az_vmss_show)。

您也可以使用 [Azure 資源總管 (預覽)](https://resources.azure.com) 或 [Azure SDK](https://azure.microsoft.com/downloads/) 來查詢擴展集的模型。

確切的輸出呈現內容取決於您提供給命令的選項。 以下是來自 Azure CLI 的範例輸出：

```
$ az vmss show -g {resourceGroupName} -n {vmScaleSetName}
{
  "location": "westus",
  "overprovision": true,
  "plan": null,
  "singlePlacementGroup": true,
  "sku": {
    "additionalProperties": {},
    "capacity": 1,
    "name": "Standard_D2_v2",
    "tier": "Standard"
  },
  .
  .
  .
}
```

如您所見，這些屬性會套用至擴展集整體。



### <a name="scale-set-instance-view"></a>擴展集執行個體檢視

擴展集也具有執行個體檢視，可擷取擴展集整體的目前「執行階段」狀態。 若要查詢擴展集的執行個體檢視，您可以使用：

* REST API： 

  `GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/instanceView?api-version={apiVersion}` 
   
  如需詳細資訊，請參閱 [REST API 文件](https://docs.microsoft.com/rest/api/compute/virtualmachinescalesets/getinstanceview)。

* PowerShell： 

  `Get-AzureRmVmss -ResourceGroupName {resourceGroupName} -VMScaleSetName {vmScaleSetName} -InstanceView` 
  
  如需詳細資訊，請參閱 [PowerShell 文件](https://docs.microsoft.com/powershell/module/azurerm.compute/get-azurermvmss)。

* Azure CLI： 

  `az vmss get-instance-view -g {resourceGroupName} -n {vmSaleSetName}` 
   
  如需詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure/vmss?view=azure-cli-latest#az_vmss_get_instance_view)。

您也可以使用 [Azure 資源總管 (預覽)](https://resources.azure.com) 或 [Azure SDK](https://azure.microsoft.com/downloads/) 來查詢擴展集的執行個體檢視。

確切的輸出呈現內容取決於您提供給命令的選項。 以下是來自 Azure CLI 的範例輸出：

```
$ az vmss get-instance-view -g {resourceGroupName} -n {virtualMachineScaleSetName}
{
  "statuses": [
    {
      "additionalProperties": {},
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "level": "Info",
      "message": null,
      "time": "{time}"
    }
  ],
  "virtualMachine": {
    "additionalProperties": {},
    "statusesSummary": [
      {
        "additionalProperties": {},
        "code": "ProvisioningState/succeeded",
        "count": 1
      }
    ]
  }
  .
  .
  .
}
```

如您所見，這些屬性會提供擴展集中虛擬機器目前執行階段狀態的摘要。 摘要包括套用至擴展集之延伸模組的狀態 (為畫面簡潔起見省略)。



### <a name="scale-set-vm-model-view"></a>擴展集虛擬機器模型檢視

就像擴展集有模型檢視一樣，擴展集內的每個 VM 也有自己的模型檢視。 若要查詢擴展集的模型檢視，您可以使用：

* REST API： 

  `GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/virtualmachines/{instanceId}?api-version={apiVersion}` 
  
  如需詳細資訊，請參閱 [REST API 文件](https://docs.microsoft.com/rest/api/compute/virtualmachinescalesetvms/get)。

* PowerShell： 

  `Get-AzureRmVmssVm -ResourceGroupName {resourceGroupName} -VMScaleSetName {vmScaleSetName} -InstanceId {instanceId}` 
  
  如需詳細資訊，請參閱 [PowerShell 文件](https://docs.microsoft.com/powershell/module/azurerm.compute/get-azurermvmssvm)。

* Azure CLI： 

  `az vmss show -g {resourceGroupName} -n {vmSaleSetName} --instance-id {instanceId}` 
  
  如需詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure/vmss?view=azure-cli-latest#az_vmss_show)。

您也可以使用 [Azure 資源總管 (預覽)](https://resources.azure.com) 或 [Azure SDK](https://azure.microsoft.com/downloads/) 來查詢擴展集中虛擬機器的模型。

確切的輸出呈現內容取決於您提供給命令的選項。 以下是來自 Azure CLI 的範例輸出：

```
$ az vmss show -g {resourceGroupName} -n {vmScaleSetName}
{
  "location": "westus",
  "name": "{name}",
  "sku": {
    "name": "Standard_D2_v2",
    "tier": "Standard"
  },
  .
  .
  .
}
```

如您所見，這些屬性會描述 VM 本身的設定，而非擴展集整體的設定。 例如，擴展集模型有 `overprovision` 作為屬性，而擴展集內虛擬機器的模型則沒有。 這項差異的原因是，過度佈建是擴展集整體的屬性，不適用於擴展集中的個別虛擬機器。 (如需過度佈建的詳細資訊，請參閱[擴展集的設計考量](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview#overprovisioning)。)



### <a name="scale-set-vm-instance-view"></a>擴展集虛擬機器執行個體檢視

就像擴展集有執行個體檢視一樣，擴展集內的每個 VM 也有自己的執行個體檢視。 若要查詢擴展集的執行個體檢視，您可以使用：

* REST API： 

  `GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/virtualmachines/{instanceId}/instanceView?api-version={apiVersion}` 
 
  如需詳細資訊，請參閱 [REST API 文件](https://docs.microsoft.com/rest/api/compute/virtualmachinescalesetvms/getinstanceview)。

* PowerShell： 

  `Get-AzureRmVmssVm -ResourceGroupName {resourceGroupName} -VMScaleSetName {vmScaleSetName} -InstanceId {instanceId} -InstanceView` 
  
  如需詳細資訊，請參閱 [PowerShell 文件](https://docs.microsoft.com/powershell/module/azurerm.compute/get-azurermvmssvm)。

* Azure CLI： 

  `az vmss get-instance-view -g {resourceGroupName} -n {vmSaleSetName} --instance-id {instanceId}` 
  
  如需詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure/vmss?view=azure-cli-latest#az_vmss_get_instance_view)。

您也可以使用 [Azure 資源總管 (預覽)](https://resources.azure.com) 或 [Azure SDK](https://azure.microsoft.com/downloads/) 來查詢擴展集中虛擬機器的執行個體檢視。

確切的輸出呈現內容取決於您提供給命令的選項。 以下是來自 Azure CLI 的範例輸出：

```
$ az vmss get-instance-view -g {resourceGroupName} -n {vmScaleSetName} --instance-id {instanceId}
{
  "additionalProperties": {
    "osName": "ubuntu",
    "osVersion": "16.04"
  },
  "disks": [
    {
      "name": "{name}",
      "statuses": [
        {
          "additionalProperties": {},
          "code": "ProvisioningState/succeeded",
          "displayStatus": "Provisioning succeeded",
          "time": "{time}"
        }
      ]
    }
  ],
  "statuses": [
    {
      "additionalProperties": {},
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "time": "{time}"
    },
    {
      "additionalProperties": {},
      "code": "PowerState/running",
      "displayStatus": "VM running"
    }
  ],
  "vmAgent": {
    "statuses": [
      {
        "additionalProperties": {},
        "code": "ProvisioningState/succeeded",
        "displayStatus": "Ready",
        "level": "Info",
        "message": "Guest Agent is running",
        "time": "{time}"
      }
    ],
    "vmAgentVersion": "{version}"
  },
  .
  .
  .
}
```

如您所見，這些屬性會描述虛擬機器本身的目前執行階段狀態。 狀態包括套用至擴展集的任何延伸模組 (為畫面簡潔起見省略)。




## <a name="techniques-for-updating-global-scale-set-properties"></a>用來更新全域擴展集屬性的技術

若要更新全域擴展集屬性，您必須更新擴展集模型中的屬性。 您可以透過下列方式執行此操作：

* REST API： 

  `PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version={apiVersion}` 
  
  如需詳細資訊，請參閱 [REST API 文件](https://docs.microsoft.com/rest/api/compute/virtualmachinescalesets/createorupdate)。

  您也可以選擇使用來自 REST API 的屬性來部署 Azure Resource Manager 範本，以更新全域擴展集屬性。

* PowerShell： 

  `Update-AzureRmVmss -ResourceGroupName {resourceGroupName} -VMScaleSetName {vmScaleSetName} -VirtualMachineScaleSet {scaleSetConfigPowershellObject}` 
  
  如需詳細資訊，請參閱 [PowerShell 文件](https://docs.microsoft.com/powershell/module/azurerm.compute/update-azurermvmss)。

* Azure CLI：

  * 若要修改屬性：`az vmss update --set {propertyPath}={value}` 
  
  * 將物件新增至擴展集內的清單屬性：`az vmss update --add {propertyPath} {JSONObjectToAdd}` 
  
  * 從擴展集內的清單屬性移除物件：`az vmss update --remove {propertyPath} {indexToRemove}` 
  
  如需詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure/vmss?view=azure-cli-latest#az_vmss_update)。 
  
  或者，如果您先前已使用 `az vmss create` 命令來部署擴展集，則可以再次執行 `az vmss create` 命令來更新擴展集。 若要這樣做，請確保 `az vmss create` 命令中的所有屬性都與之前相同，除了您想要修改的屬性之外。



您也可以使用 [Azure 資源總管 (預覽)](https://resources.azure.com) 或 [Azure SDK](https://azure.microsoft.com/downloads/) 來更新擴展集模型。

更新擴展集模型之後，新的設定就會套用至擴展集內新建立的所有虛擬機器。 不過，擴展集內現有虛擬機器的模型仍然必須藉由最新的整體擴展集模型來更新至最新狀態。 每個虛擬機器的模型中都有一個名為 `latestModelApplied` 的布林值屬性，此屬性會指出虛擬機器是否已藉由最新的整體擴展集模型更新至最新狀態。 (`true` 值表示虛擬機器已藉由最新的模型更新至最新狀態)。




## <a name="techniques-for-bringing-vms-up-to-date-with-the-latest-scale-set-model"></a>藉由最新擴展集模型將虛擬機器更新至最新狀態的技術

擴展集具有「升級原則」，可決定藉由最新擴展集模型將虛擬機器更新至最新狀態的方式。 升級原則的三個模式為：

- **自動**：在此模式下，擴展集不保證虛擬機器的關閉順序。 擴展集可能同時關閉所有虛擬機器。 
- **輪流**：在此模式下，擴展集會分批進行更新，批次之間會有選擇性的暫停時間。
- **手動**：在此模式下，當您更新擴展集模型時，現有的虛擬機器將不受影響。 若要更新現有的虛擬機器，您必須手動升級每個虛擬機器。 您可以透過下列方式執行這個手動升級：

  - REST API： 
  
    `POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/manualupgrade?api-version={apiVersion}` 
    
    如需詳細資訊，請參閱 [REST API 文件](https://docs.microsoft.com/rest/api/compute/virtualmachinescalesets/updateinstances)。

  - PowerShell： 
  
    `Update-AzureRmVmssInstance -ResourceGroupName {resourceGroupName} -VMScaleSetName {vmScaleSetName} -InstanceId {instanceId}` 
    
    如需詳細資訊，請參閱 [PowerShell 文件](https://docs.microsoft.com/powershell/module/azurerm.compute/update-azurermvmssinstance)。

  - Azure CLI： 
  
    `az vmss update-instances -g {resourceGroupName} -n {vmScaleSetName} --instance-ids {instanceIds}` 
    
    如需詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure/vmss?view=azure-cli-latest#az_vmss_update_instances)。

  您也可以使用 [Azure SDK](https://azure.microsoft.com/downloads/) 手動升級擴展集中的虛擬機器。

>[!NOTE]
> Azure Service Fabric 叢集只能使用「自動」模式，但處理更新的方式不同。 如需有關 Service Fabric 更新的詳細資訊，請參閱 [Service Fabric 文件](https://docs.microsoft.com/azure/service-fabric/service-fabric-application-upgrade)。

有一種全域擴展集屬性修改並不遵守升級原則：擴展集 OS 設定檔的變更。 (例如系統管理員使用者名稱和密碼)。這些屬性只能在 API 版本 2017-12-01 或更新版本中變更。 這些變更只會套用至變更擴展集模型之後建立的虛擬機器。 若要將現有的虛擬機器更新至最新狀態，您必須為每個現有的虛擬機器執行重新安裝映像作業。 透過以下方式為虛擬機器重新安裝映像：

* REST API： 

  `POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/reimage?api-version={apiVersion}` 
  
  如需詳細資訊，請參閱 [REST API 文件](https://docs.microsoft.com/rest/api/compute/virtualmachinescalesets/reimage)。

* PowerShell： 

  `Set-AzureRmVmssVM -ResourceGroupName {resourceGroupName} -VMScaleSetName {vmScaleSetName} -InstanceId {instanceId} -Reimage` 
  
  如需詳細資訊，請參閱 [PowerShell 文件](https://docs.microsoft.com/powershell/module/azurerm.compute/set-azurermvmssvm)。

* Azure CLI： 

  `az vmss reimage -g {resourceGroupName} -n {vmScaleSetName} --instance-id {instanceId}` 
  
  如需詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure/vmss?view=azure-cli-latest#az_vmss_reimage)。

您也可以使用 [Azure SDK](https://azure.microsoft.com/downloads/) 為擴展集內的 VM 重新安裝映像。




## <a name="properties-with-restrictions-on-modification"></a>具有修改限制的屬性

### <a name="create-time-properties"></a>建立階段屬性

有些屬性只能在初始建立擴展集時設定。 這些屬性包括：

- 區域
- 映像參考發行者
- 映像參考優惠

### <a name="properties-that-can-be-changed-based-on-the-current-value-only"></a>只能根據目前值變更的屬性

有些屬性可以變更，但除非是根據目前的值。 這些屬性包括：

- `singlePlacementGroup`：如果 `singlePlacementGroup` 為 true，則可修改為 false。 不過，如果 `singlePlacementGroup` 為 false，它「無法」修改為 true。
- `subnet`：只要擴展集的原始子網路與新子網路位於相同的虛擬網路中，便可以修改擴展集的子網路。

### <a name="properties-that-require-deallocation-to-change"></a>必須解除配置才能變更的屬性

有些屬性只有在已將擴展集內虛擬機器解除配置的情況下，才能變更成特定值。 這些屬性包括：

- `sku name`：如果擴展集目前所在的硬體不支援新的虛擬機器 SKU，您必須先將擴展集內的虛擬機器解除配置，才能修改 `sku name`。 如需有關調整 VM 大小的詳細資訊，請參閱[這篇 Azure 部落格文章](https://azure.microsoft.com/blog/resize-virtual-machines/) \(英文\)。


## <a name="vm-specific-updates"></a>VM 特定的更新

某些修改可以套用至特定虛擬機器，而無法套用至全域擴展集屬性。 目前唯一支援的 VM 特定更新是將資料磁碟連結至擴展集內的 VM，或將資料磁碟從那些 VM 中斷連結。 這項功能處於預覽狀態。 如需詳細資訊請，參閱[預覽文件](https://github.com/Azure/vm-scale-sets/tree/master/preview/disk) \(英文\)。

## <a name="scenarios"></a>案例

### <a name="application-updates"></a>應用程式更新

如果透過延伸模組將應用程式部署至擴展集，則更新延伸模組組態時，會造成應用程式根據升級原則進行更新。 例如，如果您有要在自訂指令碼延伸模組中執行的新版指令碼，則可以更新 `fileUris` 屬性以指向新的指令碼。 

在某些情況下，您可能想要強制更新，即使延伸模組組態不會變更。 (例如，您更新指令碼而不變更指令碼的 URI)。在這些情況下，您可以修改 `forceUpdateTag` 來強制更新。 Azure 平台並不會解譯此屬性，因此變更其值對於延伸模組的執行方式並無任何影響。 修改它只是會強制延伸模組重新執行。 

如需有關 `forceUpdateTag` 的詳細資訊，請參閱[延伸模組的 REST API 文件](https://docs.microsoft.com/rest/api/compute/virtualmachineextensions/createorupdate)。

透過自訂映像來部署應用程式也是常見的做法。 在下一節中將會探討此案例。

### <a name="os-updates"></a>OS 更新

如果您使用平台映像，您可以藉由修改 `imageReference` 來更新映像。 如需詳細資訊，請參閱 [REST API 文件](https://docs.microsoft.com/en-us/rest/api/compute/virtualmachinescalesets/createorupdate)。

>[!NOTE]
> 使用平台映像時，通常會指定 "latest" 作為映像參考版本。 這意謂著在進行擴展集建立、相應放大及重新安裝映像時，會使用最新可用版本來建立虛擬機器。 不過，這*並不*意謂著 OS 映像會隨著時間在新映像版本發行時自動更新。 這是個別的功能，目前為預覽版。 如需詳細資訊，請參閱[作業系統自動升級](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade)。

如果您使用自訂映像，您可以藉由更新 `imageReference` 識別碼來更新映像。 如需詳細資訊，請參閱 [REST API 文件](https://docs.microsoft.com/en-us/rest/api/compute/virtualmachinescalesets/createorupdate)。

## <a name="examples"></a>範例

### <a name="update-the-os-image-for-your-scale-set"></a>更新擴展集的 OS 映像

假設您有執行舊版 Ubuntu LTS 16.04 的擴展集。 您想要更新為較新版本的 Ubuntu LTS 16.04 (例如，版本 16.04.201801090)。 映像參考版本屬性不是清單的一部分，因此您可以直接使用下列命令來修改這些屬性：

* PowerShell： 

  `Update-AzureRmVmss -ResourceGroupName {resourceGroupName} -VMScaleSetName {vmScaleSetName} -ImageReferenceVersion 16.04.201801090`

* Azure CLI： 

  `az vmss update -g {resourceGroupName} -n {vmScaleSetName} --set virtualMachineProfile.storageProfile.imageReference.version=16.04.201801090`


### <a name="update-the-load-balancer-for-your-scale-set"></a>更新擴展集的負載平衡器

假設您有一個含有 Azure Load Balancer 的擴展集，而您想要以 Azure 應用程式閘道取代 Load Balancer。 擴展集的負載平衡器和應用程式閘道屬性是清單的一部分。 因此，您可以使用命令來移除及新增清單元素，而不是直接修改屬性。

PowerShell：
```
# Get the current model of the scale set and store it in a local PowerShell object named $vmss
> $vmss=Get-AzureRmVmss -ResourceGroupName {resourceGroupName} -Name {vmScaleSetName}

# Create a local PowerShell object for the new desired IP configuration, which includes the reference to the application gateway
> $ipconf = New-AzureRmVmssIPConfig myNic -ApplicationGatewayBackendAddressPoolsId /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/backendAddressPools/{applicationGatewayBackendAddressPoolName} -SubnetId $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0].Subnet.Id –Name $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0].Name

# Replace the existing IP configuration in the local PowerShell object (which contains the references to the current Azure load balancer) with the new IP configuration
> $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0] = $ipconf

# Update the model of the scale set with the new configuration in the local PowerShell object
> Update-AzureRmVmss -ResourceGroupName {resourceGroupName} -Name {vmScaleSetName} -virtualMachineScaleSet $vmss

```

Azure CLI：
```
az vmss update -g {resourceGroupName} -n {vmScaleSetName} --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerBackendAddressPools 0 # Remove the load balancer back-end pool from the scale set model
az vmss update -g {resourceGroupName} -n {vmScaleSetName} --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerInboundNatPools 0 # Remove the load balancer back-end pool from the scale set model; only necessary if you have NAT pools configured on the scale set
az vmss update -g {resourceGroupName} -n {vmScaleSetName} --add virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].ApplicationGatewayBackendAddressPools '{"id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/backendAddressPools/{applicationGatewayBackendPoolName}"}' # Add the application gateway back-end pool to the scale set model
```

>[!NOTE]
> 這些命令會假設擴展集上只有一個 IP 設定和負載平衡器。 如果有多個，您可能需要使用清單索引而不是 0。