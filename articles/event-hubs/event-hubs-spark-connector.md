---
title: 整合 Apache Spark 與 Azure 事件中樞 | Microsoft Docs
description: 整合 Apache Spark 以啟用事件中樞的結構化串流
services: event-hubs
documentationcenter: na
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.service: event-hubs
ms.devlang: java
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/05/2018
ms.author: shvija;sethm
ms.openlocfilehash: 3b44c7e7387eceb2d9ea0b2c214b13a82869110f
ms.sourcegitcommit: 8aab1aab0135fad24987a311b42a1c25a839e9f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/16/2018
---
# <a name="integrating-apache-spark-with-azure-event-hubs"></a>整合 Apache Spark 與 Azure 事件中樞

Azure 事件中樞會與 [Apache Spark](https://spark.apache.org/) 緊密整合，可讓您輕鬆建置端對端分散式串流應用程式。 這項整合支援 [Spark Core](http://spark.apache.org/docs/latest/rdd-programming-guide.html)、[Spark 串流](http://spark.apache.org/docs/latest/streaming-programming-guide.html)、[結構化串流](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)。 適用於 Apache Spark 的事件中樞連接器可於 [GitHub](https://github.com/Azure/azure-event-hubs-spark) 上取得。 此文件庫也可用於 [Maven 中央存放庫](http://search.maven.org/#artifactdetails%7Ccom.microsoft.azure%7Cazure-eventhubs-spark_2.11%7C2.1.6%7C)中的 Maven 專案。

本文說明如何在 [Azure Databrick](https://azure.microsoft.com/services/databricks/) 中製作連續的應用程式。 雖然本文使用 [Azure Databricks](https://azure.microsoft.com/services/databricks/)，但 Spark 叢集也可與 [HDInsight](../hdinsight/spark/apache-spark-overview.md) 搭配使用。

下列範例會使用兩個 Scala Notebook，一個用於串流事件中樞的事件，另一個用於將事件傳回給它。

## <a name="prerequisites"></a>先決條件

* Azure 訂用帳戶。 如果您沒有帳戶，請[建立免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)
* 事件中樞執行個體。 如果您沒有，請[建立一個](event-hubs-create.md)
* [Azure Databricks](https://azure.microsoft.com/services/databricks/) 執行個體。 如果您沒有，請[建立一個](../azure-databricks/quickstart-create-databricks-workspace-portal.md)
* [使用 Maven 座標建立文件庫](https://docs.databricks.com/user-guide/libraries.html#upload-a-maven-package-or-spark-package)：`com.microsoft.azure:azure‐eventhubs‐spark_2.11:2.3.0`

在 Notebook 中使用下列程式碼串流事件中樞的事件。

```scala
import org.apache.spark.eventhubs._

// To connect to an event hub, EntityPath is required as part of the connection string.
// Here, we assume that the connection string from the Azure portal does not have the EntityPath part.
val connectionString = ConnectionStringBuilder("{EVENT HUB CONNECTION STRING FROM AZURE PORTAL}")
  .setEventHubName("{EVENT HUB NAME}")
  .build 
val ehConf = EventHubsConf(connectionString)
  .setStartingPosition(EventPosition.fromEndOfStream)

// Create a stream that reads data from the specified Event Hub.
val reader = spark.readStream
  .format("eventhubs")
  .options(ehConf.toMap)
  .load()

// Select the body column and cast it to a string.
val eventhubs = reader.select($"body" cast "string")

eventhubs.writeStream
  .format("console")
  .outputMode("append")
  .start()
  .awaitTermination()
```
下列範例程式碼會透過 Spark 批次 API 將事件傳送至您的事件中樞。 您也可以撰寫串流查詢，將事件傳送至事件中樞。

```scala
import org.apache.spark.eventhubs._
import org.apache.spark.sql.functions._

// To connect to an event hub, EntityPath is required as part of the connection string.
// Here, we assume that the connection string from the Azure portal does not have the EntityPath part.
val connectionString = ConnectionStringBuilder("{EVENT HUB CONNECTION STRING FROM AZURE PORTAL}")
  .setEventHubName("{EVENT HUB NAME}")
  .build

val eventHubsConf = EventHubsConf(connectionString)

// Create a column representing the partitionKey.
val partitionKeyColumn = (col("id") % 5).cast("string").as("partitionKey")
// Create random strings as the body of the message.
val bodyColumn = concat(lit("random nunmber: "), rand()).as("body")

// Write 200 rows to the specified event hub.
val df = spark.range(200).select(partitionKeyColumn, bodyColumn)
df.write
  .format("eventhubs")
  .options(eventHubsConf.toMap)
  .save() 

```

## <a name="next-steps"></a>後續步驟

本文說明事件中樞連接器如何在建置即時、容錯之串流解決方案時發揮作用。 請使用下列連結，深入了解結構化串流和事件中樞整合連接器：

* [結構化串流 + Azure 事件中樞整合指南](https://github.com/Azure/azure-event-hubs-spark/blob/master/docs/structured-streaming-eventhubs-integration.md)
* [Spark 串流 + 事件中樞整合指南](https://github.com/Azure/azure-event-hubs-spark/blob/master/docs/spark-streaming-eventhubs-integration.md)
