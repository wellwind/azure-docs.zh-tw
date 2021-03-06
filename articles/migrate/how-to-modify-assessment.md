---
title: "自訂 Azure Migrate 評估設定 | Microsoft Docs"
description: "說明如何使用 Azure Migration Planner，設定和執行將 VMware VM 移轉至 Azure 的評估"
author: rayne-wiselman
ms.service: azure-migrate
ms.topic: article
ms.date: 02/26/2018
ms.author: raynew
ms.openlocfilehash: efb4ad59d25a0c1209e4f0f6cd406c2f0d48159c
ms.sourcegitcommit: 088a8788d69a63a8e1333ad272d4a299cb19316e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/27/2018
---
# <a name="customize-an-assessment"></a>自訂評估

[Azure Migrate](migrate-overview.md) 會使用預設屬性來建立評量。 在建立評量之後，您可以使用本文中的指示修改預設屬性。


## <a name="edit-assessment-properties"></a>編輯評量屬性

1. 在移轉專案的 [評量] 頁面中選取評量，然後按一下 [編輯屬性]。
2. 根據下表修改屬性：

    **設定** | **詳細資料** | **預設值**
    --- | --- | ---
    **目標位置** | 要作為移轉目的地的 Azure 位置。<br/><br/> Azure Migrate 目前支援 30 個區域，包括澳大利亞東部、澳大利亞東南部、巴西南部、加拿大中部、加拿大東部、印度中部、美國中部、中國東部、中國北部、東亞、美國東部、德國中部、德國東北部、美國東部 2、日本東部、日本西部、韓國中部、韓國南部、美國中北部、北歐、美國中南部、東南亞、印度南部、英國南部、英國西部、美國中西部、西歐、印度西部、美國西部和美國西部 2。 |  預設位置是「美國西部 2」。
    **儲存體備援** | Azure VM 會在移轉後使用的儲存體備援類型。 | [本地備援儲存體 (LRS)](../storage/common/storage-redundancy.md#locally-redundant-storage) 是預設值。 Azure Migrate 只支援以受控磁碟為基礎的評量，而受控磁碟只支援 LRS，因此此屬性目前只有 LRS 選項。 
    **調整大小準則** | Azure Migrate 用於為 Azure 設定適當 VM 大小的準則。 您可以執行*以效能為基礎*的大小調整或將 VM 調整為*內部部署*大小，而不必考量效能歷程記錄。 | 以效能為基礎的大小調整是預設選項。
    **效能歷程記錄** | 考量評估 VM 效能的持續時間。 當調整大小的準則為*以效能為基礎的大小調整*時，此屬性才適用。 | 預設值為一天。
    **百分位數使用率** | 要視為適當大小調整的效能取樣集百分位數值。 當調整大小的準則為*以效能為基礎的大小調整*時，此屬性才適用。  | 預設值為第 95 個百分位數。
    **定價層** | 您可以指定目標 Azure VM 的[定價層 (基本/標準)](../virtual-machines/windows/sizes-general.md)。 例如，如果您打算移轉到生產環境，則可考慮標準層，這會提供低延遲的 VM，但成本可能比較高。 另一方面，如果您具有開發測試環境，則可以考慮基本層，其中有較高延遲和較低成本的 VM。 | 預設會使用[標準](../virtual-machines/windows/sizes-general.md)層。
    **緩和因數** | Azure Migrate 會在評估期間考量緩衝區 (緩和因數)。 此緩衝區適用於 VM 的機器使用量資料 (CPU、記憶體、磁碟和網路)。 緩和因數會考量各個問題，例如季節性使用量、簡短的效能歷程記錄，以及未來可能增加的使用量。<br/><br/> 例如，10 核心的 VM，如果使用率只達 20%，一般即相當於一個 2 核心的 VM。 但如果緩和因數為 2.0 x，則結果為 4 核心的 VM。 | 預設設定是 1.3x。
    **提供項目** | 您註冊取得的 [Azure 優惠](https://azure.microsoft.com/support/legal/offer-details/)。 | 預設值是[隨用隨付](https://azure.microsoft.com/offers/ms-azr-0003p/)。
    **貨幣** | 帳單貨幣。 | 預設值為美元。
    **折扣 (%)** | 針對 Azure 供應項目所能獲得的任何訂用帳戶特定折扣。 | 預設設定為 0%。
    **Azure Hybrid Benefit** | 指定您是否擁有軟體保證以及是否符合 [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-use-benefit/) 的資格。 如果設定為 [是]，則會考慮對 Windows VM 使用非 Windows Azure 價格。 | 預設值為 [是]。

3. 按一下 [儲存] 以更新評估。


## <a name="next-steps"></a>後續步驟

[深入了解](concepts-assessment-calculation.md)評定的計算方式。
