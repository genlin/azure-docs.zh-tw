---
title: "在 Azure 入口網站中驗證儲存體帳戶的輸送量和延遲計量 | Microsoft Docs"
description: "了解如何在入口網站中驗證儲存體帳戶的輸送量和延遲計量。"
services: storage
documentationcenter: 
author: georgewallace
manager: jeconnoc
editor: 
ms.service: storage
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: csharp
ms.topic: tutorial
ms.date: 12/12/2017
ms.author: gwallace
ms.custom: mvc
ms.openlocfilehash: b3102bd4e40e10fe88c12295794da37e359c56f1
ms.sourcegitcommit: 922687d91838b77c038c68b415ab87d94729555e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/13/2017
---
# <a name="verify-throughput-and-latency-metrics-for-a-storage-account"></a>驗證儲存體帳戶的輸送量和延遲計量

本教學課程是系列課程的第四個部分，也是最後一個部分。 在前面的教學課程中，您已了解如何將大量隨機資料上傳和下載到 Azure 儲存體帳戶。 本教學課程會示範如何在 Azure 入口網站中使用計量來檢視輸送量和延遲。

在本系列的第一部分中，您了解如何：

> [!div class="checklist"]
> * 在 Azure 入口網站中設定圖表
> * 驗證輸送量和延遲計量

[Azure 儲存體計量](../common/storage-metrics-in-azure-monitor.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)會使用 Azure 監視器來提供儲存體帳戶效能和可用性的統一檢視。

## <a name="configure-metrics"></a>設定計量

在儲存體帳戶中瀏覽至 [設定] 下方的 [計量 (預覽)]。

從 [子服務] 下拉式清單中選擇 Blob。

在 [計量] 下方，選取可在下表中找到的其中一個計量：

下列計量可讓您了解應用程式的延遲和輸送量。 您在入口網站中設定的計量平均值為 1 分鐘。 如果交易在半分鐘內完成，則分鐘資料就是平均值的一半。 在應用程式中，上傳和下載作業時都會計時，並提供您應用程式上傳和下載檔案時實際耗費的時間輸出。 這項資訊可以搭配入口網站計量使用，如此可充分了解輸送量。

|計量|定義|
|---|---|
|**成功 E2E 延遲**|向儲存體服務或所指定 API 作業發出之成功要求的平均端對端延遲。 此值包括 Azure 儲存體內讀取要求、傳送回應及接收回應認可的必要處理時間。|
|**成功伺服器延遲**|Azure 儲存體用來處理成功要求的平均時間。 此值不包括在 SuccessE2ELatency 中指定的網路延遲。 |
|**交易**|向儲存體服務或所指定 API 作業傳送的要求數。 此數目包括成功與失敗的要求，以及產生錯誤的要求。 在此範例中，區塊大小設為 100 MB。 在此案例中，每個 100 MB 的區塊皆視為一項交易。|
|**輸入**|輸入資料量。 此數目包括從外部用戶端輸入到 Azure 儲存體與 Azure 內的輸入。 |
|**輸出**|輸出資料量。 此數目包括從外部用戶端輸出到 Azure 儲存體與 Azure 內的輸出。 因此，此數目未反映可收費的輸出。 |

在 [時間] 旁選取 [過去 24 小時 (自動)]。 針對 [時間細微性] 選擇 [過去一小時] 和 [分鐘]，然後按一下 [套用]。

![儲存體帳戶計量](./media/storage-blob-scalable-app-verify-metrics/figure1.png)

您可以對圖表指派多個計量，但指派多個計量會停用按照維度進行群組的功能。

## <a name="dimensions"></a>維度

[維度](../common/storage-metrics-in-azure-monitor.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#metrics-dimensions)可用來更深入地查看圖表，並取得更詳細的資訊。 不同的計量會有不同的維度。 可使用的其中一個維度是 **API 名稱**維度。 此維度會將圖表細分為每個個別的 API 呼叫。 下方第一張圖表範例顯示儲存體帳戶的交易總數。 第二張顯示相同的圖表，但選擇了 API 名稱維度。 如您所見，列出的每一筆交易都有更詳細的資料，指出每個 API 名稱所執行的呼叫數量。

![儲存體帳戶計量 - 不含維度的交易](./media/storage-blob-scalable-app-verify-metrics/transactionsnodimensions.png)

![儲存體帳戶計量 - 交易](./media/storage-blob-scalable-app-verify-metrics/transactions.png)

## <a name="clean-up-resources"></a>清除資源

若不再需要，可刪除資源群組、虛擬機器和所有相關資源。 若要這樣做，請選取 VM 的資源群組，然後按一下 [刪除]。

## <a name="next-steps"></a>後續步驟

在此系列的第 4 部分中，您已了解如何檢視解決方案範例的計量，例如如何：

> [!div class="checklist"]
> * 在 Azure 入口網站中設定圖表
> * 驗證輸送量和延遲計量

遵循以下連結以查看預先建立的儲存體範例。

> [!div class="nextstepaction"]
> [Azure 儲存體指令碼範例](storage-samples-blobs-cli.md)

[previous-tutorial]: storage-blob-scalable-app-download-files.md