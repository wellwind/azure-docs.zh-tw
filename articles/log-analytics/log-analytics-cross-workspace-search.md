---
title: "使用 Azure Log Analytics 跨資源搜尋 | Microsoft Docs"
description: "本文說明如何針對來自訂用帳戶中多個工作區和 App Insights 應用程式的資源執行查詢。"
services: log-analytics
documentationcenter: 
author: MGoedtel
manager: carmonm
editor: 
ms.assetid: 
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/21/2018
ms.author: magoedte
ms.openlocfilehash: 5485b1634013c73b58932aafa6e17d636558715d
ms.sourcegitcommit: fbba5027fa76674b64294f47baef85b669de04b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/24/2018
---
# <a name="perform-cross-resource-log-searches-in-log-analytics"></a>在 Log Analytics 中執行跨資源記錄搜尋  

先前使用 Azure Log Analytics 時，您只能分析來自目前工作區內的資料，這項限制讓您無法跨訂用帳戶中定義的多個工作區執行查詢。  此外，您只能搜尋在 Application Insights 中或從 Visual Studio 直接使用 Application Insights 從 Web 應用程式收集的遙測項目。  如此一來，也難以將作業和應用程式資料一起以原生方式分析。   

現在您不僅可以跨多個 Log Analytics 工作區查詢，也可以查詢來自相同資源群組、另一個資源群組或其他訂用帳戶中特定 Application Insights 應用程式中的資料。 這讓您對資料能有全系統的檢視。  您只能在[進階入口網站](log-analytics-log-search-portals.md#advanced-analytics-portal) (無法在 Azure 入口網站) 中執行這些類型的查詢。  

## <a name="querying-across-log-analytics-workspaces-and-from-application-insights"></a>跨 Log Analytics 工作區和從 Application Insights 查詢
若要在查詢中參考另一個工作區，請使用 [*workspace*](https://docs.loganalytics.io/docs/Language-Reference/Scope-functions/workspace()) 識別項，而若要查詢來自 Application Insights 的應用程式，請使用 [*app*](https://docs.loganalytics.io/docs/Language-Reference/Scope-functions/app()) 識別項。  

### <a name="identifying-workspace-resources"></a>識別工作區資源
下列範例示範對 Log Analytics 工作區的查詢，從目前工作區和名為 *contosoretail-it* 的另一個工作區中的 Update 資料表，傳回其分類所需的更新項目彙總計數。 

您可以下列數種方式之一來完成識別工作區：

* 資源名稱 - 是人類看得懂的工作區名稱，有時稱為「元件名稱」。 

    `workspace("contosoretail").Update | count`
 
    >[!NOTE]
    >以名稱來識別工作區時，會假設該名稱在所有可存取的訂用帳戶中是唯一的。 如果您有多個應用程式具有該指定名稱，查詢就會因為語意模糊而失敗。 在此情況下，您必須使用其中一種其他識別碼。

* 限定名稱 - 是工作區的「完整名稱」，由訂用帳戶名稱、資源群組及元件名稱所組成，格式如下：*subscriptionName/resourceGroup/componentName*。 

    `workspace('contoso/contosoretail/development').requests | count `

    >[!NOTE]
    >由於 Azure 訂用帳戶名稱並非唯一，因此這個識別碼可能語意模糊。 
    >

* 工作區識別碼 - 工作區識別碼是指派給每個工作區的唯一、不可變識別碼，會以全域唯一識別碼 (GUID) 來表示。

    `workspace("b459b4u5-912x-46d5-9cb1-p43069212nb4").Update | count`

* Azure 資源識別碼 – Azure 定義的工作區唯一身分識別。 當資源名稱語意模糊時，您可以使用資源識別碼。  就工作區而言，格式為：*/subscriptions/subscriptionId/resourcegroups/resourceGroup/providers/microsoft.OperationalInsights/workspaces/componentName*。  

    例如︰
    ``` 
    workspace("/subscriptions/e427519-5645-8x4e-1v67-3b84b59a1985/resourcegroups/ContosoAzureHQ/providers/Microsoft.OperationalInsights/workspaces/contosoretail").Event | count
    ```

### <a name="identifying-an-application"></a>識別應用程式
下列範例會傳回對 Application Insights 中名為 *fabrikamapp* 的應用程式所進行要求的彙總計數。 

在 Application Insights 中識別應用程式可以使用 *app(Identifier)* 運算式來完成。  *Identifier* 引數會使用下列其中之一指定應用程式：

* 資源名稱 - 是人類看得懂的應用程式名稱，有時稱為「元件名稱」。  

    `app("fabrikamapp")`

* 限定名稱 - 是應用程式的「完整名稱」，由訂用帳戶名稱、資源群組及元件名稱所組成，格式如下：*subscriptionName/resourceGroup/componentName*。 

    `app("AI-Prototype/Fabrikam/fabrikamapp").requests | count`

     >[!NOTE]
    >由於 Azure 訂用帳戶名稱並非唯一，因此這個識別碼可能語意模糊。 
    >

* 識別碼 - 應用程式的應用程式 GUID。

    `app("b459b4f6-912x-46d5-9cb1-b43069212ab4").requests | count`

* Azure 資源識別碼 – Azure 定義的應用程式唯一身分識別。 當資源名稱語意模糊時，您可以使用資源識別碼。 格式為：*/subscriptions/subscriptionId/resourcegroups/resourceGroup/providers/microsoft.OperationalInsights/components/componentName*。  

    例如︰
    ```
    app("/subscriptions/b459b4f6-912x-46d5-9cb1-b43069212ab4/resourcegroups/Fabrikam/providers/microsoft.insights/components/fabrikamapp").requests | count
    ```

## <a name="next-steps"></a>後續步驟

若要檢視 Log Analytics 中提供的所有查詢語法選項，請檢閱 [Log Analytics 記錄搜尋參考資料](https://docs.loganalytics.io/docs/Language-Reference)。    