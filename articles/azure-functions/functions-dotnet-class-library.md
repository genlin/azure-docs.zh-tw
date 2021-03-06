---
title: Azure Functions C# 開發人員參考
description: 了解如何使用 C# 開發 Azure Functions。
services: functions
documentationcenter: na
author: ggailey777
manager: cfowler
editor: ''
tags: ''
keywords: azure functions, 函式, 事件處理, webhook, 動態計算, 無伺服器架構
ms.service: functions
ms.devlang: dotnet
ms.topic: reference
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 12/12/2017
ms.author: glenga
ms.openlocfilehash: 70c4d6276970a781517fe49ec47e9b2ddb884c78
ms.sourcegitcommit: 168426c3545eae6287febecc8804b1035171c048
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/08/2018
---
# <a name="azure-functions-c-developer-reference"></a>Azure Functions C# 開發人員參考

<!-- When updating this article, make corresponding changes to any duplicate content in functions-reference-csharp.md -->

本文是在 .NET 類別庫中使用 C# 開發 Azure Functions 的簡介。

Azure Functions 支援 C# 和 C# 指令碼程式設計語言。 如果您需要[在 Azure 入口網站中使用 C#](functions-create-function-app-portal.md)的指引，請參閱 [C# 指令碼 (.csx) 開發人員參考](functions-reference-csharp.md)。

本文假設您已閱讀下列文章：

* [Azure Functions 開發人員指南](functions-reference.md)
* [Azure Functions Visual Studio 2017 Tools](functions-develop-vs.md)

## <a name="functions-class-library-project"></a>Functions 類別庫專案

在 Visual Studio 中，**Azure Functions** 專案範本可建立 C# 類別庫專案，其中包含下列檔案：

* [host.json](functions-host-json.md) - 儲存會影響在本機或 Azure 中執行之專案中所有函式的組態設定。
* [local.settings.json](functions-run-local.md#local-settings-file) - 儲存在本機執行時所使用的應用程式設定和連接字串。

> [!IMPORTANT]
> 建置流程會為每個函式都建立 function.json 檔案。 這個 function.json 檔案不適合直接編輯。 您無法編輯此檔案來變更繫結設定或停用函式。 若要停用函式，請使用[停用](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/DisableAttribute.cs)屬性。 例如，新增布林值應用程式設定 MY_TIMER_DISABLED，然後將 `[Disable("MY_TIMER_DISABLED")]` 套用至函式。 然後，您可以變更應用程式設定來啟用和停用。

### <a name="functionname-and-trigger-attributes"></a>FunctionName 和觸發程序屬性

在類別庫中，函式是具有 `FunctionName` 和觸發程序屬性的靜態方法，如下列範例所示：

```csharp
public static class SimpleExample
{
    [FunctionName("QueueTrigger")]
    public static void Run(
        [QueueTrigger("myqueue-items")] string myQueueItem, 
        TraceWriter log)
    {
        log.Info($"C# function processed: {myQueueItem}");
    }
} 
```

`FunctionName` 屬性會將方法標記為函式進入點。 此名稱必須是專案中的唯一名稱。

觸發程序屬性可指定觸發程序類型，並將輸入資料繫結至方法參數。 範例函式是由佇列訊息所觸發，該佇列訊息會接著傳遞給 `myQueueItem` 參數中的方法。

### <a name="additional-binding-attributes"></a>其他繫結屬性

您可以使用其他輸入和輸出繫結屬性。 下列範例修改上一個範例並新增輸出佇列繫結。 此函式會將輸入佇列訊息寫入不同佇列中的新佇列訊息。

```csharp
public static class SimpleExampleWithOutput
{
    [FunctionName("CopyQueueMessage")]
    public static void Run(
        [QueueTrigger("myqueue-items-source")] string myQueueItem, 
        [Queue("myqueue-items-destination")] out string myQueueItemCopy,
        TraceWriter log)
    {
        log.Info($"CopyQueueMessage function processed: {myQueueItem}");
        myQueueItemCopy = myQueueItem;
    }
}
```

### <a name="order-of-parameters"></a>參數的順序

函式簽章中的參數順序不重要。 例如，您可以將觸發程序參數放在其他繫結之前或之後，且可以將記錄器參數放在觸發程序或繫結參數之前或之後。

### <a name="binding-expressions"></a>繫結運算式

您可以在屬性建構函式參數和函式參數中使用繫結運算式。 例如，下列程式碼會從應用程式設定取得要監視的佇列名稱，而它會在 `insertionTime` 參數中取得佇列訊息建立時間。

```csharp
public static class BindingExpressionsExample
{
    [FunctionName("LogQueueMessage")]
    public static void Run(
        [QueueTrigger("%queueappsetting%")] string myQueueItem,
        DateTimeOffset insertionTime,
        TraceWriter log)
    {
        log.Info($"Message content: {myQueueItem}");
        log.Info($"Created at: {insertionTime}");
    }
}
```

如需詳細資訊，請參閱[觸發程序和繫結](functions-triggers-bindings.md#binding-expressions-and-patterns)中的**繫結運算式和模式**。

### <a name="conversion-to-functionjson"></a>轉換成 function.json

建置流程在組建資料夾的函式資料夾中建立 *function.json* 檔案。 如稍早所述，此檔案不適合直接編輯。 您無法編輯此檔案來變更繫結設定或停用函式。 

此檔案的目的是提供資訊給縮放控制器，以用於[使用情況方案的縮放決策](functions-scale.md#how-the-consumption-plan-works)。 因此，檔案只會有觸發程序資訊，而不會有輸入或輸出繫結。

產生的 *function.json* 檔案包含 `configurationSource` 屬性 (property)，指示執行階段使用 .NET 屬性 (attribute) 屬性進行繫結，而不是使用 *function.json* 設定。 以下是範例：

```json
{
  "generatedBy": "Microsoft.NET.Sdk.Functions-1.0.0.0",
  "configurationSource": "attributes",
  "bindings": [
    {
      "type": "queueTrigger",
      "queueName": "%input-queue-name%",
      "name": "myQueueItem"
    }
  ],
  "disabled": false,
  "scriptFile": "..\\bin\\FunctionApp1.dll",
  "entryPoint": "FunctionApp1.QueueTrigger.Run"
}
```

### <a name="microsoftnetsdkfunctions-nuget-package"></a>Microsoft.NET.Sdk.Functions NuGet 套件

*function.json* 檔案產生是由 NuGet 套件 [Microsoft\.NET\.Sdk\.Functions](http://www.nuget.org/packages/Microsoft.NET.Sdk.Functions) 執行。 

Functions 執行階段的 1.x 版和 2.x 版都是使用同一個套件。 1.x 專案與 2.x 專案可依目標架構來區分。 以下是 *.csproj* 檔案的相關部分，顯示不同的目標架構和相同的 `Sdk` 套件：

**Functions 1.x**

```xml
<PropertyGroup>
  <TargetFramework>net461</TargetFramework>
</PropertyGroup>
<ItemGroup>
  <PackageReference Include="Microsoft.NET.Sdk.Functions" Version="1.0.8" />
</ItemGroup>
```

**Functions 2.x**

```xml
<PropertyGroup>
  <TargetFramework>netstandard2.0</TargetFramework>
  <AzureFunctionsVersion>v2</AzureFunctionsVersion>
</PropertyGroup>
<ItemGroup>
  <PackageReference Include="Microsoft.NET.Sdk.Functions" Version="1.0.8" />
</ItemGroup>
```

在 `Sdk` 套件相依性中的是觸發程序和繫結。 1.x 專案會參考 1.x 觸發程序和繫結，因為那些項目會將目標設為 .NET Framework，而 2.x 觸發程序和繫結則會將目標設為 .NET Core。

`Sdk` 套件也會相依於 [Newtonsoft.Json](http://www.nuget.org/packages/Newtonsoft.Json) \(英文\)，並間接相依於 [WindowsAzure.Storage](http://www.nuget.org/packages/WindowsAzure.Storage) \(英文\)。 這些相依性可確保您的專案會使用能夠搭配專案所設為目標之 Functions 執行階段版本運作的套件版本。 例如，`Newtonsoft.Json` 含有適用於 .NET Framework 4.6.1 的 11 版，但目標為 .NET Framework 4.6.1 的 Functions 執行階段只能與 `Newtonsoft.Json` 9.0.1 相容。 因此，您在該專案中的函式程式碼也必須使用 `Newtonsoft.Json` 9.0.1。

適用於 `Microsoft.NET.Sdk.Functions` 的原始程式碼位於 GitHub 存放庫 [azure\-functions\-vs\-build\-sdk](https://github.com/Azure/azure-functions-vs-build-sdk) \(英文\)。

### <a name="runtime-version"></a>執行階段版本

Visual Studio 會使用 [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools) 來執行 Functions 專案。 Core Tools 是適用於 Functions 執行階段的命令列介面。

如果您使用 npm 安裝 Core Tools，那就不會影響 Visual Studio 所使用的 Core Tools 版本。 對於 Functions 執行階段 1.x 版，Visual Studio 會在 *%USERPROFILE%\AppData\Local\Azure.Functions.Cli* 中儲存 Core Tools 版本，並使用儲存於該處的最新版本。 對於 Functions 2.x，Core Tools 會隨附於 **Azure Functions 與 Web 工作工具**擴充功能中。 對於 1.x 和 2.x，您可以在執行 Functions 專案時，於主控台輸出中查看使用的是哪個版本：

```terminal
[3/1/2018 9:59:53 AM] Starting Host (HostId=contoso2-1518597420, Version=2.0.11353.0, ProcessId=22020, Debug=False, Attempt=0, FunctionsExtensionVersion=)
```

## <a name="supported-types-for-bindings"></a>支援的繫結類型

每個繫結都有自己支援的類型；例如，Blob 觸發程序屬性可套用至字串參數、POCO 參數、`CloudBlockBlob` 參數或任何數個其他支援的類型。 [Blob 繫結的繫結參考文章](functions-bindings-storage-blob.md#trigger---usage)會列出所有支援的參數類型。 如需詳細資訊，請參閱[觸發程序和繫結](functions-triggers-bindings.md)以及[每個繫結類型的繫結參考文件](functions-triggers-bindings.md#next-steps)。

[!INCLUDE [HTTP client best practices](../../includes/functions-http-client-best-practices.md)]

## <a name="binding-to-method-return-value"></a>繫結至方法傳回值

您可以將方法傳回值用於輸出繫結，方法是將屬性套用至方法傳回值。 如需範例，請參閱[觸發程序和繫結](functions-triggers-bindings.md#using-the-function-return-value)。

## <a name="writing-multiple-output-values"></a>撰寫多個輸出值

若要多個值寫入至輸出繫結，請使用 [`ICollector`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/ICollector.cs) 或 [`IAsyncCollector`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IAsyncCollector.cs) 類型。 這些類型是在方法完成時，寫入至輸出繫結的唯寫集合。

這個範例會使用 `ICollector` 將多個佇列訊息寫入相同佇列：

```csharp
public static class ICollectorExample
{
    [FunctionName("CopyQueueMessageICollector")]
    public static void Run(
        [QueueTrigger("myqueue-items-source-3")] string myQueueItem,
        [Queue("myqueue-items-destination")] ICollector<string> myQueueItemCopy,
        TraceWriter log)
    {
        log.Info($"C# function processed: {myQueueItem}");
        myQueueItemCopy.Add($"Copy 1: {myQueueItem}");
        myQueueItemCopy.Add($"Copy 2: {myQueueItem}");
    }
}
```

## <a name="logging"></a>記錄

若要使用 C# 將輸出記錄至串流記錄，請包含 `TraceWriter` 類型的引數。 建議您將它命名為 `log`。 避免在 Azure Functions 中使用 `Console.Write`。 

`TraceWriter` 已定義於 [Azure WebJobs SDK](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Host/TraceWriter.cs)。 可以在 [host.json](functions-host-json.md) 中設定 `TraceWriter` 的記錄層級。

```csharp
public static class SimpleExample
{
    [FunctionName("QueueTrigger")]
    public static void Run(
        [QueueTrigger("myqueue-items")] string myQueueItem, 
        TraceWriter log)
    {
        log.Info($"C# function processed: {myQueueItem}");
    }
} 
```

> [!NOTE]
> 如需您可使用之較新記錄架構 (而非 `TraceWriter`) 的資訊，請參閱**監視 Azure Functions** 一文中的[在 C# 函式中寫入記錄](functions-monitoring.md#write-logs-in-c-functions)。

## <a name="async"></a>非同步處理

若要讓函式變成非同步，請使用 `async` 關鍵字並傳回 `Task` 物件。

```csharp
public static class AsyncExample
{
    [FunctionName("BlobCopy")]
    public static async Task RunAsync(
        [BlobTrigger("sample-images/{blobName}")] Stream blobInput,
        [Blob("sample-images-copies/{blobName}", FileAccess.Write)] Stream blobOutput,
        CancellationToken token,
        TraceWriter log)
    {
        log.Info($"BlobCopy function processed.");
        await blobInput.CopyToAsync(blobOutput, 4096, token);
    }
}
```

## <a name="cancellation-tokens"></a>取消權杖

可以接受 [CancellationToken](https://msdn.microsoft.com/library/system.threading.cancellationtoken.aspx) 參數的函式，讓作業系統能夠在函式即將終止時通知您的程式碼。 您可以使用此通知來確保函數不會在讓資料維持不一致狀態的情況下意外終止。

下列範例示範如何檢查即將終止的函式。

```csharp
public static class CancellationTokenExample
{
    public static void Run(
        [QueueTrigger("inputqueue")] string inputText,
        TextWriter logger,
        CancellationToken token)
    {
        for (int i = 0; i < 100; i++)
        {
            if (token.IsCancellationRequested)
            {
                logger.WriteLine("Function was cancelled at iteration {0}", i);
                break;
            }
            Thread.Sleep(5000);
            logger.WriteLine("Normal processing for queue message={0}", inputText);
        }
    }
}
```

## <a name="environment-variables"></a>環境變數

若要取得環境變數或應用程式設定值，請使用 `System.Environment.GetEnvironmentVariable`，如下列程式碼範例所示：

```csharp
public static class EnvironmentVariablesExample
{
    [FunctionName("GetEnvironmentVariables")]
    public static void Run([TimerTrigger("0 */5 * * * *")]TimerInfo myTimer, TraceWriter log)
    {
        log.Info($"C# Timer trigger function executed at: {DateTime.Now}");
        log.Info(GetEnvironmentVariable("AzureWebJobsStorage"));
        log.Info(GetEnvironmentVariable("WEBSITE_SITE_NAME"));
    }

    public static string GetEnvironmentVariable(string name)
    {
        return name + ": " +
            System.Environment.GetEnvironmentVariable(name, EnvironmentVariableTarget.Process);
    }
}
```

## <a name="binding-at-runtime"></a>執行階段的繫結

在 C# 和其他 .NET 語言中，您可以使用相對於屬性中[宣告式](https://en.wikipedia.org/wiki/Declarative_programming)繫結的[命令式](https://en.wikipedia.org/wiki/Imperative_programming)繫結模式。 當繫結參數需要在執行階段而不是設計階段中計算時，命令式繫結非常有用。 利用此模式，您可以快速在您的函式程式碼中繫結至支援的輸入和輸出繫結。

定義命令式繫結，如下所示︰

- **請勿**在函式簽章中加入您所需命令式繫結的屬性。
- 傳入輸入參數 [`Binder binder`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Host/Bindings/Runtime/Binder.cs) 或 [`IBinder binder`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IBinder.cs)。
- 使用下列 C# 模式來執行資料繫結。

  ```cs
  using (var output = await binder.BindAsync<T>(new BindingTypeAttribute(...)))
  {
      ...
  }
  ```

  `BindingTypeAttribute` 是可定義繫結的 .NET 屬性，而 `T` 是該繫結類型所支援的輸入或輸出類型。 `T` 不能是 `out` 參數類型 (例如 `out JObject`)。 例如，Mobile Apps 資料表輸出繫結支援[六個輸出類型](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs#L17-L22)，但您只可以搭配命令式繫結使用 [ICollector<T>](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/ICollector.cs) 或 [IAsyncCollector<T>](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IAsyncCollector.cs)。

### <a name="single-attribute-example"></a>單一屬性範例

下列範例程式碼會使用在執行階段定義的 blob 路徑來建立[儲存體 blob 輸出繫結](functions-bindings-storage-blob.md#output)，然後將字串寫入 blob。

```cs
public static class IBinderExample
{
    [FunctionName("CreateBlobUsingBinder")]
    public static void Run(
        [QueueTrigger("myqueue-items-source-4")] string myQueueItem,
        IBinder binder,
        TraceWriter log)
    {
        log.Info($"CreateBlobUsingBinder function processed: {myQueueItem}");
        using (var writer = binder.Bind<TextWriter>(new BlobAttribute(
                    $"samples-output/{myQueueItem}", FileAccess.Write)))
        {
            writer.Write("Hello World!");
        };
    }
}
```

[BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/BlobAttribute.cs) 會定義[儲存體 blob](functions-bindings-storage-blob.md) 輸入或輸出繫結，而 [TextWriter](https://msdn.microsoft.com/library/system.io.textwriter.aspx) 是支援的輸出繫結類型。

### <a name="multiple-attribute-example"></a>多個屬性範例

先前的範例會取得函數應用程式主要儲存體帳戶連接字串的應用程式設定 (也就是 `AzureWebJobsStorage`)。 您可以指定要用於儲存體帳戶的自訂應用程式設定，方法是新增 [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) 並將屬性陣列傳遞至 `BindAsync<T>()`。 使用 `Binder` 參數，而不是 `IBinder`。  例如︰

```cs
public static class IBinderExampleMultipleAttributes
{
    [FunctionName("CreateBlobInDifferentStorageAccount")]
    public async static Task RunAsync(
            [QueueTrigger("myqueue-items-source-binder2")] string myQueueItem,
            Binder binder,
            TraceWriter log)
    {
        log.Info($"CreateBlobInDifferentStorageAccount function processed: {myQueueItem}");
        var attributes = new Attribute[]
        {
        new BlobAttribute($"samples-output/{myQueueItem}", FileAccess.Write),
        new StorageAccountAttribute("MyStorageAccount")
        };
        using (var writer = await binder.BindAsync<TextWriter>(attributes))
        {
            await writer.WriteAsync("Hello World!!");
        }
    }
}
```

## <a name="triggers-and-bindings"></a>觸發和繫結 

下表列出的觸發程序和繫結屬性可在 Azure Functions 類別庫專案中使用。 所有屬性都在 `Microsoft.Azure.WebJobs` 命名空間中。

| 觸發程序 | 輸入 | 輸出|
|------   | ------    | ------  |
| [BlobTrigger](functions-bindings-storage-blob.md#trigger---attributes)| [Blob](functions-bindings-storage-blob.md#input---attributes)| [Blob](functions-bindings-storage-blob.md#output---attributes)|
| [CosmosDBTrigger](functions-bindings-cosmosdb.md#trigger---attributes)| [DocumentDB](functions-bindings-cosmosdb.md#input---attributes)| [DocumentDB](functions-bindings-cosmosdb.md#output---attributes) |
| [EventHubTrigger](functions-bindings-event-hubs.md#trigger---attributes)|| [EventHub](functions-bindings-event-hubs.md#output---attributes) |
| [HTTPTrigger](functions-bindings-http-webhook.md#trigger---attributes)|||
| [QueueTrigger](functions-bindings-storage-queue.md#trigger---attributes)|| [佇列](functions-bindings-storage-queue.md#output---attributes) |
| [ServiceBusTrigger](functions-bindings-service-bus.md#trigger---attributes)|| [ServiceBus](functions-bindings-service-bus.md#output---attributes) |
| [TimerTrigger](functions-bindings-timer.md#attributes) | ||
| |[ApiHubFile](functions-bindings-external-file.md)| [ApiHubFile](functions-bindings-external-file.md)|
| |[MobileTable](functions-bindings-mobile-apps.md#input---attributes)| [MobileTable](functions-bindings-mobile-apps.md#output---attributes) | 
| |[資料表](functions-bindings-storage-table.md#input---attributes)| [資料表](functions-bindings-storage-table.md#output---attributes)  | 
| ||[NotificationHub](functions-bindings-notification-hubs.md#attributes) |
| ||[SendGrid](functions-bindings-sendgrid.md#attributes) |
| ||[Twilio](functions-bindings-twilio.md#attributes)| 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解觸發程序和繫結](functions-triggers-bindings.md)

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 的最佳做法](functions-best-practices.md)
