---
title: "如何在適用於 MySQL 的 Azure 資料庫中還原伺服器"
description: "本文說明如何使用 Azure 入口網站，在適用於 MySQL 的 Azure 資料庫中還原伺服器。"
services: mysql
author: ajlam
ms.author: andrela
manager: kfile
editor: jasonwhowell
ms.service: mysql-database
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: 5bef3f11d0b546fbd6b1161b20d7dfb81e975f99
ms.sourcegitcommit: 782d5955e1bec50a17d9366a8e2bf583559dca9e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/02/2018
---
# <a name="how-to-back-up-and-restore-a-server-in-azure-database-for-mysql-using-the-azure-portal"></a>如何使用 Azure 入口網站，在適用於 MySQL 的 Azure 資料庫中備份和還原伺服器

## <a name="backup-happens-automatically"></a>備份會自動進行
為了能使用還原功能，適用於 MySQL 的 Azure 資料庫伺服器會定期備份。 透過此功能，您可以將伺服器和其所有資料庫還原至更早的時間點 (在新的伺服器上)。

## <a name="prerequisites"></a>先決條件
若要完成本操作說明指南，您需要：
- [適用於 MySQL 的 Azure 資料庫伺服器和資料庫](quickstart-create-mysql-server-database-using-azure-portal.md)

## <a name="set-backup-configuration"></a>設定備份組態

建立伺服器時，您可以在 [定價層] 視窗中，選擇要將伺服器設定為使用本地備援備份或異地備援備份。

> [!NOTE]
> 伺服器建立後，就無法切換其備援類型 (異地備援和本地備援)。
>

透過 Azure 入口網站建立伺服器時，您可以在 [定價層] 視窗中為伺服器選取 [本地備援] 或 [異地備援] 備份。 您也可以在此視窗中選取 [備份保留期限] - 您要儲存伺服器備份的時間長度 (以天數計算)。

   ![定價層 - 選擇備份備援](./media/howto-restore-server-portal/pricing-tier.png)

如需在建立期間設定這些值的詳細資訊，請參閱[適用於 MySQL 的 Azure 資料庫伺服器快速入門](quickstart-create-mysql-server-database-using-azure-portal.md)。

透過下列步驟可變更伺服器上的備份保留期限：
1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 選取適用於 MySQL 的 Azure 資料庫伺服器。 這個動作會開啟 [概觀] 頁面。
3. 在 [設定] 下方的功能表中選取 [定價層]。 您可以使用滑桿將 [備份保留期限] 變更為想要的天數 (7 到 35 天)。
下列螢幕擷取畫面中的期限已增加到 34 天。
![已增加的備份保留期限](./media/howto-restore-server-portal/3-increase-backup-days.png)

4. 按一下 [確認] 以確認變更。

備份保留期限會控制可往回多少時間來擷取時間點還原，因為這會以可用的備份為基礎。 下一節會進一步說明時間點還原。 

## <a name="point-in-time-restore-in-the-azure-portal"></a>Azure 入口網站中的時間點還原
適用於 MySQL 的 Azure 資料庫可讓您將伺服器還原至過去的時間點，並還原至新的伺服器複本。 您可以使用這個新的伺服器來復原資料，或將用戶端應用程式指向這個新的伺服器。

比方說，如果今天中午不小心卸除資料表，您可以還原至中午之前的時刻，然後從該新的伺服器複本擷取遺漏的資料表和資料。 時間點還原是在伺服器層級進行，不是在資料庫層級。

下列步驟會將範例伺服器還原至某個時間點︰
1. 在 Azure 入口網站中，選取適用於 MySQL 的 Azure 資料庫伺服器。 

2. 在伺服器 [概觀] 頁面的工具列中，選取 [還原]。

   ![適用於 MySQL 的 Azure 資料庫 - 概觀 - 還原按鈕](./media/howto-restore-server-portal/2-server.png)

3. 在 [還原] 表單中填入必要資訊︰

   ![適用於 MySQL 的 Azure 資料庫 - 還原資訊 ](./media/howto-restore-server-portal/3-restore.png)
  - **還原點**：選取您想要還原到那時候的時間點。
  - **目標伺服器**：提供新伺服器的名稱。
  - **位置**：您無法選取區域。 它預設與來源伺服器相同。
  - **定價層**：進行時間點還原時，您無法變更這些參數。 它與來源伺服器相同。 

4. 按一下 [確定] 將伺服器還原至某個時間點。 

5. 完成還原時，找出已建立的新伺服器，確認資料如預期般還原。

>[!Note]
>請注意，所選時間點上的現有伺服器與時間點還原所建立的新伺服器，具有相同且有效的伺服器管理員登入名稱和密碼。 您可以從新伺服器的 [概觀] 頁面變更密碼。

## <a name="next-steps"></a>後續步驟
- 深入了解服務的[備份](concepts-backup.md)。
- 深入了解[商務持續性](concepts-business-continuity.md)選項。
