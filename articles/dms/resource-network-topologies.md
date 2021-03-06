---
title: 使用 Azure 資料庫移轉服務進行 Azure SQL DB 受控執行個體移轉的網路拓樸 | Microsoft Docs
description: 了解資料庫移轉服務的來源和目標設定。
services: database-migration
author: HJToland3
ms.author: jtoland
manager: ''
ms.reviewer: ''
ms.service: database-migration
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 03/06/2018
ms.openlocfilehash: 892cff02b5b70f09236bb37ae786f180ddca9316
ms.sourcegitcommit: 168426c3545eae6287febecc8804b1035171c048
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/08/2018
---
# <a name="network-topologies-for-azure-sql-db-managed-instance-migrations-using-the-azure-database-migration-service"></a>使用 Azure 資料庫移轉服務進行 Azure SQL DB 受控執行個體移轉的網路拓樸
在本文中，您將了解 Azure 資料庫移轉服務進行內部部署 SQL Server 到 Azure SQL Database 受控執行個體移轉時，用於提供順暢移轉體驗的各種網路拓撲。

## <a name="azure-sql-database-managed-instance-configured-for-hybrid-workloads"></a>針對混合式工作負載設定 Azure SQL Database 受控執行個體 
如果您的 Azure SQL Database 受控執行個體是連線到內部部署網路，請使用此拓撲。 這種方法提供最簡單的網路路由，並在移轉期間產生最大的資料輸送量。

![混合式工作負載的網路拓撲](media\resource-network-topologies\hybrid-workloads.png)

**需求**
- 在此案例中，Azure SQL Database 受控執行個體和 Azure 資料庫移轉服務執行個體是建立在相同 Azure VNET 中，但是會使用不同的子網路。  
- 此案例中使用的 VNET 會使用 ExpressRoute 或 VPN 連線到內部部署網路。

## <a name="azure-sql-database-managed-instance-isolated-from-the-on-premises-network"></a>Azure SQL Database 受控執行個體與內部部署網路分開
如果您的環境需符合一或多個下列情況案例，請使用此網路拓撲：
- Azure SQL Database 受控執行個體與內部部署的連線功能獨立分開，但您的 Azure 資料庫移轉服務執行個體是連線到內部部署網路。
- 使用角色型存取控制 (RBAC) 原則，並限制使用者存取裝載 Azure SQL Database 受控執行個體的訂用帳戶。
- Azure SQL Database 受控執行個體和 Azure 資料庫移轉服務使用的 VNET 位於不同訂用帳戶。

![受控執行個體與內部部署網路分開的網路拓撲](media\resource-network-topologies\mi-isolated-workload.png)

**需求**
- 此案例中 Azure 資料庫移轉服務使用的 VNET 也必須使用 ExpressRoute 或 VPN 連線到內部部署網路 。
- 在 Azure SQL Database 受控執行個體和 Azure 資料庫移轉服務使用的 VNET 之間，建立 VNET 網路同儕節點。


## <a name="cloud-to-cloud-migrations"></a>雲端至雲端的移轉
如果來源 SQL Server 裝載在 Azure 虛擬機器上，請使用此拓撲。

![雲端至雲端移轉的網路拓撲](media\resource-network-topologies\cloud-to-cloud.png)

**需求**
- 在 Azure SQL Database 受控執行個體和 Azure 資料庫移轉服務使用的 VNET 之間，建立 VNET 網路同儕節點。

## <a name="see-also"></a>另請參閱
- [將 SQL Server 遷移至 Azure SQL Database 受控執行個體](https://docs.microsoft.com/azure/dms/tutorial-sql-server-to-managed-instance)
- [使用 Azure 資料庫移轉服務的必要條件概觀](https://docs.microsoft.com/azure/dms/pre-reqs)
- [使用 Azure 入口網站建立虛擬網路](https://docs.microsoft.com/azure/virtual-network/quick-create-portal)

## <a name="next-steps"></a>後續步驟
如需 Azure 資料庫移轉服務和公開預覽期間區域可用性的概觀，請參閱[什麼是 Azure 資料庫移轉服務預覽](dms-overview.md)一文。 